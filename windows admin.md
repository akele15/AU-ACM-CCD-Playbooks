# TL;DR

role
1. change passwords on admin account. 
2. disable uneeded accounts
3. disable uneeded service
4. idk something about active directory
5. install software needed to manage the machine.

-   Event Viewer is your friend
-   Autoruns is your friend
-   Process Explorer and TCP View are your friend
-   OSSEC works for windows too 
-   (agent only, must talk to a Linux server for reporting)
-   Change passwords and fast! (Automate if possible)
-   Remove unused users and services
-   Turn your firewall on and REMOVE EXCEPTIONS
-   Turn off Teredo
-   Program one:
-   AutoIt (make a binary to do it faster)
-   Download one:
-   http://bit.ly/bulkpasswordcontrol (AD only - not local)
-   Advantage: pseudo random passwords
-   Built in one:
-   dsquery user ou=Users,dc=testlab,dc=net | dsmod user -pwd RedTeamSucks! -mustchpwd yes
LAPS for local admin passwords (Not built in, but it is Microsoft tool) [https://technet.microsoft.com/en-us/library/security/3062591.aspx](https://technet.microsoft.com/en-us/library/security/3062591.aspx)
Some specific Windows Group Policy to set
Security Options
-   Network security: LAN Manager authentication level - Send NTLMv2  response only\refuse NTLM & LM
-   Network security: Do not store LAN Manager hash value on next password change - Enabled
-   Network access: Do not allow anonymous enumeration of SAM accounts and shares - Enabled
-   Network access: Do not allow anonymous enumeration of SAM accounts - Enabled
-   Network access: Allow anonymous SID/name translation - Disabled
-   Accounts: Rename administrator account - Rename to something unique (but remember it)
-   Interactive logon: Message text for users attempting to log on - sometimes an inject
Local GPO is much faster to push out on small networks, and can be applied to any Windows system, not just domain joined ones (plus if the attacker kicks a box off the domain, domain GPO goes away). There isn't an easy way to do it for all GPO settings, but for security ones 'secedit' is your friend.
-- Export a config from a VM or other default install for reference:
secedit /export /cfg checkme.inf
-- Edit to to have more secure settings then import onto your target system: 
secedit /configure /db secedit.sdb /cfg securecheckme.inf

# stuff but good.
# make a backup admin account 
```
net user <name> * /ADD
net localgroup administrators <name>  /add
```
# start changing passwords

```
Net localgroup administrators >>LocalAdministratorUsers.txt
Net user {username_here} *
Net user >>localUsers.txt # useful command to have list of users to diff against
Net user {username} * 
```


Delete or disable any unnecessary accounts

Disable

```
Net user accountname /active:no 
```

Delete

```
Net user accountname /delete
```
## alternative way to disable accounts.
```
Get-LocalUser Guest | Disable-LocalUser

Get-LocalUser Administrator | Disable-LocalUser
```

## short script to change all passwords
```
Get-WmiObject win32_useraccount | Foreach-object {

([adsi]("WinNT://"+$_.caption).replace("\","/")).SetPassword("Asecurepassword123!")

}

```

# active directory
add and disable users in active directory. Same process just in AD this time. 
https://www.youtube.com/watch?v=piTFGytM1es



# patch time

 If XP/2k3 then PATCH MS08_067
>    https://docs.microsoft.com/en-us/security-updates/SecurityBulletins/2008/ms08-067

I think this is the site to donwload patch. unsure. 	
If Vista/7/2k8 then PATCH MS09_050
> http://www.microsoft.com/security/updates/bulletins/200910.aspx



# see what smb shares are available
```
 net share > shares.txt
```


Delete Unnecessary Shares on the Machine

```
Net share
Net share sharename /delete
```

flush dns cache
```
 ipconfig /flushdns
```

# firewall configuration
Turn on firewall
https://www.youtube.com/watch?v=A2jXQPprVXQ
Remove firewall exceptions 
https://www.youtube.com/watch?v=azGWLDLM3sc

## commandline firewall commands
**Important:** You only want to run the reset command if you are local to the box

```
netsh advfirewall reset
```

```
netsh advfirewall firewall delete rule *
netsh advfirewall firewall add rule dir=in action=allow protocol=tcp localport=3389 name=”Allow-TCP-3389-RDP”
```

```
netsh advfirewall firewall add rule dir=in action=allow protocol=icmpv4 name=”Allow ICMP V4”
netsh advfirewall set domainprofile firewallpolicy blockinbound,allowoutbound
netsh advfirewall set privateprofile firewallpolicy blockinbound,allowoutbound
netsh advfirewall set publicprofile firewallpolicy blockinbound,allowoutbound
netsh advfirewall set allprofile state on

```



## turn on windows defender
https://www.youtube.com/watch?v=MqcFLabRc10


Delete any scheduled tasks

```
schtasks /delete /tn * /f
```


Identify running services and processes

```
Get-service
Sc query type=service state=all
Tasklist >>RunningProcesses.Txt

```

```
wmic service list brief | findstr "Running"
```

```
wmic startup list full
```
### stop service
```
sc stop <serviceName>

```

# configure AD group policy
https://www.youtube.com/watch?v=piTFGytM1es


## Security Options
```
-   Network security: LAN Manager authentication level - Send NTLMv2  response only\refuse NTLM & LM
    
-   Network security: Do not store LAN Manager hash value on next password change - Enabled
    
-   Network access: Do not allow anonymous enumeration of SAM accounts and shares - Enabled
    
-   Network access: Do not allow anonymous enumeration of SAM accounts - Enabled
    
-   Network access: Allow anonymous SID/name translation - Disabled
    
-   Accounts: Rename administrator account - Rename to something unique (but remember it)
    
-   Interactive logon: Message text for users attempting to log on - sometimes an inject
```


```
-   Audit process tracking - Successes
    
-   Audit account management - Successes, Failures
    
-   Audit logon events - Successes, Failures
    
-   Audit account logon events - Successes, Failures
```
```
1. Create a GPO: Account Lockout
2.  Create a GPO: Enabling Verbose PowerShell Logging and Transcription
```


Configure local policies (Work in progress)


Secpol.msc
```
Security Settings>Account Policies>Account Lockout Policy Account Lockout Duration: 30min Account Lockout threshold: 2 failed logins Reset account lockout counter after: 30 mins

Local Policies>Audit Policy Enable all for failure and success



```



### turn off teredo 
is maybe just a windows 10 feature?
```
netsh interface teredo set state disabled
```

Disable SMB

-   Win 7 way is
    
    ```
    Get-Item HKLM:\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters | ForEach-Object {Get-ItemProperty $_.pspath}
    ```
    
    ```
    Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters" SMB1 -Type DWORD -Value 0 -Force 
    ```
    
    Then restart

Enable and set to highest setting UAC

```
C:\windows\system32\UserAccountControlSettings.exe
```


Check startup & disable unnecessary items via msconfig

```
msconfig
```


Uninstall any unnecessary software

```
Control appwiz.cpl
```


# installing anti virus time
install AVG for whatever version of windows is running. 


## run winpeas to look for more things to fix

https://github.com/carlospolop/PEASS-ng/blob/master/winPEAS/winPEASbat/winPEAS.bat


# process explorer
https://docs.microsoft.com/en-us/sysinternals/downloads/process-explorer

if you see notepad.exe something is wrong. if you see 


### [](https://github.com/mike-bailey/CCDC-Scripts/blob/master/WINDOWS.md#turns-off-remote-desktop)Turns off Remote Desktop
https://www.youtube.com/watch?v=QZEna-w-SEQ
`reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 1 /f`

`reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v UserAuthentication /t REG_DWORD /d 0 /f`

First, we flush the cache the traditional way. This is a common IT troubleshooting step if you have slow connectivity or other issues, by the way. `ipconfig /flushdns`

`net start` which displays running services, would be written to `servicestarted.txt` when run like this.

Get these tools onto the machine

-   Sysinternals
    -   [Sysmon](https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon)
    -   [Process Explorer](https://docs.microsoft.com/en-us/sysinternals/downloads/process-explorer)
    -   [AutoRuns](https://docs.microsoft.com/en-us/sysinternals/downloads/autoruns)
-   [BlueSpawn](https://bluespawn.cloud/quickstart/)
-   [EMET (If Windows 7)](https://www.microsoft.com/en-us/download/details.aspx?id=50766)
-   [ProcessHacker](https://processhacker.sourceforge.io/)
-   [**Microsoft Security Compliance Toolkit 1.0**](https://www.microsoft.com/en-us/download/details.aspx?id=55319)














	



 # [](https://github.com/WGU-CCDC/Blue-Team-Tools/blob/main/Windows/CCDCprep/checklist/DiscoverySteps.md#windows-intrusion-discovery)

# Windows Intrusion Discovery / patrol route

## goal: look for weird stuff. 


Check for any logged on users

```
Query session
Query user
Query process

```


Using the GUI, run Task Manager:

`C:\> taskmgr.exe`
or process explorer

Using the command prompt:

`C:\> tasklist`

`C:\> wmic process list full`

Also look for unusual services. Using the GUI:

`C:\> services.msc`

Using the command prompt:

`C:\> net start`

`C:\> sc query`

For a list of services associated with each process:

`C:\> tasklist /svc`

## Unusual Scheduled Tasks

Look for unusual scheduled tasks, especially those that run as a user in the Administrators group, as SYSTEM, or with a blank user name.

Using the GUI, run Task Scheduler: Start -> Programs -> Accessories -> System Tools -> Scheduled Tasks Using the command prompt:

`C:\> schtasks`

Check other autostart items as well for unexpected entries, remembering to check user autostart directories and registry keys.

Using the GUI, run msconfig and look at the Startup tab:

Start -> Run, msconfig.exe

Using the command prompt:

`C:\> wmic startup list full`


## Unusual Accounts

Look for new, unexpected accounts in the Administrators group:

`C:\> lusrmgr.msc`

Click on Groups, Double Click on Administrators, then check members of this group.This can also be done at the command prompt:

`C:\> net user`

`C:\> net localgroup administrators`


## Unusual Log Entries

Check your logs for suspicious events, such as:

-   “Event log service was stopped.”
-   “Windows File Protection is not active on this system.”
-   "The protected System file [file name] was not restored to its original, valid version because the Windows File Protection..."
-   “The MS Telnet Service has started successfully.”
-   Look for large number of failed logon attempts or locked out accounts.

To do this using the GUI, run the Windows event viewer:

`C:\> eventvwr.msc`

## [](https://github.com/WGU-CCDC/Blue-Team-Tools/blob/main/Windows/CCDCprep/checklist/DiscoverySteps.md#other-unusual-items)

## Other Unusual Items

Additional Supporting Tools Look for unusually sluggish performance and a single unusual process hogging the CPU:

Task Manager -> Process and Performance tabs

Or, run:

`C:\> taskmgr.exe`

Look for unusual system crashes, beyond the normal level for the given system.



