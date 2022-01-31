# TL;DR
Your role includes.
1. changing passwords on all accounts and disable uneeded acounts
2. turn on command logging and backup files to diff incase of breach
4.  upgrade kernel
5.  install random security software
6.  look for privilege escalation vectors and remove them. 
7. monitor for signs of intruders

Warning: something written in this guide is probably wrong. when something doesn't work right google the error message or ask questions. 

# change default password for default login
passwd
# open a root shell and change root password and name
```
sudo -i 
passwd
usermod -l \<newname> \<oldname># change default password for default login
passwd
```




# open a root shell and change root password
sudo -i 
passwd
usermod -l \<newname> \<oldname>-   

## make a wheel group and backup admin account
Using sudo will be restricted to wheel group
```
groupadd wheel
```


_Restrict the use of sudo to the wheel group by configuring **/etc/sudoers**._ _Use **visudo** and uncomment the following:_
```
wheel ALL=(ALL) ALL  
```

_Restrict use of su with pam._ _Uncomment or add the following line to **/etc/pam.d/su**:_
```
auth		requirement	pam_wheel.so group=wheel
```

_while root create an admin account:_
```
useradd -mg wheel <admin>
passwd <admin> 
exit
# login as admin and restrict root login and su to <admin>
sudo -i -u <admin>
sudo passwd -l root 
sudo chown <admin>:wheel /bin/su
```
\*Use sudo -i -u adminname when performing admin tasks!_


## Restrict Login Access
edit the file 
/etc/pam.d/system-login

```
# Set a delay upon authentication failure
# Lock out a user after 3 repeated failed attempts
auth optional pam_faildelay.so delay=4000000
auth required pam_tally2.so deny=3 unlock_time=600 onerr=succeed file=/var/log/tallylog
```

# Limit processes run by users
> /etc/security/limits.conf

```
* soft nproc 100
* hard nproc 200
```


## prexisting account check

### list users 
list all 
```
getent passwd 
```
list just normal users
```
getent passwd {1000..6000}
```
pay special attention to output of second command. stray accounts not needed for critical services should be locked
###  check empty passwords
```
sudo awk -F: '($2 == "") {print}' /etc/shadow
```
If you see any empty passwords change them with commands below
### check for accounts that are root 
> awk -F: '($3 == "0") {print}' /etc/passwd

see any lock account. 

Delete user
```
userdel user_name
```
lock account 
```
usermod -L <user>
```
change password
 ```
passwd <user>
 ```


# /etc/group

> Format: admin(name):*(idk):80(idk):root(the users in it)

Look at group name and user account names. Key groups to look for: admin, wheel, sudo, nopasswdlogin (?). The users for each of these should ONLY be root and your account (and nothing for nopasswdlogin)
> fixing: ( ie leave testuser only in 'root' group and it's primary 'testuser' group):

```
 $ sudo usermod -G root testuser
``` 


> remove user from one group 

```
sudo gpasswd -d testuser root
Removing user testuser from group root
```

/etc/sudoers

Format: %admin ALL=(ALL) ALL

In this case, the admin group has permissions to do whatever it wants. We should probably keep things in the sudoers file to a minimum.

stare at this for a bit and look to see if anything looks weird. 

## quick check
look for keys in the ssh folders and delete them.
~/.ssh/ (and /root/ not just /home)+
# breathe 
Okay we did likely 80 percent of the work. Pat yourself on the back. 


# get baseline stuff to diff later.
``````
mkdir -p ~/logs/initial (make a folder in your home directory called logs, inside it make a folder called initial)
``````

``````
cd ~/logs (change working dir to that new folder)
``````

``````
sudo cp -R /var/log/ . (copy the whole log folder!)
``````

``````
mv log initial (rename the newly copied folder to something different (preferably that indicates time stamp) yay nested parens)
``````
``````
diff -r /var/log initial (compare the current /var/log dir with your copy)
``````


Listing running processes:
``````
ps ax (list all running procs)
``````

``````
ps aux username (specify a user to see their procs)
``````
redirect into a file so you can diff later
> ps ax > psdiff

setuid binaries
``````
find / -perm -u=s -type f 2>/dev/null
``````
finding new ones is bad. 
compare like with GTFO bends while you're at it
> https://gtfobins.github.io/#+suid

> fix: chmod u-s /path/to/file.
# Restrict CRON Usage
check the cron tab of the users you're aware of 
```
crontab -l -u USERNAME
```

disable cron for everyone but you 
```
echo $(whoami) >> /etc/cron.d/cron.allow# echo ALL >> /etc/cron.d/cron.deny
```


# turn on command line logging 

Edit **/etc/profile** and add the following lines to the bottom of the file:
```none
# command line audit logging
function log2syslog
{
   declare COMMAND
   COMMAND=$(fc -ln -0)
   logger -p local1.notice -t bash -i -- "${USER}:${COMMAND}"
}
trap log2syslog DEBUG
```
Edit **/etc/rsyslog.conf** and add the following lines to the bottom of the file:
```none
# command line audit logging
local1.* -/var/log/cmdline
```


```none
/etc/init.d/rsyslog restart
```

The audit logging will be visible under **/var/log/syslog** and **/var/log/cmdline** 

## update kernal )(debian)
debian instrutions 
>sudo apt update

get list of linux kernals to update

> aptitude search linux-image

actually install the kernal 

>`sudo aptitude install linux-image-4.10.0-trunk-amd64-unsigned linux-headers-4.10.0-trunk-amd64`
## update kernal (redhat)
> \#**yum update kernel**

you have to restart after a kernel update
> sudo shutdown -r now

# the great software install phase


# if you have a web server
install mod_security 
> https://github.com/SpiderLabs/ModSecurity/wiki/Reference-Manual

# ssh 

install fail2ban
fail to ban
https://linuxhandbook.com/fail2ban-basic/

### iptstate
> iptstate

can be used to watch traffic possibly 

# tripwire
https://www.techrepublic.com/article/how-to-install-and-use-tripwire-to-detect-modified-files-on-ubuntu-server/

monitors for changes in files


# ossec

https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-ossec-security-notifications-on-ubuntu-14-04

# php 
https://tkacz.pro/nginx-and-php-configure-php-ini-file/


# patrol route (this is your idle animation)
> who

shows who's logged in. If you see someone that isn't you panik. Panik very fast. 

> check for weird files in the /tmp folder

this is also bad news 
-   /var/log/auth.log --- logs authorization attempts
-   /var/log/daemon.log --- logs background apps
-   /var/log/debug --- logs debugging data
-   /var/log/kern.log --- logs kernel data
-   /var/log/syslog --- logs system data
-   /var/log/faillog --- logs failed logins
look at logs 

check connections to the machine
```
netstat –nap (Linux),
netstat –na (Solaris),
```
look at processes each user is running. 
ps aux username


use ls -al to check for hidden folders.

check diffs of these files
 > /etc/passwd & /etc/shadow & /etc/sudoers

repeat. check if anyone needs any help. go get some water. then come back and do it again. 
 bad news if you see anything here.


  
find all immutable files, 
I have litterally no clue what this does. 
>find . | xargs -I file lsattr -a file 2>/dev/null | grep ‘^….i’

use in /etc/, /tmp/, ~/ etc.

but I assume it's bad if it finds stuff
 fix immutable files
> chattr -i file










# idenitifying services running 
## list services 
Listing all services could potientially be information overload but skimming it and disabling anything that seems not needed could be useful 
> systemctl list-unit-files --type=service

how to disable 
 > systemctl disable httpd.service
	
## listening ports 
This will show all ports listening on the machine. you will use this information to set host firewall rules. find the ports the machine's critical services are running on and take note of it. 

> netstat -tulpn

> ss -tulpn




# breathe 

We made some cool progress and did the major stuff. relax for like 15 seconds. then move on . our next goal is to get some baseline knowledge of what the system is like before getting hacked. 


# round up of comman service stuff to look at 
# ssh stuff

 **/etc/ssh/sshd_config**
```
PermitRootLogin no # disables root loginMaxAuthTries 3 # limits authentication attemptsPasswordAuthentication no # disables password authenticationPermitEmptyPasswords no # disables empty passwordsX11Forwarding no # disables GUI transmissionDebianBanner no # disbales verbose bannerAllowUsers *@XXX.X.XXX.0/24 # restrict users to an IP range
```


Check these files and make sure they are blank / only have information that you put there. These store public keys that are authorized to log in as this user WITHOUT password.
> ~/.ssh/authorized_keys

>/root/.ssh/authorized_keys

if you see stuff in this directory of the user home directory problably want to delete. 







# monitor logs
-   /var/log/auth.log --- logs authorization attempts
-   /var/log/daemon.log --- logs background apps
-   /var/log/debug --- logs debugging data
-   /var/log/kern.log --- logs kernel data
-   /var/log/syslog --- logs system data
-   /var/log/faillog --- logs failed logins

# final set of less importnat stuff 
check host files and clear any uneeded stuff :/etc/hosts 
unalias -a remove aliases 



maybe try to find an ids to install 



# todo 


