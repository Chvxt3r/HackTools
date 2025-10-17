# Windows Privilege Escalation

## Foothold

## Initial Enumeration
### System Enumeration
* Systeminfo
```cmd
systeminfo
```
* Using the pipe (|) as grep
```cmd
systeminfo | findstr /B /C:"OS Name" /C:"OS Version" /C:"System Type"
```
* Enumerating Patching Level
> wmic must be installed for this to work. Not installed by default on Win10/11
```cmd
wmic qfe get Caption,Description,HotFixID,Installation
```

### User Enumeration
* Who am I?
```cmd
whoami
```
* What are my privileges?
```cmd
whoami /priv
```
* What groups am I in?
```cmd
whoami /groups
```
* What users are on this machine?
```cmd
net user
```
* What info can I get about a specific user?
```cmd
net user [user]
```
* What are the local groups?
```cmd
net localgroup
```
* Who's in what group?
```cmd
net localgroup [group]
```

### Network Enumeration
* Get the entire network config
```cmd
ipconfig /all
```
* Find our computer neighbors
```cmd
arp -a
```
* View the routing table
```cmd
route print
```
* What ports and services are running?
```cmd
netstat -ano
```

### Password Hunting
* Basic Password Hunting with findstr
```cmd
findstr /si [search term] [file extensions]

# Example
findstr /si password *.txt *.ini *.config
```
* Wifi passwords
```cmd
# Find the profile
netsh wlan show profile

# Show the passwords
netsh wlan show provile [SSID] key=clear
```

### AV Enumeration
* Windows Defender
```cmd
sc query windefund
```
* All the services on the machine (Look for AV)
```cmd
sc queryex type= service
```
* Firewall Settings
```cmd
netsh advfirewall firewall dump
# or
netsh firewall show state
```
* Show firewall configuration
```cmd
netsh firewall show config
```

## Automated Tools
* [WinPEAS](https://github.com/peass-ng/PEASS-ng)

* [Windows PrivEsc Checklist](https://book.hacktricks.wiki/en/windows-hardening/checklist-windows-privilege-escalation.html)

* [Seatbelt](https://github.com/GhostPack/Seatbelt)

* [WES-NG(Windows Exploit Suggester)](https://github.com/bitsadmin/wesng)

## Kernel Exploits
* [Big list of kernel exploits - Has not been updated in a while](https://github.com/SecWiki/windows-kernel-exploits)
* usually just upload and execute. 
* can be found using the exploit suggest in MSF (post/multi/recon/local_exploit_suggester)

## Passwords and Port Forwarding
* Search for cleartext passwords
```cmd
findstr /si password *.txt
findstr /si password *.xml
findstr /si password *.ini

#Find all those strings in config files.
dir /s *pass* == *cred* == *vnc* == *.config*

# Find all passwords in all files.
findstr /spin "password" *.*
findstr /spin "password" *.*
```
* Common files containing passwords
```cmd
c:\sysprep.inf
c:\sysprep\sysprep.xml
c:\unattend.xml
%WINDIR%\Panther\Unattend\Unattended.xml
%WINDIR%\Panther\Unattended.xml

dir c:\*vnc.ini /s /b
dir c:\*ultravnc.ini /s /b 
dir c:\ /s /b | findstr /si *vnc.ini
```
* Common Registry keys containing passwords
```cmd
# VNC
reg query "HKCU\Software\ORL\WinVNC3\Password"

# Windows autologin
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon"

# SNMP Paramters
reg query "HKLM\SYSTEM\Current\ControlSet\Services\SNMP"

# Putty
reg query "HKCU\Software\SimonTatham\PuTTY\Sessions"

# Search for password in registry
reg query HKLM /f password /t REG_SZ /s
reg query HKCU /f password /t REG_SZ /s
```
## WSL

## Impersonation and Potato Attacks

## GetSystem

## RUNAS

## Registry

## Executables

## Startup Applications

## DLL Hijacking

## Service Permissions

## CVE-2019-1388
