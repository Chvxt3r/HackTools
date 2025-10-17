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

## Automated Tools

## Kernel Exploits

## Passwords and Port Forwarding

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


