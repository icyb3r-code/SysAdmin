# Bind9 DNS Server

## Table of Contents

- [Introduction](#introduction)
- [Lab Environment](#lab-environment)
- [Cheat Sheet](#cheat-sheet)
- [Step-1: Customize OS](#step-1-customize-os)
- [Step-2: Install Bind Utilities](#step-2-install-bind-utilities)
- [Step-3: Configure DNS Server](#step-3-configure-dns-server-namedconf)
- [Step-4: Create Zones](#step-4-create-zones)
- [Step-5: Configure Forward Zone](#step-5-configure-forward-zone)
- [Step-6: Configure Reverse Zone](#step-6-configure-reverse-zone)
- [Step-7: Validate Configurations](#step-7-validate-configurations)
- [Step-8: Enable Chroot Environment](#step-8-enable-chroot-environment)
- [Step-9: Allow DNS on Firewall](#step-9-allow-dns-on-firewall)
- [Step-10: Put DNS in Action](#step-10-put-dns-in-action)
- [Step-11: Add A Record](#step-11-add-a-record)
- [Step-12: Start/Stop DNS](#step-12-startstop-dns)

---

## Introduction

**What is DNS?**

DNS stands for Domain Name System. In short, DNS is a distributed database system used to associate hostnames with their respective IP addresses.

![DNS Diagram][def]

**Available DNS Software**

There are many different DNS server implementations, each differing in interface, platform support, packaging, and additional features:

| Software   | Description                              |
|------------|------------------------------------------|
| BIND       | The most widely used DNS software        |
| PowerDNS   | Feature-rich with database backends      |
| Unbound    | Focused on validation and security       |
| Dnsmasq    | Lightweight DNS/DHCP for small networks  |
| MaraDNS    | Simple, security-focused DNS server      |

---

## Lab Environment

| Component       | Value              |
|-----------------|--------------------|
| DNS Server OS   | Oracle Linux 9     |
| DNS Software    | Bind9              |
| DNS IP Address  | 10.10.2.134/24     |
| DNS Network     | 10.10.2.0/24       |
| Windows Server  | 10.10.2.131        |

---

## Cheat Sheet

A condensed reference for all steps. For full details, follow the step-by-step sections below.

```bash
#################################
### Set Hostname and Domain Name
#################################

hostnamectl
hostnamectl set-hostname dnssrv.lab.com
hostname -s
hostname -d
hostname -f

#################################
### Set Static IP Address
#################################

# Oracle Linux 7/8
sudo vim /etc/sysconfig/network-scripts/ifcfg-ens160

# Change this:
BOOTPROTO=none

# Add below:
ONBOOT=yes
IPADDR=10.10.2.134
PREFIX=24
GATEWAY=10.10.2.2
DNS1=10.10.2.2
DNS2=10.10.2.134

# Oracle Linux 9
sudo vim /etc/NetworkManager/system-connections/ens160.nmconnection

# Add below lines:
[ipv4]
address1=10.10.2.134/24,10.10.2.2
dns=10.10.2.2;10.10.2.134;
method=manual

#################################
### Installation
#################################

sudo dnf install -y bind bind-chroot
rpm -aq | grep bind

#################################
### Configure DNS Server
#################################

su - root
export TIMESTAMP=$(date '+%Y%m%d_%H%M%S')
systemctl stop named
systemctl disable named

cp -p /etc/named.conf /etc/named.conf.bkp_$TIMESTAMP
vim /etc/named.conf

# Edit below lines:
listen-on port 53 { 127.0.0.1; 10.10.2.134; };
allow-query     { localhost; 10.10.2.0/24; };

named-checkconf

#################################
### Create Zones
#################################

su - root
export TIMESTAMP=$(date '+%Y%m%d_%H%M%S')
cp -p /etc/named.rfc1912.zones /etc/named.rfc1912.zones.bkp_$TIMESTAMP

vim /etc/named.rfc1912.zones

# Add below zones to end of the file:
// forward zone
zone "lab.com" IN {
     type master;
     file "lab.com.zone";
     allow-update { none; };
     allow-query { any; };
};
// backward/reverse zone
zone "2.10.10.in-addr.arpa" IN {
     type master;
     file "2.10.10.in-addr.arpa.zone";
     allow-update { none; };
     allow-query { any; };
};

#################################
### Create Forward Zone File
#################################

cd /var/named
cp -p named.loopback lab.com.zone
vim lab.com.zone

$TTL 1D
@       IN SOA  lab.com.         root.lab.com (
                                        0       ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
                IN NS   localhost
localhost       IN A    127.0.0.1
dnssrv          IN A    10.10.2.134

#################################
### Create Backward/Reverse Zone
#################################

cp -p named.localhost 2.10.10.in-addr.arpa.zone
vim 2.10.10.in-addr.arpa.zone

$TTL 1D
@       IN SOA  lab.com.        root.lab.com. (
                                        0       ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
                IN NS   localhost.
134             IN PTR  dnssrv.lab.com.

#################################
### Validate Configuration
#################################

named-checkconf

# For chroot named file if activated:
mkdir /var/named/chroot/usr/lib64/named
named-checkconf -t /var/named/chroot/ etc/named.conf

named-checkzone 127.0.0.1 /var/named/2.10.10.in-addr.arpa.zone
named-checkzone dnssrv /var/named/lab.com.zone

#################################
### Enable Chroot Jail Environment
#################################

mount | grep chroot
/usr/libexec/setup-named-chroot.sh /var/named/chroot on
mount | grep chroot

#################################
### Start named-chroot Service
#################################

systemctl start named-chroot
systemctl status named-chroot

#################################
### Allow DNS on Firewall
#################################

ss -anl | grep ":53 "
firewall-cmd --add-port=53/tcp --add-port=53/udp --permanent
firewall-cmd --list-ports
firewall-cmd --reload
firewall-cmd --list-ports

#################################
### Put DNS in Action
#################################

# On DNS server, add to resolv.conf:
vim /etc/resolv.conf
nameserver 10.10.2.134

# On Windows, set DNS in network adapter IPv4:
# DNS: 10.10.2.134
# Append DNS suffix: lab.com

#################################
### Add DNS Record
#################################

vim /var/named/lab.com.zone
# Add: win-sqldb01     IN A    10.10.2.131

vim /var/named/2.10.10.in-addr.arpa.zone
# Add: 131             IN PTR  win-sqldb01.lab.com.

systemctl restart named-chroot

nslookup win-sqldb01
nslookup 10.10.2.131

#################################
### Stop/Start DNS Service
#################################

# Stop:
systemctl stop named-chroot
/usr/libexec/setup-named-chroot.sh /var/named/chroot off
mount | grep chroot

# Start:
/usr/libexec/setup-named-chroot.sh /var/named/chroot on
mount | grep chroot
systemctl start named-chroot
systemctl status named-chroot
```

---

## Step-1: Customize OS

Set the server hostname, domain name, and a static IP address before installing Bind.

```bash
# Check current hostname configuration
hostnamectl

# Set hostname and domain
hostnamectl set-hostname dnssrv.lab.com

# Verify
hostname -s   # short name
hostname -d   # domain name
hostname -f   # fully qualified domain name

# Show network interfaces
ip a show
```

**Oracle Linux 7/8 — Static IP via ifcfg:**

```bash
sudo vim /etc/sysconfig/network-scripts/ifcfg-ens160

# Change this:
BOOTPROTO=none

# Add below:
ONBOOT=yes
IPADDR=10.10.2.134
PREFIX=24
GATEWAY=10.10.2.2
DNS1=10.10.2.2
```

**Oracle Linux 9 — Static IP via NetworkManager:**

```bash
sudo vim /etc/NetworkManager/system-connections/ens160.nmconnection

# Add below lines:
[ipv4]
address1=10.10.2.134/24,10.10.2.2
dns=10.10.2.2;
method=manual
```

---

## Step-2: Install Bind Utilities

Install the `bind` and `bind-chroot` packages.

```bash
sudo dnf install -y bind bind-chroot
rpm -aq | grep bind
```

**Key files and directories:**

| Path                         | Description                                                   |
|------------------------------|---------------------------------------------------------------|
| `/etc/named.conf`            | Main configuration file                                       |
| `/etc/named/`                | Auxiliary directory for included configuration files          |
| `/etc/named.rfc1912.zones`   | Forward and reverse zone definitions                          |
| `/var/named/`                | Zone data files                                               |
| `/var/named/chroot/`         | Chroot jail environment for Bind                              |

---

## Step-3: Configure DNS Server (named.conf)

The `named.conf` file controls the core Bind service behavior.

```bash
su - root
export TIMESTAMP=$(date '+%Y%m%d_%H%M%S')
systemctl stop named
systemctl disable named

# Backup the original file
cp -p /etc/named.conf /etc/named.conf.bkp_$TIMESTAMP

vim /etc/named.conf

# Edit these lines to match your environment:
listen-on port 53 { 127.0.0.1; 10.10.2.134; };
allow-query     { localhost; 10.10.2.0/24; };

# Validate the configuration
named-checkconf
```

---

## Step-4: Create Zones

Define forward and reverse zones in `/etc/named.rfc1912.zones`.

> **Note:** You could add zones directly to `named.conf`, but placing them in `named.rfc1912.zones` keeps the configuration better organized.

```bash
su - root
export TIMESTAMP=$(date '+%Y%m%d_%H%M%S')
cp -p /etc/named.rfc1912.zones /etc/named.rfc1912.zones.bkp_$TIMESTAMP

vim /etc/named.rfc1912.zones

# Append the following zones at the end of the file:

// forward zone
zone "lab.com" IN {
     type master;
     file "lab.com.zone";
     allow-update { none; };
     allow-query { any; };
};

// backward/reverse zone
zone "2.10.10.in-addr.arpa" IN {
     type master;
     file "2.10.10.in-addr.arpa.zone";
     allow-update { none; };
     allow-query { any; };
};
```

---

## Step-5: Configure Forward Zone

The forward zone resolves hostnames to IP addresses.

```bash
cd /var/named

# Copy the loopback template, preserving permissions and ownership
cp -p named.loopback lab.com.zone

vim lab.com.zone
```

Edit the file to match the following:

```dns
$TTL 1D
@       IN SOA  lab.com.         root.lab.com (
                                        0       ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
                IN NS   localhost
localhost       IN A    127.0.0.1
dnssrv          IN A    10.10.2.134
```

---

## Step-6: Configure Reverse Zone

The reverse zone resolves IP addresses back to hostnames (PTR records).

```bash
cd /var/named
cp -p named.localhost 2.10.10.in-addr.arpa.zone

vim 2.10.10.in-addr.arpa.zone
```

Edit the file to match the following:

```dns
$TTL 1D
@       IN SOA  lab.com.        root.lab.com. (
                                        0       ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
                IN NS   localhost.
134             IN PTR  dnssrv.lab.com.
```

---

## Step-7: Validate Configurations

Check all configuration and zone files for syntax errors before starting the service.

```bash
# Validate the main configuration file
named-checkconf

# For chroot environment (if activated):
mkdir /var/named/chroot/usr/lib64/named
named-checkconf -t /var/named/chroot/ etc/named.conf

# Validate the reverse zone file
named-checkzone 127.0.0.1 /var/named/2.10.10.in-addr.arpa.zone

# Validate the forward zone file
named-checkzone dnssrv /var/named/lab.com.zone
```

---

## Step-8: Enable Chroot Environment

Running Bind inside a chroot jail improves security by isolating the process from the rest of the system.

```bash
# Check current chroot mounts
mount | grep chroot

# Enable the chroot environment
/usr/libexec/setup-named-chroot.sh /var/named/chroot on

# Confirm chroot is active
mount | grep chroot

# Start the chroot-enabled named service
systemctl start named-chroot
systemctl status named-chroot
```

---

## Step-9: Allow DNS on Firewall

Open TCP and UDP port 53 to allow DNS traffic through the firewall.

```bash
# Verify the DNS port is listening
ss -anl | grep ":53 "

# Allow port 53 on TCP and UDP (persistent)
firewall-cmd --add-port=53/tcp --add-port=53/udp --permanent

# Apply the changes
firewall-cmd --reload

# Confirm the ports are listed
firewall-cmd --list-ports
```

---

## Step-10: Put DNS in Action

Point clients to the new DNS server.

**On the DNS server (Linux):**

```bash
vim /etc/resolv.conf

# Add:
nameserver 10.10.2.134
```

**On Windows clients:**

1. Open network adapter properties > IPv4 settings
2. Set **Preferred DNS**: `10.10.2.134`
3. In **Advanced TCP/IP Settings**, append DNS suffix: `lab.com`

---

## Step-11: Add A Record

Add a new host record to both the forward and reverse zone files.

```bash
# Add to the forward zone
vim /var/named/lab.com.zone

# Append:
win-sqldb01     IN A    10.10.2.131


# Add to the reverse zone
vim /var/named/2.10.10.in-addr.arpa.zone

# Append:
131             IN PTR  win-sqldb01.lab.com.


# Restart the DNS service to apply changes
systemctl restart named-chroot


# Test forward and reverse resolution
nslookup win-sqldb01
nslookup 10.10.2.131
```

---

## Step-12: Start/Stop DNS

**Stop the service:**

```bash
systemctl stop named-chroot
systemctl status named-chroot

# Disable the chroot jail environment
/usr/libexec/setup-named-chroot.sh /var/named/chroot off
mount | grep chroot
```

**Start the service:**

```bash
# Re-enable the chroot jail environment
/usr/libexec/setup-named-chroot.sh /var/named/chroot on
mount | grep chroot

# Start the service
systemctl start named-chroot
systemctl status named-chroot
```


[def]: attachments/Pasted%20image%2020220919181305.png