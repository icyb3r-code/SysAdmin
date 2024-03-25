

# Upgrade Oracle Linux 7 to 8

references 

https://oracle-base.com/articles/linux/leapp-upgrade-ol7-to-ol8
https://docs.oracle.com/en/learn/ol-linux-leapp/index.html#upgrade-the-system


**Establish an SSH connection to your lab system instance.**

**Verify that the system locale is set to **en_US.UTF-8**.**

```bash
cat /etc/locale.conf
```

**Update to latest packages**

```bash
dnf check-update
dnf update
```

**This takes a few minutes, when the updates are completed, reboot the system.**

```bash
sudo reboot
```



**Enable Repos for Leapp application**

```bash
sudo yum install -y leapp --enablerepo=ol7_leapp,ol7_latest
```

**If not working or face issues you can add this to repo**

```bash
[ol7_leapp]
name=Oracle Linux $releasever Latest ($basearch)
baseurl=https://yum$ociregion.oracle.com/repo/OracleLinux/OL7/leapp/$basearch/
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-oracle
gpgcheck=1
enabled=1

```

****

```bash
sudo leapp preupgrade --oraclelinux
```


```bash
sudo cat /var/log/leapp/answerfile

Debug output written to /var/log/leapp/leapp-preupgrade.log

============================================================
                           REPORT
============================================================

A report has been generated at /var/log/leapp/leapp-report.json
A report has been generated at /var/log/leapp/leapp-report.txt

============================================================
                       END OF REPORT
============================================================

Answerfile has been generated at /var/log/leapp/answerfile
```

```bash
[root@aber-mongo03 ~]# cat /var/log/leapp/answerfile
[remove_pam_pkcs11_module_check]
# Title:              None
# Reason:             Confirmation
# =================== remove_pam_pkcs11_module_check.confirm ==================
# Label:              Disable pam_pkcs11 module in PAM configuration? If no, the upgrade process will be interrupted.
# Description:        PAM module pam_pkcs11 is no longer available in OL-8 since it was replaced by SSSD.
# Reason:             Leaving this module in PAM configuration may lock out the system.
# Type:               bool
# Default:            None
# Available choices: True/False
confirm = True
```

```bash
[root@aber-mongo03 ~]# cat /etc/oracle-release
Oracle Linux Server release 7.9
[root@aber-mongo03 ~]# uname -r
4.14.35-2047.521.4.el7uek.x86_64
[root@aber-mongo03 ~]#
```

```bash
leapp upgrade --oraclelinux
```