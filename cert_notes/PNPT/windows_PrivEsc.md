# Windows Privilege Escalation

## Foothold

## Initial Enumeration
### System Enumeration
* Systeminfo
```powershell
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

### Network Enumeration

### Password Hunting

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


