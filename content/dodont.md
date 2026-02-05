---
title: 10 Do's and Don't for FreeBSD
subtitle: 
hidden: true
---
_by ShelLuser ([original link](https://forums.freebsd.org/threads/10-dos-and-dont-for-freebsd.65618/))_

FreeBSD is plain and simple my all time favorite Unix operating system. And yes: technically speaking it's Unix-like but that's mostly due to licensing concerns. If you look at the backwards compatibility aspects (see */usr/src/sys/amd64/conf/GENERIC*, the *COMPAT_FREEBSDxx* options), the slow pace of changes (we don't get "exciting" and "OS changing" features with every update) and the display of high respect for current standards (this is just me, but I think that *pkg_add* (old) vs. `pkg add` (current) is a good example and also extremely well thought off)... so if you look at all that then I think you can agree that when solely looking at the environment then FreeBSD has everything it needs to be considered a true Unix operating system.

Another thing is that the system is like elastic. You can bend and shape it in any way you want. In both good and bad ways. So I figured.. why not a small collection of very general things which you might want to pay attention to while working with this system?

## 1. Don't mix ports and binary packages

This is an aspect which has confused many people and I'm sure it will continue to do so for many more in the future. In short: either install your software through the ports collection (so building it using /usr/ports, either using the regular build commands or a utility of some sort) or install your software using merely pkg. So, for example: `pkg install mystuff`.

Don't do both!

Why? Because it has the potential to seriously disrupt your entire system. If you upgrade your system and suddenly see packages which you wanted to keep get automatically deinstalled then this could be a clear sign of trouble. If you have upgraded your system and a program you're using suddenly spits out error messages about missing libraries or missing functions, library symbols or whatever more. That too could be a clear signal that you broke things. The reason this is happening is because binary packages were build with default settings and the expectation of the same thing for all their dependencies.

What do you think could happen if you installed a notepad system using the package manager while you installed a required database library using the ports collection, while also exchanging MySQL for PostgreSQL? The application itself might still work. But all automated backups are likely to fail miserably because chances are high that it would depend on mysqldump for that, a program which is no longer available on your system. So the next time you experience a crash and figure that you came fully prepared then you might be in for a very nasty surprise.

## 2. Don't edit 'default' files

Have you ever wondered which processes FreeBSD would start on its own? Or maybe what about the boot stage: would FreeBSD automatically load any kernel modules? These questions can be easily answered by taking a closer look at /etc/defaults/rc.conf and /boot/defaults/loader.conf.

Whatever you do: do not edit these files!

First, the most obvious reason: this is a file which is a part of the base system. Therefor it could get updated at any given time, fully overwriting whatever changes you made. If something suddenly stopped starting after you upgraded your system then... maybe this is why.

But the other reason is because there is no need. Any changes which you want to make should be applied to the main official file instead. So /etc/rc.conf and /boot/loader.conf respectfully. The default files are at their best as research material. It's not merely to tell FreeBSD what to do by default, it can even help us out!

Example: maybe you don't enjoy Sendmail and wish to use another MTA. I don't know, maybe mail/postfix? As you probably know Sendmail is a part of the base system, so it would also have set up several settings to cater for that. But which ones....

Well, easy:
```sh
peter@unicron:/etc/defaults $ grep sendmail rc.conf
mta_start_script="/etc/rc.sendmail"
# Settings for /etc/rc.sendmail and /etc/rc.d/sendmail:
sendmail_enable="NO"    # Run the sendmail inbound daemon (YES/NO).
sendmail_pidfile="/var/run/sendmail.pid"        # sendmail pid file
sendmail_procname="/usr/sbin/sendmail"          # sendmail process name
sendmail_flags="-L sm-mta -bd -q30m" # Flags to sendmail (as a server)
sendmail_cert_create="YES"      # Create a server certificate if none (YES/NO)
#sendmail_cert_cn="CN"          # CN of the generate certificate
sendmail_submit_enable="YES"    # Start a localhost-only MTA for mail submission
sendmail_submit_flags="-L sm-mta -bd -q30m -ODaemonPortOptions=Addr=localhost"
sendmail_outbound_enable="YES"  # Dequeue stuck mail (YES/NO).
sendmail_outbound_flags="-L sm-queue -q30m" # Flags to sendmail (outbound only)
sendmail_msp_queue_enable="YES" # Dequeue stuck clientmqueue mail (YES/NO).
sendmail_msp_queue_flags="-L sm-msp-queue -Ac -q30m"
                                # Flags for sendmail_msp_queue daemon.
sendmail_rebuild_aliases="NO"   # Run newaliases if necessary (YES/NO).
```
Everything you might wanted to know about FreeBSD's default Sendmail usage but were too afraid to ask ;) Of course an easier way to grep only the entries we need would be: `grep -e "^sendmail.*=\"YES" rc.conf`.

So basically: these default files should only be used to look up specific settings and/or behavior. And they can truly be an invaluable source for that. When I set up my own private SSL certificates for Sendmail I didn't want it to continuesly try to build its own useless stuff in /etc/mail/certs. And this is how I found out to stop that behavior.

## 3. Don't mess with /etc/crontab

As you might know /etc/crontab is the main Cron configuration file so it would be the obvious place to set up our own entries for it, right? Well... not so much. Although not necessarily wrong there are some risks involved. Just like with most other files in /etc you're basically messing with a system file which could undergo changes during upgrades.

But the most compelling reason is that any changes you make would not be immediately picked up by the system. You'd have to reload the crontab as well. Why all the extra bother if you don't have to?

Instead it is advised to use the crontab command. Do you need some jobs run as the bind user? Just use `# crontab -u bind -e` and done. Do you need some specific system settings? Install those as root: `# crontab -e` and done. You only need to worry about the 5 time fields (m h dom m dow) and the command(s) to run and that's it.

After you're done editing the crontab will be automatically installed and activated. No need to manually mess with service reloading. And as an added bonus: this way you also don't have to worry about accidentally removing important system entries whenever you no longer need certain jobs and want to remove those.

## 4. Don't mess with /etc/passwd and /etc/groups either!

Since we're focusing ourselves on /etc a little here is another one ;) I'll be perfectly honest: I sometimes also do this myself, but fortunately very seldomly.

If you insist on editing these files manually at the very least rely on `vipw` and/or vigr. There are many advantages for that. First it will apply a lock on the file so that you can't accidentally mess things up (for example: if another process would try to change a password and you finish making your changes after that you'd effectively undo the password change, which could lead to very nasty results). And another aspect is that any made changes will be automatically processed if they have to. Better yet: vipw will also apply some syntax checking so if you're at risk of trashing your entire login process because of a messed up edit then this would save you from that.

What's that? You don't like using vipw because it always uses vi? So change that! All it does is honor the EDITOR environment variable. Just point that to something else and you'll be home free.

But there are more reasons not to do this:
```sh
operator:*:2:5::0:0:System &:/:/usr/sbin/nologin
```

Which entry was the home directory again? Wait, I know! I think it was the empty entry between those two colons, let's change that...

vs:

`# chpass operator` (note: *chsh* also works):
```sh 
#Changing user information for operator.
Login: operator
Password: *
Uid [#]: 2
Gid [# or name]: 5
Change [month day year]:
Expire [month day year]:
Class:
Home directory: /
Shell: /usr/sbin/nologin
Full Name: System &
Office Location:
Office Phone:
Home Phone:
Other information:
```
Everything you might want to edit in an impossible to miss way. This is how you edit /etc/passwd without the risks of making any accidentally dumb mistakes (like the one I hinted at above). For more information check out chpass(1). This is also the best way to change your own settings, even as a regular user. Don't like your current shell? chsh to the rescue!

And there's something even better: pw. I'll admit: the sheer complexity of this command also set me back many times. I mean, just look at the massive sea of entries when you check pw(8). Honestly: I have opened that manual page many times in the past only to glance at it, quickly close it and then run either vipw and/or vigr and left it at that.

If you're new to pw it is very easy to get totally overwhelmed and/or confused at first, and that is definitely not your fault.

But despite its difficult appearance it's actually extremely easy to use. Want to delete a user? # `pw userdel unwanteduser`, and done. Or what about a group? Fortunately logic is heavily applied here as well, so you can simply swap out userdel for groupdel.

Need to add someone to the wheel group? This means you need to modify a group: groupmod. So: `pw groupmod wheel -d currentmember -m newmember`. This would delete currentmember and add newmember.

Bottom line: there really is no need to manually edit /etc/passwd and/or /etc/group at all. At the very least rely on vipwd.

## 5 Reconsider the removal of any options from your customized kernel configuration

Before you wonder... I'm doing the same thing myself, even made a sport out of this. Here is the top of /root/kernels/unicron.conf which is the symlink target for /usr/src/sys/amd64/conf/UNICRON:

```sh

include GENERIC
ident OMICRON

nooptions       QUOTA
nooptions       GEOM_RAID
nooptions       INCLUDE_CONFIG_FILE

# Floppy
nodevice        fdc

# ATA
nodevice        ahci, mvs, siis

# SCSI
nodevice        ahc, ahd, esp, hptiop, ispfw, mps, mpr, sym
nodevice        trm, adv, adw, aic, bt, isci
nooptions       AHC_REG_PRETTY_PRINT
nooptions       AHD_REG_PRETTY_PRINT
#nodevice       ispfw, ncr
```

(I know that I should rename the kernel as well, but.. that's for a later time)

This was manually edited against GENERIC, manually kept updated and before every major upgrade I carefully compare the new GENERIC with this file.

So here's the problem: What if my current hardware would suddenly fail and as an emergency replacement I got hold of a Tekram DC395U/UW/F DC315U adapter? ;) Well, simple: I can install the hardware but as you can see above my current kernel would totally ignore it because I manually removed support for that (notice the trm entry above?).

Basically: in order to use new hardware I'd have to rebuild a whole new kernel. Sounds like the perfect strategy for a server which hundreds of users depend on! :sssh:

Now, don't get me wrong here. These things can be fun to do. I also enjoy tinkering with my system (see above) and it also gives me a good sense of satisfaction that my kernel is pretty streamlined. Not to mention the fact that setting all of this up has taught me a lot about the kernel and the source tree itself.

But always heed the risks.

And in case you wonder... I definitely enjoy tinkering but obviously I also came prepared ;) On my system I maintain /boot/kernel.gen as well which contains a copy of a GENERIC kernel with all the available modules (I usually rely on the latest kernel.txz). And my /boot/loader.conf contains this entry: kernels="kernel kernel.old kernel.gen". So if things do go dramatically wrong then I always have a GENERIC failsave available.

## 6. Don't change the root shell to something else

/bin/csh is the shell for the root user while everyone else usually gets /bin/sh, and this is done for a very good reason. Although many people use the root user and its privileges extremely casually you should never underestimate the amount of damage which a small mistake as root can do to your system. Don't treat root sessions as something casual!

This is one of the reasons why using a different shell can be a very good idea: because if you don't often use this then it will force you to think about your actions:
```sh
peter@unicron:/home/peter $ for a in a b c; do mkdir -p test/$a; done
peter@unicron:/home/peter $ file test
test: directory
peter@unicron:/home/peter $ csh
% for a in a b c; do mkdir -p test/$a; done
for: Command not found.
a: Undefined variable.
```
Of course there's more. The C shell also has a strong emphasis on interaction. That 'for' command I showed above? The C shell is perfectly capable of that as well, but in a much more interactive way:
```sh
% foreach a ( a b c )
foreach? mkdir -p test2/$a
foreach? end
% file test2
test2: directory
```
See what I mean? Instead of simply typing out one huge command on a single line this allows you to cut the whole thing up into smaller pieces, also allowing you to check for any typoes in the single command(s) and to (re)think about each of those as well.

Sure there's also a downside here: if you have to repeat this set of commands a few times then this is obviously not the best of options. But why not write a temporary shell script and use that instead?

## 7. Don't use the root user all the time

Many people claim that Unix is a very secure operating system which can hardly be affected by any nastiness such as viruses. Although there is definitely some truth to that the underlying reasons for this don't necessarily have much to do with Unix. See, under normal circumstances you wouldn't be logged on as root and as a direct result of that....
```sh
peter@unicron:/home/peter $ rm /boot/kernel/*
override r-xr-xr-x  root/wheel uarch for /boot/kernel/agp.ko? y
rm: /boot/kernel/agp.ko: Permission denied
override r-xr-xr-x  root/wheel uarch for /boot/kernel/coretemp.ko? y
rm: /boot/kernel/coretemp.ko: Permission denied
peter@unicron:/home/peter $ rm -rf /boot/kernel/kernel
rm: /boot/kernel/kernel: Permission denied
peter@unicron:/home/peter $ echo "rm -rf ~/*" >> /etc/profile
/usr/local/bin/ksh: cannot create /etc/profile: Permission denied
peter@unicron:/home/peter $ echo "/home/peter/bin/passwordh4x >> /home/peter/ >> /root/.profile
/usr/local/bin/ksh: cannot create /root/.profile: Permission denied
```
'Toegang geweigerd' aka: 'Permission denied'

I'm actually using Windows as a regular (non-administrator) user and as a result I don't have access to most system environments. Some ransomware trying to encrypt c:\windows\system32 on my system? Best of luck with that! :p

Back to FreeBSD: part of keeping your system secure is not to use excessive options if you don't have to. And root is an extremely powerful user because it basically has little to no restrictions. A mistake as root can have much bigger consequences than those of a regular user. Stay safe...

## 8. /var/backups is a thing

Now, sure, you should not rely too much on automated "out of the box" backups, but if certain problems do occur then definitely check out this location first. Did something trash your package database? You might find the solution here. Were you too quickly with that pw command and removed an important account? This might be a quick way out too.

Of course you shouldn't overly depend on this either but this system very slick. It is powered by the Periodic system and actually quite intelligent too. For example: although it can keep a "roulation backup" of your files (with this I mean that it keeps several retention copies) it won't if there is no need. For example: /var/backups/aliases.bak dates from October last year on my system. And there's also an aliases.bak2 dating from July last year. Guess what? Last time I applied changes to the /etc/aliases file were.. you might have guessed it: around July and October last year ;)

## 9. Check system integrity using /etc/mtree

Did you know that FreeBSD includes a full fledged intrusion detection system out of the box called mtree? Now you do ;)

Keep in mind that the specifications in /etc/mtree only track directory permissions and not those of the individual files. So if you want to use the files in /etc/mtree then you also might want to use the -e parameter to exclude any file checks:
```sh
peter@unicron:/# mtree -e < /etc/mtree/BSD.root.dist
./etc/autofs missing
./etc/bluetooth missing
./etc/ppp missing
```
Guess what? It's correct about that. If I wanted to I could tell mtree to fix these issues automatically using the -u option.

This can be the perfect way to check if no one messed with some directory permissions:

```sh
peter@unicron:/var# mtree -e < /etc/mtree/BSD.var.dist
db/ports:
        permissions (0755, 0775)
```
Another very well spotted change.

I actually set this up as an extension of #7; very often I don't build ports as root but using my normal user account. This permission allows any user within the wheel group to modify the options of a port without having to elevate their permissions.

Want to learn more about mtree (it can do tons more)? Then you might enjoy my small mtree guide.

## 10. What works for me doesn't have to work for you!

All these items are basically best practices but definitely not mandatory rules. Always keep in mind that every situation is most often unique and when it comes to security, configuration and an operating system there is no such thing as a "one size fits all" solution.

Sure there are risks involved when manually firing up vi to edit /etc/passwd, but not so much if you're in single user mode. Running a fully customized kernel definitely has its quirks if you need to replace hardware, but how often would you do that on a laptop anyway? It's cool to be able and check permission defaults, but what if you applied better permission settings specific for your own environment? Then you probably don't care about /etc/mtree at all.

"Your milage may vary" as the saying goes.

And there you have it, 10 regular do's and dont's for FreeBSD. Hope this was useful for some of you.