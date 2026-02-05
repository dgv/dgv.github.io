---
title: Basic Configuration Checklist
subtitle: 
hidden: true
---
_by ross ([original post]())_

<details>
  <summary><b>Checklist</b></summary>
<ul>
<li><b>Boot loader configuration</b><br><i>/boot/loader.conf</i></li>
<li><b>System variables</b><br><i>/etc/sysctl.conf</i></li>
<li><b>System startup options</b><br><i>/etc/rc.conf</i></li>
<li><b>Device ownership and permissions</b><br><i>/etc/devfs.rules</i></li>
<li><b>Periodic job configuration</b><br><i>/etc/periodic.conf</i></li>
<li><b>System build process configuration</b><br><i>/etc/make.conf</i><br><i>/etc/src.conf</i></li>
<li><b>Network setup</b><br><i>/etc/start_if.*</i><br><i>/etc/resolv.conf</i><br><i>/etc/dhclient.conf</i><br><i>/etc/hosts</i></li>
<li><b>File systems configuration</b><br><i>/etc/fstab</i><br><i>/etc/exports</i></li>
<li><b>Logs and log rotation</b><br><i>/etc/syslog.conf</i><br><i>/etc/newsyslog.conf</i></li>
<li><b>Accounts information</b><br><i>/etc/login.conf</i><br><i>/etc/master.passwd</i><br><i>/etc/passwd</i><br><i>/etc/group</i></li>
</ul> 
</details>
<br>
<details>
  <summary><b>Boot loader configuration</b></summary>
<br>
  <b>/boot/loader.conf</b> is used to tweak boot options, load kernel modules and set read-only kernel tunables (read-write kernel variables could be set with sysctl and /etc/sysctl.conf).
An example of /boot/loader.conf:

```sh
# Timeout in boot menu
autoboot_delay="3"

# A reasonable value here. But also see kern.* tunables in sysctl.conf
kern.maxusers="256"

# Uncomment if you use (and boot from) zfs,
# replace "system" with the pool name, i.e. zroot, tank
#zfs_load="YES"
#vfs.root.mountfrom="zfs:system"

# see https://wiki.freebsd.org/ZFSTuningGuide:
vfs.zfs.arc_min="1024M"
vfs.zfs.arc_max="1024M"
vfs.zfs.write_limit_override="256M"
vfs.zfs.txg.timeout="5"
vfs.zfs.prefetch_disable="1"

# Uncomment if you use gmirror
#geom_mirror_load="YES"

# Uncomment if you use gstripe
#geom_stripe_load="YES"
```

If you use ZFS you might want to limit ARC size otherwise ZFS will eat all the memory (controlled by vfs.zfs.arc_max). An alternative approach is to set vm.kmem_size and vm.kmem_size_max which limit memory available to the kernel.

  See [loader.conf(7)]() and <b>/boot/defaults/loader.conf</b>.
  
</details>
<br>
<details>
  <summary><b>System variables</b></summary>
<br>
<b>/etc/sysctl.conf</b>:

```sh
# Uncomment this to prevent users from seeing information about processes that
# are being run under another UID.
security.bsd.see_other_uids=0

# FS tunables
kern.maxvnodes=307200
kern.maxfiles=307200
kern.maxfilesperproc=300000

# Better handling of CPU-intensive tasks
kern.sched.preempt_thresh=220

# More shared memory
kern.ipc.shmmax=1073741824
kern.ipc.shmall=2097152

# Network
net.inet.tcp.recvspace=65536
net.inet.udp.recvspace=65536
net.inet.udp.maxdgram=131072

# Users can mount
vfs.usermount=1
```
See also [tuning(7)]().

You might not need that many vnodes (or might need more). Do <b>sysctl -a | grep vnodes</b> after all the services has started to see how many vnodes are in use. Set your maximum to something higher than that (if you ran out of vnodes disk IO will stall). Or use the default which is around 200K.
</details>
<br>
<details>
  <summary><b>System startup options</b></summary>
<br>
<b>/etc/rc.conf</b>:

```sh
# System options
export PATH=/root/bin:/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin
rcshutdown_timeout="180"
keymap=us.iso.kbd

# Network setup
hostname="coffin.lan"
ifconfig_re0="DHCP"
ifconfig_re1="inet 192.168.10.1/24"
#defaultrouter="10.0.39.1"
gateway_enable="YES"
icmp_bmcastecho="NO"
icmp_drop_redirect="YES"
icmp_log_redirect="NO"

# dhclient setup
synchronous_dhclient="YES"
defaultroute_delay="60"
defaultroute_carrier_delay="10"

# Wait for network
netwait_enable="YES"
netwait_ip="8.8.8.8"
netwait_if="re0"

# ZFS
zfs_enable="YES"

# UFS
fsck_y_enable="YES"
background_fsck="NO"
quota_enable="YES"

# mdmfs on /tmp
tmpmfs="YES"
tmpsize="512M"

# devfs rules
devfs_system_ruleset="system"

# Disable networking in syslogd
syslogd_flags="-ss"

# Enable Secure Shell
sshd_enable="YES"

# Enable NFS Server and Client
nfs_server_enable="YES"
rpcbind_enable="YES"
rpc_lockd_enable="YES"
rpc_statd_enable="YES"
nfs_client_enable="YES"
```
See rc.conf(5) and <b>/etc/defaults/rc.conf</b>.
</details>
<br>
<details>
  <summary><b>Device ownership and permissions</b></summary>
<br>
Add to <b>/etc/rc.conf</b>:

```sh
# devfs rules
devfs_system_ruleset="system"
```

Create <b>/etc/devfs.rules</b>:

```sh
[system=10]
# With the following growisofs will write DVDs as user:
add path 'cd*' mode 0660
# "camcontrol devlist" will show passN of cd0, add it too:
add path 'pass4' mode 0660
# Enable flash automount in KDE:
add path 'da*' mode 0660
```
By default FreeBSD uses group <b>operator</b> on devices special files, so add yourself and other users that should have access to hardware to that group.

Default permissions on devices are 0600 so only the owner (root) have access. You can change it to 0660, for example.

If you want to change user or group of the device add <b>user username</b> or <b>group groupname</b> to the end of the line.

</details>
<br>
<details>
  <summary><b>Periodic job configuration</b></summary>
<br>
See periodic.conf(7) and /etc/defaults/periodic.conf. Setup periodic jobs:
/etc/periodic.conf:

```sh
# Clean /tmp
daily_clean_tmps_enable="YES"
daily_clean_tmps_days="3"

# If you installed sysutils/smartmontools:
daily_status_smart_devices="AUTO"

# Uncomment if you use zfs
#daily_status_zfs_enable="YES"

# Uncomment if you use gmirror
#daily_status_gmirror_enable="YES"

# Uncomment if you use gstripe
#daily_status_gstripe_enable="YES"

# Uncomment if you don't use sendmail
#daily_clean_hoststat_enable="NO"
#daily_status_mail_rejects_enable="NO"
#daily_status_include_submit_mailq="NO"
#daily_submit_queuerun="NO"
```

Additional periodic scripts
Hourly periodic scripts support

If you are going to use/create hourly scripts you need to add support for it:

Add to /etc/periodic.conf:
```sh
hourly_output="root"       # user or /file
hourly_show_success="NO"   # scripts returning 0
hourly_show_info="YES"     # scripts returning 1
hourly_show_badconfig="NO" # scripts returning 2
```
Add to /etc/crontab:
```sh
1       *       *       *       *       root    periodic hourly
```
```sh
# mkdir /etc/periodic/hourly
# service cron restart
```
ZFS snapshots

Install sysutils/zfs-periodic. It will list available variables on install. I use it like this (/etc/periodic.conf):
```sh
daily_zfs_snapshot_enable="YES"
daily_zfs_snapshot_pools="system"
daily_zfs_snapshot_keep=7
monthly_zfs_scrub_enable="YES"
monthly_zfs_scrub_pools="system"
```
Daily ports/packages status check

Your nightly mail will have the list of ports/packages in your system and in your ezjails that should be upgraded. It will also update the ports tree using portsnap cron command.
```sh
# mkdir -p /usr/local/etc/periodic/daily
# cd /usr/local/etc/periodic/daily
# fetch http://daemon-notes.com/downloads/assets/scripts/910.check-updates
# chmod a+x 910.check-updates
```
Look inside 910.check-updates for available variables.
Hourly disk-free status check

This script requires hourly periodic support and is written for UFS.
```sh
# mkdir -p /usr/local/etc/periodic/hourly
# cd /usr/local/etc/periodic/hourly
# fetch http://daemon-notes.com/downloads/assets/scripts/900.status-df
# chmod a+x 900.status-df
```
Look inside 900.status-df for available variables.
Daily directories backup
```sh
# mkdir -p /usr/local/etc/periodic/daily
# cd /usr/local/etc/periodic/daily
# fetch http://daemon-notes.com/downloads/assets/scripts/250.backup-dirs
# chmod a+x 250.backup-dirs
```
Look inside 250.backup-dirs for available variables. I use it backup /etc, /usr/local/etc, /root, /var/cron/tabs and directories of virtual hosts.
Daily SVN repositories backup
```sh
# mkdir -p /usr/local/etc/periodic/daily
# cd /usr/local/etc/periodic/daily
# fetch http://daemon-notes.com/downloads/assets/scripts/251.backup-svn
# chmod a+x 251.backup-svn
```
Look inside 251.backup-svn for available variables.

</details>
<br>
<details>
  <summary><b>System build process configuration</b></summary>
<br>
make.conf

Copy <b>/usr/share/examples/etc/make.conf</b> to <b>/etc/make.conf</b>.

_diff -u /usr/share/examples/etc/make.conf /etc/make.conf_
```diff
--- /usr/share/examples/etc/make.conf   2008-09-27 00:22:11.000000000 +0300
+++ /etc/make.conf      2008-09-27 13:09:39.000000000 +0300
@@ -43,6 +43,7 @@
 # (?= allows to buildworld for a different CPUTYPE.)
 #
 #CPUTYPE?=pentium3
+CPUTYPE?=core2
 #NO_CPU_CFLAGS=                # Don't add -march=<cpu> to CFLAGS automatically
 #NO_CPU_COPTFLAGS=     # Don't add -march=<cpu> to COPTFLAGS automatically
 #
@@ -57,6 +58,7 @@
 # explicitly turn it off when using compiling with the -O2 optimization level.
 #
 #CFLAGS= -O2 -fno-strict-aliasing -pipe
+CFLAGS= -O2 -fno-strict-aliasing -pipe
 #
 # CXXFLAGS controls the compiler settings used when compiling C++ code.
 # Note that CXXFLAGS is initially set to the value of CFLAGS.  If you wish
@@ -88,6 +90,7 @@
 # so can cause problems.
 #
 #COPTFLAGS= -O -pipe
+COPTFLAGS= -O -pipe
 #
 # Compare before install
 #INSTALL=install -C
@@ -270,3 +273,14 @@
 # /etc/mail/Makefile.  Defaults to 0640.
 #
 #SENDMAIL_MAP_PERMS=
+
+WITHOUT_X11=YES
```
If your host is behind NAT and you must use proxy add the following to <b>/etc/make.conf</b>:
```sh
FETCH_ENV=HTTP_PROXY=http://coffin.lan:3128/ FTP_PROXY=http://coffin.lan:3128/
```
Replace <b>coffin.lan</b> with the name of the proxy and <b>3128</b> with the port to use.
<b>/etc/src.conf</b>:
```sh
WITHOUT_PROFILE=YES
```
WITHOUT_PROFILE: Avoid compiling profiled libraries. This is optional and could be ommited. There are other options, see the man page: <b>src.conf(1)</b>.
</details>
<br>
<details>
  <summary><b>Network setup</b></summary>
<br>
You can initialize a network interface by creating file start_if.<interface name>. Here we will enable device polling (requires DEVICE_POLLING option in kernel) and set media manually.
<b>/etc/start_if.rl0</b>:

ifconfig rl0 polling media 100baseTX mediaopt full-duplex

<b>/etc/start_if.nfe0</b>:

ifconfig nfe0 polling media 100baseTX mediaopt full-duplex

> [!NOTE]
> When setting the media manually make sure both ends of the connection are set manually and identically. Or better just ommit media and mediaopt options for auto-negotiation.

<b>/etc/resolv.conf</b> (with local DNS server running):
```sh
search local
nameserver 127.0.0.1
nameserver 123.45.67.89
nameserver 123.45.67.90
```
<b>/etc/dhclient.conf</b>:
```diff
--- /usr/src/etc/dhclient.conf  2010-12-21 19:09:25.000000000 +0200
+++ /etc/dhclient.conf  2011-03-01 14:13:30.000000000 +0200
@@ -6,3 +6,17 @@
 #      In most cases an empty file is sufficient for most people as the
 #      defaults are usually fine.
 #
+
+timeout 60;
+retry 60;
+reboot 10;
+select-timeout 5;
+initial-interval 2;
+
+interface "re0" {
+        send dhcp-lease-time 86400;
+        supersede host-name "coffin";
+        supersede domain-name "local";
+        # The following requires local DNS server
+        prepend domain-name-servers 127.0.0.1;
+}
```
<b>/etc/hosts</b>:
```diff
--- /usr/src/etc/hosts  2010-12-21 19:09:25.000000000 +0200
+++ /etc/hosts  2011-03-01 14:14:08.000000000 +0200
@@ -29,3 +29,5 @@
 # numbers but instead get one from your network provider (if any) or
 # from your regional registry (ARIN, APNIC, LACNIC, RIPE NCC, or AfriNIC.)
 #
+192.168.10.1            coffin.lan coffin
+192.168.10.10           striker.lan striker
```
Packet Filter

If you prefer <b>ipfw</b> it's about time to setup your firewall. If you use <b>pf</b> read on.

Have a look at my [pf.conf]() if you don't have your own.

Add to <b>/etc/rc.conf</b>:
```sh
# Enable PF
pf_enable="YES"
pflog_enable="YES"
```
If you change your pf.conf you can reload firewall with this command:
```sh
# pfctl -f /etc/pf.conf
```
 

</details>
<br>
<details>
  <summary><b>File systems configuration</b></summary>
<br>
<b>/etc/fstab</b>:

```sh
# Device                Mountpoint      FStype  Options         Dump    Pass#
/dev/gpt/swap0          none            swap    sw              0       0
/dev/ufs/root0          /               ufs     rw              1       1
/dev/ufs/var0           /var            ufs     rw              2       2
/dev/ufs/usr0           /usr            ufs     rw              2       3
/dev/ufs/ports0         /usr/ports      ufs     rw              2       4
/dev/ufs/home0          /usr/home       ufs     rw              2       5
/dev/cd0                /media/cdrom    cd9660  ro,noauto       0       0
/dev/da0s1              /media/flash    msdosfs rw,sync,noauto,longnames,-Lru_RU.UTF-8 0 0
```
As you can see I use UFS2 labels to mount partitions (parameter -L to newfs). If you forgot to label filesystems then you could boot FreeBSD installation media, choose "Shell" and create labels like this:
```sh
tunefs -L root0 /dev/ada0s1a
```
If you need quota support modify appropriate fs lines of fstab:
```sh
/dev/ufs/home0         /usr/home       ufs     rw,userquota,groupquota 2 5
```
But that's not all. Add enable_quotas="YES" to /etc/rc.conf and options QUOTA to your kernel config. Use edquota(8) to edit quota.
NFS exports

If you need help activating NFS support refer to the Handbook.

To see the list of NFS exported filesystems:
```sh
# showmount -e
``` 
To export UFS2 filesystem add to /etc/exports:
```sh
/usr/ports             -maproot=root -network 192.168.10.0 -mask 255.255.255.0
```
UFS2 Filesystem /usr/ports is exported to 192.168.10.0/24 read-write, remote root is mapped to local root.

To export ZFS filesystem /usr/ports of pool system, run:
```sh
# zfs set \
  sharenfs="-maproot=root -network 192.168.10.0 -mask 255.255.255.0" \
  system/usr/ports
```
To appy the changes reload mountd:
```sh
# service mountd onereload
```
Flash automount:

In case you need it you can enable flash automount by creating /etc/devd/umass.conf:
```sh
attach 100 {
        device-name "umass[0-9]+";
        action      "sleep 1; mount /media/flash";
};

detach 100 {
        device-name "umass[0-9]+";
        action      "umount -f /media/flash";
};
```
```sh
# mkdir /media/flash
# chmod 777 /media/flash
# service devd restart
```
</details>
<br>
<details>
  <summary><b>Logs and log rotation</b></summary>
<br>
<b>/etc/syslog.conf</b> defines system log files. <b>/etc/newsyslog.conf</b> defines rotation of the logs.

Create <b>/var/log/all.log</b>:
```sh
# touch /var/log/all.log
# chmod 600 /var/log/all.log
```
<b>diff -u /usr/share/examples/etc/syslog.conf /etc/syslog.conf</b>
```diff
--- /usr/share/examples/etc/syslog.conf 2008-09-27 00:22:12.000000000 +0300
+++ /etc/syslog.conf    2008-09-26 23:00:25.000000000 +0300
@@ -19,7 +19,7 @@
 #console.info                                  /var/log/console.log
 # uncomment this to enable logging of all log messages to /var/log/all.log
 # touch /var/log/all.log and chmod it to mode 600 before it will work
-#*.*                                           /var/log/all.log
+*.*                                            /var/log/all.log
 # uncomment this to enable logging to a remote loghost named loghost
 #*.*                                           @loghost
 # uncomment these if you're running inn
```
<b>/etc/newsyslog.conf</b> allready has entry to rotate <b>/var/log/all.log</b>. 
</details>
<br>
<details>
  <summary><b>Accounts information</b></summary>
<br>
User login classes are defined in <b>/etc/login.conf</b>. Run "<b>cap_mkdb /etc/login.conf</b>" after modifying the file. Changed default class to disable core dumps:

<b>diff -u /usr/share/examples/etc/login.conf /etc/login.conf</b>
```diff
--- /usr/share/examples/etc/login.conf  2008-09-27 00:22:12.000000000 +0300
+++ /etc/login.conf     2008-10-09 09:56:47.000000000 +0300
@@ -35,7 +35,7 @@
        :memorylocked=unlimited:\
        :memoryuse=unlimited:\
        :filesize=unlimited:\
-       :coredumpsize=unlimited:\
+       :coredumpsize=0:\
        :openfiles=unlimited:\
        :maxproc=unlimited:\
        :sbsize=unlimited:\
```
  
You should modify accounts database with pw utility. If you alter /etc/master.passwd file directly (i.e. when copy it from backup) run "pwd_mkdb /etc/master.passwd".

Setup defaults for pw ("standard" as default login class, nologin as the shell and Maildir structure premade):
```sh
# pw useradd -D -L standard -k /etc/skel -s /sbin/nologin
# cp /usr/share/skel/* /etc/skel/
# mkdir /etc/skel/Maildir
# mkdir /etc/skel/Maildir/cur
# mkdir /etc/skel/Maildir/new
# mkdir /etc/skel/Maildir/tmp
# chmod -R 700 /etc/skel/Maildir
```
  
pw has created its configuration file <b>/etc/pw.conf</b>. It has a number of useful options like uid ranges, password expiration settings, etc.

Edit <b>/etc/skel/dot.cshrc</b> and other files there. Have a look at my <b>dot.cshrc</b>.

Create your user:
```sh
# pw groupadd ross
# pw useradd ross -g ross -G wheel -m -s /bin/tcsh
```
  
Setup passwordless login

```sh
ross@client> ssh-keygen -t rsa
ross@client> scp .ssh/id_rsa.pub server.example.com:

ross@server> mkdir .ssh
ross@server> chmod 700 .ssh
ross@server> cat id_rsa.pub >> .ssh/authorized_keys
ross@server> rm id_rsa.pub
```
</details>

You can find default configuration files in <b>/usr/share/examples/etc</b>. For example, there is no <b>/etc/make.conf</b> on newly installed system. You can get one by copying <b>/usr/share/examples/etc/make.conf</b> to <b>/etc</b> and editing it.