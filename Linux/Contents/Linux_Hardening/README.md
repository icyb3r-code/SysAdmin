
1.1 Filesystem Configuration ............................................................................ 19
1.1.13 Ensure separate partition exists for /home (Automated)................................. 51
1.1.14 Ensure nodev option set on /home partition (Automated) .............................. 53
1.1.15 Ensure nodev option set on /dev/shm partition (Automated) ....................... 55
1.1.16 Ensure nosuid option set on /dev/shm partition (Automated) ..................... 57
1.1.17 Ensure noexec option set on /dev/shm partition (Automated) ..................... 59
1.1.22 Disable Automounting (Automated) ......................................................................... 68
1.1.23 Disable USB Storage (Automated) .............................................................................. 70

1.2 Configure Software Updates ...................................................................................... 72
1.2.1 Ensure GPG keys are configured (Manual) ................................................................ 73
1.2.2 Ensure gpgcheck is globally activated (Automated) .............................................. 75
1.2.3 Ensure package manager repositories are configured (Manual) ...................... 77

3 Configure sudo ................................................................................................................................... 79
1.3.1 Ensure sudo is installed (Automated) ......................................................................... 80
1.3.2 Ensure sudo commands use pty (Automated) ......................................................... 82
1.3.3 Ensure sudo log file exists (Automated) ..................................................................... 84

1.5 Secure Boot Settings ................................................................................. 92
1.5.1 Ensure permissions on bootloader config are configured (Automated) ........ 93
1.5.2 Ensure bootloader password is set (Automated).................................................... 96
1.5.3 Ensure authentication required for single user mode (Automated) ............... 99

1.6 Additional Process Hardening ................................................................................................... 101
1.6.1 Ensure core dumps are restricted (Automated) .................................................... 102
1.6.2 Ensure address space layout randomization (ASLR) is enabled (Automated)
.............................................................................................................................................................. 104

1.8 Warning Banners..................................................................... 123
1.8.1 Command Line Warning Banners .................................................................................... 124
1.8.1.1 Ensure message of the day is configured properly (Automated) ................ 125
1.8.1.2 Ensure local login warning banner is configured properly (Automated) 127
1.8.1.3 Ensure remote login warning banner is configured properly (Automated)
..................................................................................... 129
1.8.1.4 Ensure permissions on /etc/motd are configured (Automated) ................ 131
1.8.1.5 Ensure permissions on /etc/issue are configured (Automated) ................. 133
1.8.1.6 Ensure permissions on /etc/issue.net are configured (Automated) ......... 134
1.8.2 Ensure GDM login banner is configured (Automated) ........................................ 136
1.9 Ensure updates, patches, and additional security software are installed
(Manual) ................................................................... 138
1.10 Ensure system-wide crypto policy is not legacy (Automated).......................... 140
1.11 Ensure system-wide crypto policy is FUTURE or FIPS (Automated) ............. 142



2 Services .......................................................................................... 144
2.1 inetd Services ............................................................................ 145
2.1.1 Ensure xinetd is not installed (Automated)...................................... 146
2.2 Special Purpose Services ............................................................................ 148
2.2.1 Time Synchronization ......................................................................................... 149
2.2.1.1 Ensure time synchronization is in use (Manual) ............................................... 150
2.2.1.2 Ensure chrony is configured (Automated) ........................................................... 152
2.2.2 Ensure X Window System is not installed (Automated) ..................................... 154
2.2.3 Ensure rsync service is not enabled (Automated) ................................................ 156
4|Page2.2.4 Ensure Avahi Server is not enabled (Automated) ................................................. 158
2.2.5 Ensure SNMP Server is not enabled (Automated) ................................................ 160
2.2.6 Ensure HTTP Proxy Server is not enabled (Automated) .................................... 162
2.2.7 Ensure Samba is not enabled (Automated) ............................................................. 164
2.2.8 Ensure IMAP and POP3 server is not enabled (Automated) ............................. 166
2.2.9 Ensure HTTP server is not enabled (Automated) ................................................. 168
2.2.10 Ensure FTP Server is not enabled (Automated) .................................................. 170
2.2.11 Ensure DNS Server is not enabled (Automated) ................................................. 172
2.2.12 Ensure NFS is not enabled (Automated) ................................................................ 174
2.2.13 Ensure RPC is not enabled (Automated) ................................................................ 176
2.2.14 Ensure LDAP server is not enabled (Automated) ............................................... 178
2.2.15 Ensure DHCP Server is not enabled (Automated) .............................................. 180
2.2.16 Ensure CUPS is not enabled (Automated) ............................................................. 182
2.2.17 Ensure NIS Server is not enabled (Automated) ................................................... 184
2.2.18 Ensure mail transfer agent is configured for local-only mode (Automated)
.............................................................................................................................................................. 186