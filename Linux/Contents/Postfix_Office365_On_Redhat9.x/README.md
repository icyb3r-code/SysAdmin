
# Configure Postfix with Office365 for Notifications

## Step 1/7 — Installing packages

```bash
# ─── 1. Install Postfix ───────────────────────────────────────────────────────
info "Step 1/7 — Installing packages"
dnf install -y postfix cyrus-sasl cyrus-sasl-plain s-nail 2>&1 | tee -a "$LOG_FILE"
if rpm -q postfix cyrus-sasl cyrus-sasl-plain s-nail &>/dev/null; then
    info "Packages installed: $(rpm -q postfix cyrus-sasl cyrus-sasl-plain s-nail | tr '\n' ' ')"
else
    error "Package installation failed — check $LOG_FILE"
fi
```

## Step 2/7 — Backing up existing configuration

```bash
# ─── 2. Backup existing config ───────────────────────────────────────────────
info "Step 2/7 — Backing up existing configuration"
BACKUP_DIR="/etc/postfix/backup_$(date +%Y%m%d_%H%M%S)"
mkdir -p "$BACKUP_DIR"
for f in main.cf sender_canonical header_checks sasl_passwd; do
    [[ -f "$POSTFIX_DIR/$f" ]] && cp "$POSTFIX_DIR/$f" "$BACKUP_DIR/" && info "  Backed up $f"
done
info "Backup saved to $BACKUP_DIR"
```


## Step 3/7 — Writing main.cf

```bash
# ─── 3. Write main.cf ────────────────────────────────────────────────────────
info "Step 3/7 — Writing main.cf"
cat > "$POSTFIX_DIR/main.cf" <<EOF
# Server identity
myhostname = $FQDN
mydomain = $DOMAIN
myorigin = \$mydomain

# Listen on localhost only
inet_interfaces = loopback-only
inet_protocols = ipv4

# Do not deliver locally
mydestination =

# Relay all mail through Office 365
relayhost = [$RELAY_HOST]:$RELAY_PORT

# TLS settings for Office 365
smtp_tls_security_level = encrypt
smtp_tls_CAfile = $CA_FILE
smtp_tls_loglevel = 1
smtp_tls_protocols = !SSLv2,!SSLv3,!TLSv1,!TLSv1.1
smtp_tls_ciphers = high
smtp_tls_session_cache_database = btree:/var/lib/postfix/smtp_tls_session_cache
smtp_tls_session_cache_timeout = 3600s

# SASL authentication
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:$POSTFIX_DIR/sasl_passwd
smtp_sasl_security_options = noanonymous
smtp_sasl_tls_security_options = noanonymous

# Sender rewriting
sender_canonical_maps = hash:$POSTFIX_DIR/sender_canonical
smtp_header_checks = regexp:$POSTFIX_DIR/header_checks

# Queue and size limits
maximal_queue_lifetime = 1d
bounce_queue_lifetime = 6h
message_size_limit = 10485760
mailbox_size_limit = 0

# Connection tuning
smtp_connect_timeout = 30s

# Security hardening
disable_vrfy_command = yes
smtpd_banner = \$myhostname ESMTP
EOF

```


## Step 4/7 — Writing sender_canonical and header_checks

```bash
# ─── 4. Write lookup tables ───────────────────────────────────────────────────
info "Step 4/7 — Writing sender_canonical and header_checks"

cat > "$POSTFIX_DIR/sender_canonical" <<EOF
# Rewrite local senders to the licensed O365 account
root                    $ALERT_EMAIL
root@${SHORT}           $ALERT_EMAIL
root@${FQDN}            $ALERT_EMAIL
@${SHORT}               $ALERT_EMAIL
@${FQDN}                $ALERT_EMAIL
EOF

cat > "$POSTFIX_DIR/header_checks" <<EOF
# Rewrite From header to match the licensed O365 sender
/^From:\s+.*\$/    REPLACE From: $ALERT_EMAIL
EOF
```
## Step 5/7 — Writing sasl_passwd

```bash
# ─── 5. Write sasl_passwd ────────────────────────────────────────────────────
info "Step 5/7 — Writing sasl_passwd"
printf '[%s]:%s\t%s:%s\n' "$RELAY_HOST" "$RELAY_PORT" "$ALERT_EMAIL" "$RELAY_PASS" \
    > "$POSTFIX_DIR/sasl_passwd"
chmod 600 "$POSTFIX_DIR/sasl_passwd"

# Build hash maps
postmap "$POSTFIX_DIR/sasl_passwd"
postmap "$POSTFIX_DIR/sender_canonical"
chmod 600 "$POSTFIX_DIR/sasl_passwd.db"
chmod 644 "$POSTFIX_DIR/sender_canonical.db"
info "Hash maps built."
```
## Step 6/7 — Validating configuration

```bash
# ─── 6. Validate and restart Postfix ─────────────────────────────────────────
info "Step 6/7 — Validating configuration"
if ! postfix check 2>&1 | tee -a "$LOG_FILE"; then
    error "postfix check failed — review the log at $LOG_FILE"
fi

info "Enabling and restarting Postfix"
systemctl enable postfix 2>&1 | tee -a "$LOG_FILE"
systemctl restart postfix 2>&1 | tee -a "$LOG_FILE"

# Give postfix a moment to come up
sleep 2
if systemctl is-active --quiet postfix; then
    info "Postfix is running."
else
    error "Postfix failed to start. Check: journalctl -u postfix -n 50"
fi
```

## Step 7/7 — Post-deployment checks

```bash
# ─── 7. Optional test email ──────────────────────────────────────────────────
info "Step 7/7 — Post-deployment checks"
if [[ -n "$TEST_RECIPIENT" ]]; then
    info "Sending test email to $TEST_RECIPIENT"
    echo "Postfix relay deployment successful on $(hostname) at $(date)" \
        | s-nail -s "[CompanyX] Postfix test from $(hostname)" "$TEST_RECIPIENT" \
        && info "Test email sent. Watch: tail -f /var/log/maillog" \
        || warn "s-nail command failed — check /var/log/maillog"
else
    info "No test recipient specified (-t). Skipping test email."
    info "To test manually:  echo 'test body' | s-nail -s 'postfix test' you@example.com"
fi

info "Deployment complete on $(hostname). Full log: $LOG_FILE"
info "Monitor mail logs:  tail -f /var/log/maillog"
info "Check queue:        mailq"
```

## Full Script

```bash
#!/bin/bash
# deploy_postfix.sh
# Deploys the CompanyX standard Postfix relay configuration on RHEL 9.x servers.
# Relays all mail through Office 365 using SASL auth, rewrites sender to dbalerts@example.com.
#
# Usage:
#   sudo ./deploy_postfix.sh -p <O365_PASSWORD> [-e <alert_email>] [-t <test_recipient>]
#
# Options:
#   -p  SASL password for the O365 relay account (required)
#   -e  Sender/relay account email  (default: dbalerts@example.com)
#   -t  Send a test email to this address after deployment (optional)
#   -h  Show this help

set -euo pipefail

# ─── Defaults ────────────────────────────────────────────────────────────────
RELAY_HOST="smtp.office365.com"
RELAY_PORT="587"
ALERT_EMAIL="dbalerts@example.com"
RELAY_PASS=""
TEST_RECIPIENT=""
CA_FILE="/etc/pki/tls/certs/ca-bundle.crt"
POSTFIX_DIR="/etc/postfix"
LOG_FILE="/var/log/postfix_deploy.log"

# ─── Colours ─────────────────────────────────────────────────────────────────
RED='\033[0;31m'; GREEN='\033[0;32m'; YELLOW='\033[1;33m'; NC='\033[0m'
touch "$LOG_FILE" 2>/dev/null || LOG_FILE="/tmp/postfix_deploy_$$.log"
info()  { echo -e "${GREEN}[INFO]${NC}  $*" | tee -a "$LOG_FILE"; }
warn()  { echo -e "${YELLOW}[WARN]${NC}  $*" | tee -a "$LOG_FILE"; }
error() { echo -e "${RED}[ERROR]${NC} $*" | tee -a "$LOG_FILE"; exit 1; }

# ─── Parse arguments ─────────────────────────────────────────────────────────
usage() {
    grep '^#' "$0" | grep -v '#!/' | sed 's/^# \?//'
    exit 0
}

while getopts "p:e:t:h" opt; do
    case $opt in
        p) RELAY_PASS="$OPTARG" ;;
        e) ALERT_EMAIL="$OPTARG" ;;
        t) TEST_RECIPIENT="$OPTARG" ;;
        h) usage ;;
        *) error "Unknown option. Run with -h for help." ;;
    esac
done

[[ -z "$RELAY_PASS" ]] && error "O365 SASL password is required. Use -p <password>"
[[ $EUID -ne 0 ]]      && error "This script must be run as root (sudo)."

# ─── Derived values ───────────────────────────────────────────────────────────
FQDN="$(hostname).example.com"
SHORT="dcCompanyXpppgbdb02v"
DOMAIN="example.com"

info "Starting Postfix deployment on $(hostname) at $(date)"
info "  FQDN     : $FQDN"
info "  Domain   : $DOMAIN"
info "  Relay    : [$RELAY_HOST]:$RELAY_PORT"
info "  Sender   : $ALERT_EMAIL"

# ─── 1. Install Postfix ───────────────────────────────────────────────────────
info "Step 1/7 — Installing packages"
dnf install -y postfix cyrus-sasl cyrus-sasl-plain s-nail 2>&1 | tee -a "$LOG_FILE"
if rpm -q postfix cyrus-sasl cyrus-sasl-plain s-nail &>/dev/null; then
    info "Packages installed: $(rpm -q postfix cyrus-sasl cyrus-sasl-plain s-nail | tr '\n' ' ')"
else
    error "Package installation failed — check $LOG_FILE"
fi

# ─── 2. Backup existing config ───────────────────────────────────────────────
info "Step 2/7 — Backing up existing configuration"
BACKUP_DIR="/etc/postfix/backup_$(date +%Y%m%d_%H%M%S)"
mkdir -p "$BACKUP_DIR"
for f in main.cf sender_canonical header_checks sasl_passwd; do
    [[ -f "$POSTFIX_DIR/$f" ]] && cp "$POSTFIX_DIR/$f" "$BACKUP_DIR/" && info "  Backed up $f"
done
info "Backup saved to $BACKUP_DIR"

# ─── 3. Write main.cf ────────────────────────────────────────────────────────
info "Step 3/7 — Writing main.cf"
cat > "$POSTFIX_DIR/main.cf" <<EOF
# Server identity
myhostname = $FQDN
mydomain = $DOMAIN
myorigin = \$mydomain

# Listen on localhost only
inet_interfaces = loopback-only
inet_protocols = ipv4

# Do not deliver locally
mydestination =

# Relay all mail through Office 365
relayhost = [$RELAY_HOST]:$RELAY_PORT

# TLS settings for Office 365
smtp_tls_security_level = encrypt
smtp_tls_CAfile = $CA_FILE
smtp_tls_loglevel = 1
smtp_tls_protocols = !SSLv2,!SSLv3,!TLSv1,!TLSv1.1
smtp_tls_ciphers = high
smtp_tls_session_cache_database = btree:/var/lib/postfix/smtp_tls_session_cache
smtp_tls_session_cache_timeout = 3600s

# SASL authentication
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:$POSTFIX_DIR/sasl_passwd
smtp_sasl_security_options = noanonymous
smtp_sasl_tls_security_options = noanonymous

# Sender rewriting
sender_canonical_maps = hash:$POSTFIX_DIR/sender_canonical
smtp_header_checks = regexp:$POSTFIX_DIR/header_checks

# Queue and size limits
maximal_queue_lifetime = 1d
bounce_queue_lifetime = 6h
message_size_limit = 10485760
mailbox_size_limit = 0

# Connection tuning
smtp_connect_timeout = 30s

# Security hardening
disable_vrfy_command = yes
smtpd_banner = \$myhostname ESMTP
EOF

# ─── 4. Write lookup tables ───────────────────────────────────────────────────
info "Step 4/7 — Writing sender_canonical and header_checks"

cat > "$POSTFIX_DIR/sender_canonical" <<EOF
# Rewrite local senders to the licensed O365 account
root                    $ALERT_EMAIL
root@${SHORT}           $ALERT_EMAIL
root@${FQDN}            $ALERT_EMAIL
@${SHORT}               $ALERT_EMAIL
@${FQDN}                $ALERT_EMAIL
EOF

cat > "$POSTFIX_DIR/header_checks" <<EOF
# Rewrite From header to match the licensed O365 sender
/^From:\s+.*\$/    REPLACE From: $ALERT_EMAIL
EOF

# ─── 5. Write sasl_passwd ────────────────────────────────────────────────────
info "Step 5/7 — Writing sasl_passwd"
printf '[%s]:%s\t%s:%s\n' "$RELAY_HOST" "$RELAY_PORT" "$ALERT_EMAIL" "$RELAY_PASS" \
    > "$POSTFIX_DIR/sasl_passwd"
chmod 600 "$POSTFIX_DIR/sasl_passwd"

# Build hash maps
postmap "$POSTFIX_DIR/sasl_passwd"
postmap "$POSTFIX_DIR/sender_canonical"
chmod 600 "$POSTFIX_DIR/sasl_passwd.db"
chmod 644 "$POSTFIX_DIR/sender_canonical.db"
info "Hash maps built."

# ─── 6. Validate and restart Postfix ─────────────────────────────────────────
info "Step 6/7 — Validating configuration"
if ! postfix check 2>&1 | tee -a "$LOG_FILE"; then
    error "postfix check failed — review the log at $LOG_FILE"
fi

info "Enabling and restarting Postfix"
systemctl enable postfix 2>&1 | tee -a "$LOG_FILE"
systemctl restart postfix 2>&1 | tee -a "$LOG_FILE"

# Give postfix a moment to come up
sleep 2
if systemctl is-active --quiet postfix; then
    info "Postfix is running."
else
    error "Postfix failed to start. Check: journalctl -u postfix -n 50"
fi

# ─── 7. Optional test email ──────────────────────────────────────────────────
info "Step 7/7 — Post-deployment checks"
if [[ -n "$TEST_RECIPIENT" ]]; then
    info "Sending test email to $TEST_RECIPIENT"
    echo "Postfix relay deployment successful on $(hostname) at $(date)" \
        | s-nail -s "[CompanyX] Postfix test from $(hostname)" "$TEST_RECIPIENT" \
        && info "Test email sent. Watch: tail -f /var/log/maillog" \
        || warn "s-nail command failed — check /var/log/maillog"
else
    info "No test recipient specified (-t). Skipping test email."
    info "To test manually:  echo 'test body' | s-nail -s 'postfix test' you@example.com"
fi

info "Deployment complete on $(hostname). Full log: $LOG_FILE"
info "Monitor mail logs:  tail -f /var/log/maillog"
info "Check queue:        mailq"
```

## Run Script

```bash
chmox +x deploy_postfix.sh
./deploy_postfix.sh   -t emailx@companyZ.com -p Password123
```