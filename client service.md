-   Turn on text only email reading if email is in play
-   Microsoft Security Essentials free for SMB and home users so White Cell should be ok with it and hands down the best AV (IMHO)
-   They have firewalls too! (nudge nudge)
-   On windows systems install PeerBlock, it's a very small software package that does IP blocking for windows and supports LARGE IP lists (like every IP but my subnet) and supports egress    
-   On Linux remove all remote access options. It's a client, it doesn't need SSHd

# TL;DR
1. remove uneeded services 
2. set up host firewalls
3.  help with threat hunting

# remove uneeded services
We will begin by installing nmap locally on the machine we're testing. 

## install nmap debian 
> sudo apt update
>sudo apt-get install nmap
## install nmap redhat
> dnf install nmap


scan the host

```
nmap -p- -Pn -A -sC -sV localhost > nmapoutput.txt
```

one of these programs should be installed and can be used to read the files
```
less/ more/ cat nmapoutput.txt
```

Shelf this file for a bit. We're going to configure the firewall first. 

We have 2 possible programs we can use for configure local firewall on linux iptables and UFW. UFW is simpler. Iptables will only be used if UFW can't be installed for some reason. 

# ufw
# redhat install instructions
UFW isn’t available on the CentOS repository. We need to install the EPEL repository on our server.

```bash
sudo dnf install epel-release -y
```

Now install and enable UFW:

```bash
# install
sudo dnf install ufw -y

```

# debian install 
 just do apt-get install ufw

 # setup
``` shell
# enable
sudo ufw enable
```

check it is inabled. 

```bash
sudo ufw status
```
To disable UFW, you’ve to run this command:

```bash
sudo ufw disable
```
use this is something breaks while you're configuring the firewall

By default, the UFW block all incoming traffic and allow all outgoing traffic. You can run the below commands to set the default policy:

```bash
sudo ufw default deny outgoing
sudo ufw default deny incoming
```



We can easily add rules in UFW firewall. Let’s open HTTP (80) port:
```

sudo ufw allow http
# or
sudo ufw allow 80

```

We can deny any incoming and outgoing traffic to any port:

```bash
sudo ufw deny 8081
```
shouldn't really need to do this. since we're denying everything by default. Just allow the ports for the critical services.

We can check the running ports easily:

```bash
# method 1
sudo ufw status
# method 2
sudo ufw status numbered
# method 3
sudo ufw status verbose
```


##  Delete Firewall Rules

Like addition, we can easily remove rules too. Here’s the example:

```bash
sudo ufw delete allow http
sudo ufw delete deny 8081
```

We can also delete a rule by its number. Run `sudo ufw status numbered` and you’ll see the number beside the rules. Then you can delete like this:

```bash
sudo ufw delete 5
```

To remove all rules:

```bash
sudo ufw reset
```
 Can also be done if you mess up.


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


# check our work

Redo the nmap scan and hopefully the extra services are no longer open.  If everything looks good we can work 


# debloating services


```
`# chkconfig --list | grep '3:on'`  
To disable service, enter:  
`# service serviceName stop  
# chkconfig serviceName off`
# services --status-all
```


Modern Linux distros with systemd use the systemctl command for the same purpose.
Print a list of services that lists which runlevels each is configured on or off
```
# systemctl list-unit-files --type=service
# systemctl list-dependencies graphical.target
```
Turn off service at boot time

```# systemctl disable service
# systemctl disable httpd.service
``````
Start/stop/restart service

``````
# systemctl disable service
# systemctl disable httpd.service
``````
Get status of service
``````
# systemctl status service
# systemctl status httpd.service
``````


list of services that should be disabled if not required to be up
1. FTP
2. VNC 
3. RDP 
4. SSH 
5. Telnet 


# windows 


