#    Chrony

## Intro

Chrony is a flexible implementation of the Network Time Protocol (NTP). its used to synchronized the system clock from different NTP servers, reference clock or via manual input. It  also can be used NTPv4 server to provide time service to other servers in the same network.

Chrony performs well in a wide range of conditions, including intermittent  network connections, heavily congested networks, changing temperatures  (ordinary computer clocks are sensitive to temperature), and systems  that do not run continuously, or run on a virtual machine.

**Comparison** between chrony and NTP can be found here in the [link](https://chrony.tuxfamily.org/comparison.html) , and for Chrony full **Documentation** check this [link](https://chrony.tuxfamily.org/documentation.html) .

## Structure

Chrony consists of `chronyd, chronyc, chrony.conf` 

* `chrony.conf` file locate on `/etc/` or `/etc/chrony/` directory  
* `chronyd` is a daemon process that run as service to 
* `chronyc` is a tool to monitor and control the `chronyd` process 

## Installation 

```bash
$ sudo apt install chrony 
```



## TimeDateCTL

The `timedatectl` command is using to set the date, time, timezone, enable and disable features such as NTP sync and HW clock.

```bash
# this command show the date and current configuration for time and date 
$ timedatectl 

# Sync Time to NIST Atomic Clock
$ timedatectl set–ntp yes 
$ timedatectl set-ntp no

# Change the Time Note time format should be in HH:MM:SS (Hours, Minutes, Seconds)
$ timedatectl set–time 21:45:53
# Change the Date, should be formatted as YYYY-MM-DD (Year, Month, and Day
$ timedatectl set–time 2019-04-10

# Set a Timezone
$ timedatectl list-timezones
$ timedatectl set–timezone Region/Location

# Set Universal Time
$ timedatectl set–timezone UTC

# Sync Hardware Clock
# RTC stands for Real-Time Clock, another name for the hardware clock in your computer. Your system has a tiny quartz crystal and a battery that keeps time at times when the system disconnects from a network.
$ timedatectl set-local-rtc 1 # this local time zone 
$ timedatectl set-local-rtc 0 # UTC


```



## Sever Configuration 

he nice thing about the Chrony configuration file is that this single  file configures the host as both a client and a server. To add a server  function to our host—it will always be a client, obtaining its time from a reference server—we just need to make a couple of changes to the  Chrony configuration, then configure the host's firewall to accept NTP  requests.

```bash
# ubuntu Firewall ufw.serive
systemctl stop ufw.serivce 
```



```bash
# Welcome to the chrony configuration file. See chrony.conf(5) for more
# information about usuable directives.

# This will use (up to):
# - 4 sources from ntp.ubuntu.com which some are ipv6 enabled
# - 2 sources from 2.ubuntu.pool.ntp.org which is ipv6 enabled as well
# - 1 source from [01].ubuntu.pool.ntp.org each (ipv4 only atm)
# This means by default, up to 6 dual-stack and up to 2 additional IPv4-only
# sources will be used.
# At the same time it retains some protection against one of the entries being
# down (compare to just using one of the lines). See (LP: #1754358) for the
# discussion.
#
# About using servers from the NTP Pool Project in general see (LP: #104525).
# Approved by Ubuntu Technical Board on 2011-02-08.
# See http://www.pool.ntp.org/join.html for more information.
#pool ntp.ubuntu.com        iburst maxsources 4
#pool 0.ubuntu.pool.ntp.org iburst maxsources 1
#pool 1.ubuntu.pool.ntp.org iburst maxsources 1
#pool 2.ubuntu.pool.ntp.org iburst maxsources 2
server 10.10.20.153 iburst prefer

# This directive specify the location of the file containing ID/key pairs for
# NTP authentication.
keyfile /etc/chrony/chrony.keys

# This directive specify the file into which chronyd will store the rate
# information.
driftfile /var/lib/chrony/chrony.drift

# Uncomment the following line to turn logging on.
#log tracking measurements statistics

# Log files location.
logdir /var/log/chrony

# Stop bad estimates upsetting machine clock.
maxupdateskew 100.0

# This directive enables kernel synchronisation (every 11 minutes) of the
# real-time clock. Note that it can’t be used along with the 'rtcfile' directive.
rtcsync

# Step the system clock instead of slewing it if the adjustment is larger than
# one second, but only in the first three clock updates.
makestep 1 3

local stratum 10

allow 10.10.20.0/24 
```





## Client configuration 

The NTP client configuration is simple and requires little or no  intervention. The NTP server can be defined during the Linux  installation or provided by the DHCP server at boot time. The default  `/etc/chrony.conf` file (shown below in its entirety) requires no intervention to work  properly as a client. For Fedora, Chrony uses the Fedora NTP pool, and  CentOS and RHEL have their own NTP server pools. Like many Red Hat-based distributions, the configuration file is well commented.

```bash
# Use public servers from the pool.ntp.org project.
# Please consider joining the pool (http://www.pool.ntp.org/join.html).
pool 2.fedora.pool.ntp.org iburst

# Record the rate at which the system clock gains/losses time.
driftfile /var/lib/chrony/drift

# Allow the system clock to be stepped in the first three updates
# if its offset is larger than 1 second.
makestep 1.0 3

# Enable kernel synchronization of the real-time clock (RTC).


# Enable hardware timestamping on all interfaces that support it.
#hwtimestamp *

# Increase the minimum number of selectable sources required to adjust
# the system clock.
#minsources 2

# Allow NTP client access from local network.
#allow 192.168.0.0/16

# Serve time even if not synchronized to a time source.
#local stratum 10

# Specify file containing keys for NTP authentication.
keyfile /etc/chrony.keys

# Get TAI-UTC offset and leap seconds from the system tz database.
leapsectz right/UTC

# Specify directory for log files.
logdir /var/log/chrony

# Select which information is logged.
#log measurements statistics tracking
```



Force Server time to client run below command:

```bash
# 
chronyd -q "server 10.10.20.153 iburst"
# OR
systemctl restart chronyd.serivce
```



After my client computers have synchronized with the NTP server, I like  to set the system hardware clock from the system (OS) time by using the  following command:

```bash
$ /sbin/hwclock --systohc
```



## Server Commands 

```bash
$ date 
Sat 27 Feb 2021 09:11:57 PM UTC

# stop Chrony service
$ systemctl stop chrony.service 
$ timedatectl set-ntp no
$ timedatectl set-time 21:18:25
$ timedatectl set-ntp yes
$ systemctl start chrony.service 
$ date 

```



## Client Commands 

```bash
$ nano /etc/chrony/chrony.conf
$ date 
$ chronyc sources -v 
# check the time difference between the client and server
$ watch chronyc tracking 
```



