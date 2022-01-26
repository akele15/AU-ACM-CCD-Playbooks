Monitor OUTBOUND connectionsMonitor OUTBOUND connections
-   Egress and Ingress filtering
    
-   IPv6 OFF (Unless required)
    
-   deny any any is your friend
> https://github.com/google/capirca


maybe useful


# stuff I wrote 

o Know how to implement the Intrusion Prevention System (or related) features on your device

o Disable unnecessary ports

o Know how to do a console-based password reset
1. Change the default _admin_ password before connecting the firewall to any network.

```
1.  Change the default _admin_ password before connecting the firewall to any network.
2.  Enable admin profiles and groups to limit access to other administrators.
3.  Enable password profile to enforce tough passwords. Change passwords regularly on a scheduled interval.
4.  Set up notifications for system and configuration log messages that indicate modifications of the firewall's operational parameters (these notifications can be sent via email, Syslog, and/or SNMP traps)
5.  Set an SNMP community string that is not easy to guess and is preferably not shared by other network equipment.
6.  Only enable SNMP on internal interfaces that you need them on.
7.  Interface management profiles: do not enable ping, ssh, htttp/s, and other services on the firewall interfaces that don't require them. Note that the "Response pages" may not be necessary on certain interfaces. These are the pages the firewall uses for URL filtering notification, virus block messages, SSL VPN, and captive portal.
8.  Set up IP-based access control on all interfaces that have management profiles including the management interface.
9.  Place the management interface into a management VLAN that limits access to authorized personnel. Do not turn on management profiles on interfaces that are accessed by non-authorized personnel.
10.  Monitor system and configuration logs on a regular basis to monitor for unauthorized login attempts or changes to configuration settings.

```