# Chrony — Local NTP Time Server

## Table of Contents

- [Introduction](#introduction)
- [Components](#components)
- [Installation](#installation)
- [timedatectl Reference](#timedatectl-reference)
- [Server Configuration](#server-configuration)
- [Client Configuration](#client-configuration)
- [Useful Commands](#useful-commands)

---

## Introduction

Chrony is a flexible implementation of the **Network Time Protocol (NTP)**. It synchronizes the system clock from NTP servers, reference clocks, or manual input. It can also act as an **NTPv4 server**, providing time services to other hosts on the same network.

Chrony performs well across a wide range of conditions:

- Intermittent or congested network connections
- Systems with changing temperatures (hardware clocks are temperature-sensitive)
- Systems that do not run continuously or run inside virtual machines

**References:**
- [Chrony vs NTP Comparison](https://chrony.tuxfamily.org/comparison.html)
- [Chrony Full Documentation](https://chrony.tuxfamily.org/documentation.html)

---

## Components

Chrony consists of three main components:

| Component    | Description                                                                 |
|--------------|-----------------------------------------------------------------------------|
| `chronyd`    | The daemon process that runs as a background service                        |
| `chronyc`    | CLI tool to monitor and control the `chronyd` process                       |
| `chrony.conf`| Main configuration file, located at `/etc/chrony.conf` or `/etc/chrony/`   |

> A single `chrony.conf` file configures the host as **both a client and a server**. No separate configuration file is needed for each role.

---

## Installation

**Debian / Ubuntu:**

```bash
sudo apt install -y chrony
```

**RHEL / CentOS / Oracle Linux / Fedora:**

```bash
sudo dnf install -y chrony
```

**Enable and start the service:**

```bash
sudo systemctl enable --now chronyd
systemctl status chronyd
```

---

## timedatectl Reference

`timedatectl` is a system utility for managing date, time, timezone, NTP sync, and hardware clock settings.

```bash
# Show current date, time, and sync status
timedatectl

# Enable or disable NTP synchronization
timedatectl set-ntp yes
timedatectl set-ntp no

# Set the system time manually (HH:MM:SS)
# Note: NTP must be disabled first
timedatectl set-ntp no
timedatectl set-time 21:45:53

# Set the system date manually (YYYY-MM-DD)
timedatectl set-time 2019-04-10

# List available timezones
timedatectl list-timezones

# Set a timezone
timedatectl set-timezone Region/Location

# Set timezone to UTC
timedatectl set-timezone UTC

# Sync hardware clock (RTC) to local time or UTC
# RTC = Real-Time Clock (hardware clock, battery-backed)
timedatectl set-local-rtc 1   # use local timezone
timedatectl set-local-rtc 0   # use UTC (recommended)
```

---

## Server Configuration

To serve time to other hosts on the network, configure `chrony.conf` to point to an upstream time source and allow client access from the local subnet. Then open the firewall for NTP traffic (UDP port 123).

### Firewall

**Ubuntu (ufw):**

```bash
sudo ufw allow 123/udp
sudo ufw reload
```

**RHEL / Oracle Linux (firewalld):**

```bash
sudo firewall-cmd --add-service=ntp --permanent
sudo firewall-cmd --reload
```

### /etc/chrony/chrony.conf (Server)

```bash
# Upstream NTP server (replace with your preferred source or upstream NTP server)
# iburst: send a burst of packets on startup for faster initial sync
# prefer: mark this as the preferred source
server 10.10.20.153 iburst prefer

# Key file for NTP authentication
keyfile /etc/chrony/chrony.keys

# File to store clock drift rate
driftfile /var/lib/chrony/chrony.drift

# Log directory
logdir /var/log/chrony

# Protect against large clock adjustments from bad sources
maxupdateskew 100.0

# Sync hardware RTC clock every 11 minutes (cannot be used with 'rtcfile')
rtcsync

# Step the clock (instead of slewing) if offset > 1 second, but only on first 3 updates
makestep 1 3

# Act as a local stratum 10 server if upstream is unreachable
# (stratum 10 indicates a local fallback, not a primary source)
local stratum 10

# Allow NTP client access from this subnet
allow 10.10.20.0/24
```

**Restart and verify:**

```bash
sudo systemctl restart chronyd
chronyc sources -v
chronyc tracking
```

---

## Client Configuration

The client configuration requires only pointing to the NTP server. The default config file works with no changes for most distros — simply replace the default pool with your internal server.

### /etc/chrony.conf (Client — RHEL/Fedora based)

```bash
# Internal NTP server
server 10.10.20.153 iburst

# Record the rate at which the system clock gains/loses time
driftfile /var/lib/chrony/drift

# Step the clock if offset > 1 second, only on first 3 updates
makestep 1.0 3

# Enable kernel synchronization of the hardware RTC
rtcsync

# Enable hardware timestamping on all interfaces that support it
#hwtimestamp *

# Minimum number of selectable sources required to adjust the system clock
#minsources 2

# Key file for NTP authentication
keyfile /etc/chrony.keys

# Get TAI-UTC offset and leap seconds from the system tz database
leapsectz right/UTC

# Log directory
logdir /var/log/chrony

# Select which information is logged
#log measurements statistics tracking
```

### Force Immediate Sync

Use this to force a one-time sync without starting the service:

```bash
chronyd -q "server 10.10.20.153 iburst"
```

Or restart the service to trigger re-sync:

```bash
sudo systemctl restart chronyd
```

### Sync Hardware Clock from System Time

After the system clock is synchronized, update the hardware (RTC) clock:

```bash
sudo /sbin/hwclock --systohc
```

---

## Useful Commands

### Server-Side

```bash
# Check current date and time
date

# Manually set time (stop NTP, set time, re-enable NTP)
systemctl stop chronyd
timedatectl set-ntp no
timedatectl set-time 21:18:25
timedatectl set-ntp yes
systemctl start chronyd
date
```

### Client-Side

```bash
# Edit the client chrony configuration
vim /etc/chrony/chrony.conf

# Check current date and time
date

# List NTP sources with verbose detail
chronyc sources -v

# Monitor time tracking in real-time (offset, frequency, skew)
watch chronyc tracking

# Show brief tracking summary
chronyc tracking

# Show activity (sources being used)
chronyc activity
```

### chronyc tracking — Key Fields

| Field              | Description                                              |
|--------------------|----------------------------------------------------------|
| `Reference ID`     | The NTP source currently being used                      |
| `Stratum`          | Distance from a primary time source (lower = better)     |
| `System time`      | Current offset between system clock and NTP source       |
| `Last offset`      | Clock offset at the last update                          |
| `RMS offset`       | Long-term average offset                                 |
| `Frequency`        | Rate at which the system clock gains/loses time (ppm)    |
| `Residual freq`    | Remaining frequency error                                |
| `Skew`             | Estimated error bound on the frequency                   |
| `Root delay`       | Total network path delay to the primary source           |
| `Root dispersion`  | Total dispersion accumulated to the primary source       |
| `Leap status`      | Whether a leap second is pending                         |
