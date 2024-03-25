
# Oracle Linux 8 ISO Local Repo


## Download Full ISO 

First you need to download the ISO image that belong to the OS version that currently running.

```bash
# check the version of the os 
cat /etc/os-release

# download the os release FULL ISO

wget https://yum.oracle.com/ISOS/OracleLinux/OL8/u7/x86_64/OracleLinux-R8-U7-x86_64-dvd.iso

```

## Prepare for Configuration


Create Directories 

```bash
# create below dirs 
mkdir /u01/iso
mkdir /u01/iso/ole8.7

# copy iso file to this dir
mv OracleLinux-R8-U7-x86_64-dvd.iso /u01/iso
```

Add below line to `fstab`

```bash
# add below line to fstab file 
echo "/u01/iso/OracleLinux-R8-U7-x86_64-dvd.iso /u01/iso/ole8.7 iso9660 loop,ro 0 0" >> /etc/fstab

# mount the iso 
mount -a
```

## Disable all Repos

```bash
#List all repos 
yum repolist all
dnf repolist all

# check and disable all repos
cd /etc/yum.repos.d/
grep -Ri "enabled=1" *
sed -i "s/enabled=1/enabled=0/" *
grep -Ri "enabled=1" *
# or 
dnf repolist all
```

## Add ISO Repo

```bash
vim /etc/yum.repos.d/OL87.repo

###################### Repo 
[OL87]
name=Oracle Linux 8.7 x86_64
baseurl=file:///u01/iso/ole8.7/AppStream/
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-oracle
gpgcheck=1
enabled=1

[OL87_v2]
name=Oracle Linux 8.7 x86_64
baseurl=file:///u01/iso/ole8.7/BaseOS/
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-oracle
gpgcheck=1
enabled=1
####################### End Repo

dnf repolist all

```


## Check Repo

```bash
# check repo 
dnf check-update
```

