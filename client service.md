-   Turn on text only email reading if email is in play
-   Microsoft Security Essentials free for SMB and home users so White Cell should be ok with it and hands down the best AV (IMHO)
-   They have firewalls too! (nudge nudge)
-   On windows systems install PeerBlock, it's a very small software package that does IP blocking for windows and supports LARGE IP lists (like every IP but my subnet) and supports egress    
-   On Linux remove all remote access options. It's a client, it doesn't need SSHd


# goals: uninstall all the crap on all the machines that's uneeded. 

fail to ban
https://linuxhandbook.com/fail2ban-basic/


service likely needed to be disabled.
1. FTP
2. VNC 
3. RDP Brute
4. SSH Brute
5. Telnet Brute


# secure needed services
## Check the business required services for admin panels and default passwords. Check the service version and update.

We need to find out:

1.  Are there any default credentials associated with the service and how do we change them?
2.  Where are the configuration files for the service stored?
3.  Is the version of the service we are running vulnerable and can we update it?
4.  Where does the service output log files and how do we configure where the log files go?

Once we have that information, we should:

1.  Change the default passwords and turn off any anonymous logins or default credentials.
2.  Check the configuration files for anything we might have missed in documentation.
3.  Update the service where possible.
4.  Make note of where the service outputs logs so we can monitor them for potential indicators that something is wrong. If there are no logs by default try and turn some on.
5.  SCP a copy of the log files to our JumpBox so we can see if they have been altered later.

# host firewall 
## drop all ipv6
 ip6tables -P INPUT DROP  
 ip6tables -P OUTPUT DROP  
 ip6tables -P FORWARD DROP


 https://www.ossec.net/download-ossec/