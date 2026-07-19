# Fedora 43 — Post-Installation Configuration Guide

A complete step-by-step guide for configuring a Fedora 43 workstation after a fresh installation. Suitable for both beginners and experienced users.

---

## Table of Contents

- [1. Verify Your System](#1-verify-your-system)
- [2. Speed Up DNF](#2-speed-up-dnf)
- [3. Set Hostname](#3-set-hostname)
- [4. Set Timezone](#4-set-timezone)
- [5. Enable RPM Fusion](#5-enable-rpm-fusion)
- [6. Update and Upgrade](#6-update-and-upgrade)
- [7. Enable Flathub (Flatpak)](#7-enable-flathub-flatpak)
- [8. Development Tools](#8-development-tools)
- [9. Multimedia Codecs](#9-multimedia-codecs)
- [10. OpenH264 for Firefox](#10-openh264-for-firefox)
- [11. NVIDIA Drivers](#11-nvidia-drivers)
- [12. Install Common Packages](#12-install-common-packages)
- [13. Configure Zsh Shell](#13-configure-zsh-shell)
- [14. Git Configuration](#14-git-configuration)
- [15. Firewall Basics](#15-firewall-basics)
- [16. SSH Key Setup](#16-ssh-key-setup)
- [17. VMware Workstation](#17-vmware-workstation)
- [18. Podman / Docker](#18-podman--docker)
- [19. Fonts](#19-fonts)
- [20. Power Management](#20-power-management)
- [Suggested Topics to Add](#suggested-topics-to-add)

---

## 1. Verify Your System

Before making any changes, confirm your Fedora version and system details:

```bash
# Check Fedora version
cat /etc/fedora-release

# Check OS details
hostnamectl

# Check kernel version
uname -r

# Check CPU and memory
lscpu
free -h

# Check disk layout
lsblk
df -h
```

---

## 2. Speed Up DNF

DNF is the default package manager on Fedora. Adding a few options to its configuration file significantly speeds up downloads and reduces interactive prompts.

```bash
sudo vim /etc/dnf/dnf.conf
```

Add the following lines under the `[main]` section:

```ini
# Speed up DNF downloads
fastestmirror=True
max_parallel_downloads=10
defaultyes=True
keepcache=True
```

| Option                  | Effect                                               |
|-------------------------|------------------------------------------------------|
| `fastestmirror`         | Automatically selects the fastest available mirror   |
| `max_parallel_downloads`| Downloads up to 10 packages simultaneously          |
| `defaultyes`            | Skips confirmation prompts (equivalent to `-y`)      |
| `keepcache`             | Keeps downloaded packages in cache for reuse         |

---

## 3. Set Hostname

The hostname identifies your machine on the network. Set it once after installation:

```bash
# Set a hostname (replace 'my-fedora' with your preferred name)
hostnamectl set-hostname my-fedora

# Verify
hostnamectl
```

---

## 4. Set Timezone

```bash
# List all available timezones
timedatectl list-timezones

# Filter by region
timedatectl list-timezones | grep -i asia
timedatectl list-timezones | grep -i europe

# Set your timezone
sudo timedatectl set-timezone Region/City

# Example:
sudo timedatectl set-timezone Asia/Dubai

# Verify date, time, and timezone
timedatectl
```

---

## 5. Enable RPM Fusion

RPM Fusion provides packages that Fedora does not ship officially due to licensing restrictions — including NVIDIA drivers, media codecs, and more.

> Reference: [rpmfusion.org/Configuration](https://rpmfusion.org/Configuration)

```bash
# Enable both free and non-free repositories
sudo dnf install \
  https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm \
  https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm

# Verify repositories are enabled
dnf repolist
```

> **Note:** RPM Fusion must be enabled before installing NVIDIA drivers, multimedia codecs, or Steam.

---

## 6. Update and Upgrade

Always update the system immediately after installation to get the latest kernel and security patches:

```bash
# Refresh metadata cache
sudo dnf makecache

# Apply all available updates
sudo dnf update && sudo dnf upgrade

# Reboot to apply kernel updates
reboot
```

---

## 7. Enable Flathub (Flatpak)

Flatpak is pre-installed on Fedora. Flathub is the largest Flatpak application repository and gives you access to thousands of apps (VS Code, Spotify, Discord, etc.) in sandboxed containers.

```bash
# Add the Flathub repository
flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo

# Verify
flatpak remotes

# Search for an app (example)
flatpak search firefox

# Install an app (example)
flatpak install flathub com.visualstudio.code

# Update all Flatpak apps
flatpak update
```

---

## 8. Development Tools

Install the compiler toolchain and kernel headers. These are required for building software from source, compiling kernel modules (VMware, VirtualBox, NVIDIA), and general development work.

```bash
# List available groups
sudo dnf grouplist | grep -i development

# Install the core development tools group
sudo dnf groupinstall "development-tools"

# Install kernel headers (needed for module compilation)
sudo dnf install kernel-devel kernel-headers
```

---

## 9. Multimedia Codecs

Fedora does not ship proprietary codecs by default. Install GStreamer plugins to enable playback of MP3, MP4, H.264, and other common formats.

> Reference: [Fedora Multimedia Docs](https://docs.fedoraproject.org/en-US/quick-docs/openh264/)

```bash
# Install GStreamer plugins for audio and video
sudo dnf install \
  gstreamer1-plugins-{bad-\*,good-\*,base} \
  gstreamer1-plugin-openh264 \
  gstreamer1-plugin-libav \
  --exclude=gstreamer1-plugins-bad-free-devel

# Install LAME MP3 encoder
sudo dnf install lame\* --exclude=lame-devel

# Upgrade multimedia group with all optional packages
sudo dnf group upgrade --with-optional Multimedia
```

---

## 10. OpenH264 for Firefox

Install H.264 codec support to enable hardware-accelerated video in Firefox and WebRTC video calls:

```bash
sudo dnf install gstreamer1-plugin-openh264 mozilla-openh264
```

After installation, open Firefox and navigate to `about:config`. Set the following preferences to `true` by double-clicking each entry:

```
media.gmp-gmpopenh264.autoupdate
media.gmp-gmpopenh264.enabled
media.gmp-gmpopenh264.provider.enabled
media.peerconnection.video.h264_enabled
```

Restart Firefox. Confirm the codec was downloaded by checking that this preference has a version value:

```
media.gmp-gmpopenh264.version
```

---

## 11. NVIDIA Drivers

> **Before you start:**
> - Only follow this section for supported GPUs: GT/GTX 600, 700, 800, 900, 1000, 1600 or RTX 2000, 3000, 4000 series.
> - Fedora ships with the open-source **Nouveau** driver. It may work for older unsupported GPUs.
> - **Disable Secure Boot** in UEFI/BIOS settings before proceeding.
> - [RPM Fusion](#5-enable-rpm-fusion) (non-free) must be enabled.

```bash
# Step 1: Update to the latest kernel and reboot
sudo dnf update
reboot

# Step 2: Install the NVIDIA akmod driver
# akmod automatically rebuilds the module after each kernel update
sudo dnf install akmod-nvidia

# Step 3: (Optional) Install CUDA support
# Required for GPU-accelerated apps: Blender, DaVinci Resolve, machine learning, etc.
sudo dnf install xorg-x11-drv-nvidia-cuda

# Step 4: Wait at least 5 minutes for the kernel module to build, then verify
modinfo -F version nvidia

# Step 5: Reboot
reboot
```

After reboot, verify the NVIDIA driver is active:

```bash
nvidia-smi
```

---

## 12. Install Common Packages

A baseline set of tools for a developer workstation:

```bash
sudo dnf install \
  gnome-tweaks gnome-extensions-app \
  vim git tmux zsh \
  wl-clipboard xclip xsel xrandr \
  gcc gcc-c++ make \
  egl-wayland egl-wayland-devel \
  htop wget curl unzip p7zip \
  NetworkManager-openvpn-gnome
```

| Package                       | Purpose                                          |
|-------------------------------|--------------------------------------------------|
| `gnome-tweaks`                | Advanced GNOME settings (fonts, theme, etc.)     |
| `gnome-extensions-app`        | Manage GNOME Shell extensions                    |
| `vim`                         | Terminal text editor                             |
| `git`                         | Version control                                  |
| `tmux`                        | Terminal multiplexer (split panes, sessions)     |
| `zsh`                         | Feature-rich shell (alternative to bash)         |
| `wl-clipboard`                | Clipboard tool for Wayland sessions              |
| `htop`                        | Interactive process viewer                       |
| `curl` / `wget`               | Command-line file download tools                 |
| `NetworkManager-openvpn-gnome`| OpenVPN integration for GNOME NetworkManager     |

---

## 13. Configure Zsh Shell

Zsh is a powerful shell with better auto-completion and plugin support compared to bash. [Oh My Zsh](https://ohmyz.sh/) is the most popular framework for managing Zsh configuration.

```bash
# Step 1: Install zsh (if not already installed)
sudo dnf install -y zsh

# Step 2: Set zsh as your default shell
chsh -s $(which zsh)

# Step 3: Log out and log back in, then verify
echo $SHELL

# Step 4: Install Oh My Zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

# Step 5: Edit your Zsh configuration
vim ~/.zshrc
```

Useful Oh My Zsh plugins to enable in `~/.zshrc`:

```bash
plugins=(git dnf sudo z tmux colored-man-pages)
```

---

## 14. Git Configuration

Set your identity before making any commits:

```bash
# Set your name and email
git config --global user.name "Your Name"
git config --global user.email "you@example.com"

# Set default branch name
git config --global init.defaultBranch main

# Set preferred editor
git config --global core.editor vim

# Enable color output
git config --global color.ui auto

# Verify configuration
git config --list
```

---

## 15. Firewall Basics

Fedora uses `firewalld` as its default firewall. It is enabled by default and uses a zone-based model.

```bash
# Check firewall status
sudo firewall-cmd --state

# List active zones and rules
sudo firewall-cmd --list-all

# Allow a service (example: SSH)
sudo firewall-cmd --add-service=ssh --permanent

# Allow a specific port (example: TCP 8080)
sudo firewall-cmd --add-port=8080/tcp --permanent

# Remove a rule
sudo firewall-cmd --remove-port=8080/tcp --permanent

# Apply all permanent changes
sudo firewall-cmd --reload

# List all allowed services and ports
sudo firewall-cmd --list-services
sudo firewall-cmd --list-ports
```

---

## 16. SSH Key Setup

SSH keys allow passwordless authentication to remote servers and services like GitHub.

```bash
# Generate an ED25519 key (recommended, modern and fast)
ssh-keygen -t ed25519 -C "your_email@example.com"

# Or generate an RSA key (4096-bit, wider compatibility)
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

# View your public key (copy this to remote servers or GitHub)
cat ~/.ssh/id_ed25519.pub

# Start the SSH agent and add your key
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# Copy your public key to a remote server
ssh-copy-id user@remote-server-ip

# Test the connection
ssh user@remote-server-ip
```

---

## 17. VMware Workstation

Download the VMware Workstation bundle from the [VMware website](https://www.vmware.com/products/workstation-pro.html) for Linux, then follow the steps below.

> Replace `17.5.0` and the bundle filename with your actual installed version throughout these steps.

### Step 1: Install Dependencies

```bash
sudo dnf install \
  fuse3 gtkmm4.0 \
  kernel-devel kernel-headers \
  gcc gcc-c++ make git \
  libnsl libnsl2 \
  libxcrypt libxcrypt-compat
```

### Step 2: Run the Installer

```bash
chmod +x VMware-Workstation-Full-17.5.0-22583795.x86_64.bundle
sudo ./VMware-Workstation-Full-17.5.0-22583795.x86_64.bundle
```

### Step 3: Build Kernel Modules

The `vmmon` and `vmnet` kernel modules must be compiled against your running kernel. Use one of the two methods below.

**Method 1 — Build and install from source (recommended):**

```bash
wget https://github.com/mkubecek/vmware-host-modules/archive/workstation-17.5.0.tar.gz
tar -xzf workstation-17.5.0.tar.gz
cd vmware-host-modules-workstation-17.5.0
make
sudo make install
```

**Method 2 — Replace original tarballs:**

```bash
wget https://github.com/mkubecek/vmware-host-modules/archive/workstation-17.5.0.tar.gz
tar -xzf workstation-17.5.0.tar.gz
cd vmware-host-modules-workstation-17.5.0
tar -cf vmmon.tar vmmon-only
tar -cf vmnet.tar vmnet-only
sudo cp -v vmmon.tar vmnet.tar /usr/lib/vmware/modules/source/
sudo vmware-modconfig --console --install-all
```

### Step 4: Start VMware Services

```bash
sudo systemctl enable --now vmware.service
sudo systemctl status vmware.service
```

---

## 18. Podman / Docker

Fedora ships with **Podman** by default — a daemonless, rootless container engine fully compatible with Docker commands.

```bash
# Podman is pre-installed — verify
podman --version

# Pull and run a container (same syntax as Docker)
podman run hello-world

# Install Docker if you specifically need it
sudo dnf install -y docker
sudo systemctl enable --now docker
sudo usermod -aG docker $USER
# Log out and back in for group change to take effect

# Verify Docker
docker --version
docker run hello-world
```

---

## 19. Fonts

Fedora does not include Microsoft fonts. Install them for better compatibility with Office documents and websites:

```bash
# Install the Fira Code developer font (ligatures, monospace)
sudo dnf install fira-code-fonts

# Install Google Noto fonts (broad Unicode coverage)
sudo dnf install google-noto-fonts-common

# Install Microsoft-compatible fonts via RPM Fusion (requires RPM Fusion free)
sudo dnf install mscore-fonts-all

# Refresh font cache after installation
fc-cache -fv

# List installed fonts
fc-list | sort
```

---

## 20. Power Management

Fedora uses `power-profiles-daemon` by default, which integrates with GNOME's power settings.

```bash
# Check available power profiles
powerprofilesctl list

# Set a profile
powerprofilesctl set performance     # maximum performance
powerprofilesctl set balanced        # default
powerprofilesctl set power-saver     # battery saving

# Check current profile
powerprofilesctl get
```

For laptops, install `TLP` for more granular battery and CPU power management:

```bash
sudo dnf install tlp tlp-rdw

# Enable and start TLP
sudo systemctl enable --now tlp

# Check TLP status
sudo tlp-stat -s
```

> **Note:** Do not run both `power-profiles-daemon` and `TLP` simultaneously — they conflict. Disable one before enabling the other:
> ```bash
> sudo systemctl disable --now power-profiles-daemon
> ```

---

## Suggested Topics to Add

The following topics are not yet covered in this guide but are recommended additions:

| Topic | Description |
|-------|-------------|
| **GNOME Customization** | Themes, icon packs, extensions (Dash to Panel, AppIndicator) |
| **Wayland vs X11** | Switching sessions, checking which is active, app compatibility |
| **Bluetooth Setup** | Pairing devices, fixing common Bluetooth issues on Fedora |
| **Printer Setup** | Installing CUPS, adding network printers, driver notes |
| **VPN Configuration** | WireGuard or OpenVPN client setup via NetworkManager |
| **Swap / Zram** | Fedora uses zram by default — tuning swap pressure and size |
| **Automatic Updates** | Configuring `dnf-automatic` for unattended security updates |
| **Backup Strategy** | Using `rsync`, `Timeshift`, or `Deja Dup` for system backups |
| **SELinux Basics** | Understanding and managing SELinux policies on Fedora |
| **Disk Encryption (LUKS)** | Encrypting home directory or secondary drives post-install |
