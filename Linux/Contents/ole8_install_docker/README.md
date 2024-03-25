
# Docker Installation on OLE8


## Configure Docker Repo

```bash
#### Enable all the required repositories. To do this you are going to need the yum-utils package.
dnf install -y dnf-utils zip unzip
dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
```

## Install Docker

```bash
#### Install docker 
dnf remove -y runc
dnf install -y docker-ce --nobest
```


## Add Disk to Hold Docker Container

```bash
#### Check the LVM DISK
[root@nsi home]# lsblk
NAME           MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda              8:0    0  100G  0 disk
├─sda1           8:1    0    2G  0 part /boot/efi
├─sda2           8:2    0   16G  0 part
│ └─ol-swap    252:0    0   16G  0 lvm  [SWAP]
└─sda3           8:3    0   82G  0 part /
sdb              8:16   0  500G  0 disk
└─dockervg-u01 252:1    0  500G  0 lvm  /u01 ===> Here is the disk
sr0             11:0    1 1024M  0 rom
[root@nsi home]#

#### add new disk to fstab 
[root@nsi home]# cat /etc/fstab

#
# /etc/fstab
# Created by anaconda on Wed May 10 13:58:26 2023
#
# Accessible filesystems, by reference, are maintained under '/dev/disk/'.
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
#
# After editing this file, run 'systemctl daemon-reload' to update systemd
# units generated from this file.
#
UUID=4f021877-ceeb-41b8-b6ad-27a255c6ccd9 /                       xfs     defaults        0 0
UUID=2FEA-CBAF          /boot/efi               vfat    umask=0077,shortname=winnt 0 2
/dev/mapper/ol-swap     none                    swap    defaults        0 0
/dev/mapper/dockervg-u01        /u01    xfs     defaults        0       0
[root@nsi home]#
```

## Create A Docker Root Dir

```bash
mkdir /u01/docker
```


```bash
#### Add/Edit below file dockerd (docker daemon) configuration
vim /etc/docker/daemon.json
##add 
{
	"data-root" : "/u01/docker"
}
```

## Start Docker Daemon services

```bash
## Start Docker 
systemctl enable docker.service
systemctl start docker.service

## Start Docker socket
systemctl enable docker.socket
systemctl start docker.socket
```

## Check Docker Status

```bash
# Check docker status  
systemctl status docker.service
docker info
docker info | grep -i root 
docker version
```

## Install legacy Docker Compose

```bash
#### Install Docker Compose
curl -L https://github.com/docker/compose/releases/download/v2.5.0/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose


#### chmod of docker-compose permission
chmod +x /usr/local/bin/docker-compose
```

## Create Docker User

```bash
#### Create Group 
sudo groupadd docker


#### create User dckradmin
useradd dockeradmin -m

#### add docker group to dckradmin
usermod -aG docker dockeradmin
```