-   Upgrade your kernel ASAP +
-   Fail2Ban
-   If ($PHP) then shoot.self; (Fix php.ini)
-   SETUID
-   Create a process list file so IR can diff it +
-   Remove any unused users or services+
-   IPTSTATE is like TCPview for Linux, use it. love it.
-   GRSEC IF YOU HAVE TIME, custom kernels take time to compile but, it's fun to watch Red Teamers attempt privilege escalation on older kernels. -
-   Turn off the ability to change grsec settings via sysctl-
-   Turn on EXEC logging+
-   Watch the audit log for signs of escalation attempts+
-   File Integrity logging pays dividends:-
-   Tripwire-
-   OSSec (has pre-configurations for most *nix)-
-   Nothing new should enter here without you knowing:+
-   /tmp/ (new files or binaries in here are bad news)+
-   .hidden directory is a common place to put stuff +
-   crontab for all users+
-   ~/.ssh/ (and /root/ not just /home)+
-   /etc/
-   Know all SetUID binaries and watch for new ones -
-   Final all 'immutable' files
-   find . | xargs -I file lsattr -a file 2>/dev/null | grep '^....i' +
-   'chattr -i file' to change it back+ 
-   Doing this on / takes a long time, point it where it counts: /etc/, ~/, /tmp/   etc.. etc..+


# stuff I wrote 
alright peeps time for linux harden guide. main goals: decrease attack surface area, monitor for signs of intrution, keep services running. 

Warning: something written in this guide is probably wrong. when something doesn't work right google the error message or ask questions. 

step one make accounts and do our rounds to see what is going on.
# making an account
first thing things first you are going to want to create your own account on the machine. you will likely be given a root account but it is good practice not to constantly use the for everything.
```
# adduser <username> sudo
```

# list users 
###  check empty passwords
```
sudo awk -F: '($2 == "") {print}' /etc/shadow
```
If you see any empty passwords change them with commands below

# todo flesh out
# /etc/group

Format: admin:*:80:root

Look at group name and user account names. Key groups to look for: admin, wheel, sudo, nopasswdlogin (?). The users for each of these should ONLY be root and your account (and nothing for nopasswdlogin)

/etc/sudoers

Format: %admin ALL=(ALL) ALL

In this case, the admin group has permissions to do whatever it wants. We should probably keep things in the sudoers file to a minimum.
### check for accounts that are root 
> awk -F: '($3 == "0") {print}' /etc/passwd

see any delete or change passwd. 


list all 
```
getent passwd 
```
list just normal users
```
getent passwd {1000..6000}
```
pay special attention to output of second command. stray accounts not needed for critical services should be deleted

also noted any user without /usr/bin/nologin as their shell should definitely have their password changed at a minimaum or be deleted. 

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


# iptables
 this will be the local firewall software for our box. 
this will let machine talk on the loopback adrress and is fine
> sudo iptables -A INPUT -i lo -j ACCEPT

allow traffic on a specific port. You're going to allow the ports used by your critical services 
```
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
```

The port number you've allowed follows dport. possibly we need to change tcp if the service running doesn't use  the tcp protocal. 

Command to drop all other traffic that isn't allowed 
```
sudo iptables -A INPUT -j DROP
```

Now take a moment and see if anything is on fire. If so Your priority is keeping services up so delete the drop rule and try to figure out what else you need to allow through the firewall.

## delete a rule 
see rules
```
sudo iptables -L --line-numbers
```
delete unwanted rules 
```
sudo iptables -D INPUT <Number>
```


## save changes
debian
```
sudo /sbin/iptables–save
```

redhat 
```
sudo /sbin/service iptables save
```


# breathe 

We made some cool progress and did the major stuff. relax for like 15 seconds. then move on . our next goal is to get some baseline knowledge of what the system is like before getting hacked. 
# easy get baseline stuff to diff later.

mkdir -p ~/logs/initial (make a folder in your home directory called logs, inside it make a folder called initial)

cd ~/logs (change working dir to that new folder)

sudo cp -R /var/log/ . (copy the whole log folder!)

mv log initial (rename the newly copied folder to something different (preferably that indicates time stamp) yay nested parens)

diff -r /var/log initial (compare the current /var/log dir with your copy)



Listing running processes:

ps ax (list all running procs)

ps aux username (specify a user to see their procs)

redirect into a file so you can diff later


setuid binaries
find / -perm -u=s -type f 2>/dev/null
finding new ones is bad. 

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




# Restrict CRON Usage
```
crontab -l -u USERNAME
```
check the cron tab of the users you're aware of 
disable cron for everyone but you 
```
echo $(whoami) >> /etc/cron.d/cron.allow# echo ALL >> /etc/cron.d/cron.deny
```

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

## update kernal 
debian instrutions 
>sudo apt update

get list of linux kernals to update

> aptitude search linux-image

actually install the kernal 

>`sudo aptitude install linux-image-4.10.0-trunk-amd64-unsigned linux-headers-4.10.0-trunk-amd64`

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

The audit logging will be visible under **/var/log/syslog** and **/var/log/cmdline** and will look like this:

maybe try to find an ids to install 


# if you have a web server
install mod_security 
> https://github.com/SpiderLabs/ModSecurity/wiki/Reference-Manual
# patrol route 
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

look at processes each user is running. 

use ls -al to check for hidden folders.

repeat. check if anyone needs any help. go get some water. then come back and do it again. 
 > /etc/passwd & /etc/shadow & /etc/sudoers


 bad news if you see anything here.

  find all immutable files, there should not be any
I have litterally no clue what this does. 
>find . | xargs -I file lsattr -a file 2>/dev/null | grep ‘^….i’

use in /etc/, /tmp/, ~/ etc.

but I assume it's bad if it finds stuff
 fix immutable files

> chattr -i file
# todo 


