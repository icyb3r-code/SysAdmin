# Bind9 DNS Server

## Introduction 

**What is DNS?**

DNS stands for Domain Name System. in shorts DNS is a Distributed Database system that is used to associate host names with their respective IP addresses. 

![](attachments/Pasted%20image%2020220919181305.png)

**Available DNS Software**

There are many different implementations of the DNS system today. Each DNS server represents a different implementation of characteristics, such as the interface, platform support, packaging and additional features.

* BIND
* PowerDNS
* Unbound
* Dnsmasq
* MaraDNS

## Lab Environment

DNS Server OS: Oracle linux 9
DNS software: Bind9
DNS IP Addr: 10.10.2.134/24
DNS Network: 10.10.2.0/24
Windows Server: 10.10.2.131

## Cheat-Sheet 

You can find below the steps in-short, for more details skip this part and follow the detailed steps.

```bash

#################################
### Set Host and Domain Name
#################################

hostnamectl 
hostnamectl set-hostname dnssrv.lab.com
hostname -s
hostname -d 
hostname -f

#################################
### Set Static IP Addr
#################################

## Oracle Linux 8/7
sudo vim /etc/sysconfig/network-scripts/ifcfg-ens160

# Change this 
BOOTPROTO=none

# Add below
ONBOOT=yes
IPADDR=10.10.2.134
PREFIX=24
GATEWAY=10.10.2.2
DNS1=10.10.2.2
DNS2=10.10.2.134

## Oracle Linux 9
sudo vim /etc/NetworkManager/system-connections/ens160.nmconnection

# add below lines 
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

# backup the file 
cp -p /etc/named.conf /etc/named.conf.bkp_$TIMESTAMP

vim /etc/named.conf

# Edit below lines
listen-on port 53 { 127.0.0.1; 10.10.2.134; };
allow-query     { localhost; 10.10.2.0/24; };

# validate the named.conf file 
named-checkconf 

#################################
### Create Zones
#################################

# Backup zones file 
su - root
export TIMESTAMP=$(date '+%Y%m%d_%H%M%S')
cp -p /etc/named.rfc1912.zones /etc/named.rfc1912.zones.bkp_$TIMESTAMP  

# Edit the file 
vim /etc/named.rfc1912.zones 

# add below zones to end of the file
//forward zone
zone "lab.com" IN {
     type master;
     file "lab.com.zone";
     allow-update { none; };
     allow-query { any; };
};
//backward zone or reverse zone
zone "2.10.10.in-addr.arpa" IN {
     type master;
     file "2.10.10.in-addr.arpa.zone";
     allow-update { none; };
     allow-query { any; };
};

#################################
### Create Forward Zone file
#################################

cd /var/named
cp -p named.loopback lab.com.zone
# -p to preserve the mode and ownership 

# chown root:named lab.com.zone
vim lab.com.zone
## edit the file to be like below 
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
#chown root:named 2.10.10.in-addr.arpa.zone
vim 2.10.10.in-addr.arpa.zone

## Edit the file to be like below 
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

# check configuration file
named-checkconf
# for chroot named file if activated
mkdir /var/named/chroot/usr/lib64/named
named-checkconf -t /var/named/chroot/ etc/named.conf

# check backward/reverse zone file
named-checkzone 127.0.0.1 /var/named/2.10.10.in-addr.arpa.zone

# check forwared zone file
named-checkzone dnssrv /var/named/lab.com.zone

#################################
### Enable chroot Jail Env
#################################

# check the chroot 
 mount | grep chroot

# Enable chroot 
 /usr/libexec/setup-named-chroot.sh /var/named/chroot on

# check chroot if enabled 
 mount | grep chroot

#################################
### Start named-chroot service
#################################

# Start service
systemctl start named-chroot
systemctl status named-chroot

#################################
### Allow DNS on Firewall
#################################

# check if ports are listening 
ss -anl | grep ":53 "

# add port 53 udp/tcp to firewall
firewall-cmd  --add-port=53/tcp --add-port=53/udp --permanent

# List ports 
firewall-cmd --list-ports

# Apply the changes 
firewall-cmd --reload

# List ports 
firewall-cmd --list-ports

#################################
### Put DNS in Action
#################################

# on DNS server locally add below to resolv.conf
vim /etc/resolv.conf
nameserver 10.10.2.134

# On windows add DNS to network adapter IPv4
10.10.2.134

# show advanced TCP/IP settings append DNS suffix
lab.com 

#################################
### Add DNS Record
#################################

# Open zone file to edit 
vim /var/named/lab.com.zone

# add below record 
win-sqldb01     IN A    10.10.2.131

# Open Reverse zone file to edit 
vim /var/named/2.10.10.in-addr.arpa.zone

# add below record 
win-sqldb01     IN A    10.10.2.131

# restart the server 
systemctl restart named-chroot

# Test from both DNS and Windows Server sides
nslookup win-sqldb01
nslookup 10.10.2.131

#################################
### Stop/Start DNS Service 
#################################

# stop service
systemctl stop named-chroot
systemctl status named-chroot

# disable the chroot jail env
/usr/libexec/setup-named-chroot.sh /var/named/chroot off
mount | grep chroot

##  Start Service 

# enable chroot jail env 
/usr/libexec/setup-named-chroot.sh /var/named/chroot on
mount | grep chroot

# Start service
systemctl start named-chroot
systemctl status named-chroot

```

## Step-1: Customize OS

```bash
## Set Proper server name and domain name

# check the hostname configuration
hostnamectl

# set hostname and domain
hostnamectl set-hostname dnssrv.lab.com

## Set Static IP 

#show the available networks
ip a show 

# make sure to make it static like below output
# Oracle linux 7/8
sudo vim /etc/sysconfig/network-scripts/ifcfg-ens160

# Change this 
BOOTPROTO=none

# Add below
ONBOOT=yes
IPADDR=10.10.2.134
PREFIX=24
GATEWAY=10.10.2.2
DNS1=10.10.2.2

# Oracle Linux 9

# edit below file using vim 
sudo vim /etc/NetworkManager/system-connections/ens160.nmconnection

# add below lines 
[ipv4]
address1=10.10.2.134/24,10.10.2.2
dns=10.10.2.2;
method=manual
```


## Step-2: Install Bind Utilities

Install the bind package.

```bash
sudo dnf install -y bind bind-chroot
rpm -aq | grep bind
```

Important files and directories good to know about, before we get stated:

```bash
# main configuration File 
/etc/named.conf 

# An auxiliary Direcotry for configuration file that are included in the man configuration file
/etc/named/

# File contains the Forward and Reverse Zones file names and configurations
/etc/named.rfc1912.zones

# conatins the zones files 
/var/named/

# Contains the jailed environment for bind DNS
/var/named/chroot/
```


## Step-3: Configure DNS Server (named.conf)

This file holds the bind configuration, which affect the service 

```bash
export TIMESTAMP=$(date '+%Y%m%d_%H%M%S')
systemctl stop named
systemctl disable named

# backup the file 
cp -p /etc/named.conf /etc/named.conf.bkp_$TIMESTAMP

vim /etc/named.conf

# Edit below lines
listen-on port 53 { 127.0.0.1; 10.10.2.134; };
allow-query     { localhost; 10.10.2.0/24; };

# validate the named.conf file 
named-checkconf 

```

## Step-4: Create Zones
Next create the DNS zone entries in the file `/etc/named.rfc1912.zones.`This is where you will configure the forward and reverse zones.

>You can add the zones in named.conf file in sake of keeping everything organized better to place them in `named.rfc1912.zones`  file.

```bash
# Backup zones file
su - root
export TIMESTAMP=$(date '+%Y%m%d_%H%M%S')
cp -p /etc/named.rfc1912.zones /etc/named.rfc1912.zones.bkp_$TIMESTAMP  

# Edit the file 
vim /etc/named.rfc1912.zones 

# add below zones to end of the file
//forward zone
zone "lab.com" IN {
     type master;
     file "lab.com.zone";
     allow-update { none; };
     allow-query { any; };
};
//backward zone or reverse zone
zone "2.10.10.in-addr.arpa" IN {
     type master;
     file "2.10.10.in-addr.arpa.zone";
     allow-update { none; };
     allow-query { any; };
};

```

## Step-5: Configure Forward Zone

```bash

cd /var/named
cp -p named.loopback lab.com.zone
# -p to preserve the mode and ownership 

# chown root:named lab.com.zone
vim lab.com.zone
## edit the file to be like below 
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

## Step-6: Configure Reverse Zone

```bash
cp -p named.localhost 2.10.10.in-addr.arpa.zone
vim 2.10.10.in-addr.arpa.zone

## Edit the file to be like below 
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

## Step-7: Validate Configurations

```bash
# check configuration file
named-checkconf
# for chroot named file
mkdir /var/named/chroot/usr/lib64/named
named-checkconf -t /var/named/chroot/ etc/named.conf

# check backward/reverse zone file
named-checkzone 127.0.0.1 /var/named/2.10.10.in-addr.arpa.zone

# check forwared zone file
named-checkzone dnssrv /var/named/lab.com.zone
```

## Step-8:  Enable Chroot Environment
Enable chroot environment to run bind in a jail environment.

```bash 
# check the chroot 
mount | grep chroot

# Enable chroot 
/usr/libexec/setup-named-chroot.sh /var/named/chroot on

# check chroot if enabled 
mount | grep chroot

# start named-chroot service
systemctl start named-chroot
systemctl status named-chroot
```

## Step-9: Allow DNS on Firewall

Allow the Incoming Firewall TCP/UDP port 53

```bash
# check if ports are listening 
ss -anl | grep ":53 "

# add port 53 udp/tcp to firewall
firewall-cmd --add-port=53/udp --permanent

# List ports 
firewall-cmd --list-ports

# Apply the changes 
firewall-cmd --reload

# List ports 
firewall-cmd --list-ports
```

## Step-10: Put DNS in action

```bash
# on DNS server locally add below to resolv.conf
vim /etc/resolv.conf
nameserver 10.10.2.134

# On windows add DNS to network adapter IPv4
10.10.2.134

# show advanced TCP/IP settings append DNS suffix
lab.com 
```

## Step-11: Add A Record

```bash
# Open zone file to edit 
vim /var/named/lab.com.zone

# add below record 
win-sqldb01     IN A    10.10.2.131

# Open Reverse zone file to edit 
vim /var/named/2.10.10.in-addr.arpa.zone

# add below record 
win-sqldb01     IN A    10.10.2.131

# restart the server 
systemctl restart named-chroot

# Test from both DNS and Windows Server sides
nslookup win-sqldb01
nslookup 10.10.2.131
```

## Step-12: Start/Stop DNS

```bash
## Stop Service

# stop service
systemctl stop named-chroot
systemctl status named-chroot

# disable the chroot jail env
/usr/libexec/setup-named-chroot.sh /var/named/chroot off
mount | grep chroot

##  Start Service 

# enable chroot jail env 
/usr/libexec/setup-named-chroot.sh /var/named/chroot on
mount | grep chroot

# Start service
systemctl start named-chroot
systemctl status named-chroot
```



