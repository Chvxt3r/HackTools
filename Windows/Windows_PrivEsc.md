# Windows Privilege Escalation
## Useful Tools
[SeatBelt](https://github.com/GhostPack/Seatbelt)  
[WinPEAS](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS)  
[PowerUp](https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Privesc/PowerUp.ps1)  
[SharpUp](https://github.com/GhostPack/SharpUp) - C# Version of PowerUp  
[JAWS](https://github.com/411Hall/JAWS)  
[SessionGopher](https://github.com/Arvanaghi/SessionGopher)  
[Watson](https://github.com/rasta-mouse/Watson)  
[LaZagne](https://github.com/AlessandroZ/LaZagne)  
[WES-NG](https://github.com/bitsadmin/wesng)  
[SysInternals](https://docs.microsoft.com/en-us/sysinternals/downloads/sysinternals-suite)  

### Usually Safe Spaces
```cmd
c:\Windows\Temp # usually writable by the BUILTIN\Users group
```
## Lay of the Land
### Situational Awareness
```cmd
ipconfig /all #Get status and configuration of network interfaces
```
```cmd
arp -a #Show the ARP Table
```
```cmd
route print #Show routes to other networks (Useful for pivoting through a dmz)
```
```powershell
Get-MpComputerStatus #Check Windows Defender Status
```
```powershell
Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections #List AppLocker Rules
```
```powershell
# Test AppLocker Policy against an executable and User
Get-AppLockerPolicy -Local | Test-AppLockerPolicy -path C:\Windows\System32\cmd.exe -User Everyone
```
## Initial Enumeration
### Targets
NT Authority\SYSTEM  
Built-in Administrator account  
Any other local account that is a member of the local Administrators Group  
Standard domain user is part of local Admins
Domain admin that is part of the local Administrators group
### System Info
```cmd
tasklist /svc #List all Services
set #Display All Environment Variabls
systeminfo #Display Detailed Configuration Info
wmic qfe #Display Patches and Updates (Powershell - Get-Hotfix | ft -AutoSize)
wmic product get name #Display Installed Software (Powershell - Get-WmiObject -Class Win32_Product | select Name, Version)
netstat -ano #Display Active TCP/UDP connections
```
```powershell
gci \\.\pip\ #Get a listing of named pipes
```
```cmd
#List DACL's of a named pipe
accesschk.exe /accepteula \\.\Pipe\lsass -v #accesschk.exe is part of sysinternals
```
### User/Group Info
```cmd
query user #Get a list of logged in users
echo %username% #Get current user (equivalent to whoami)
whoami /priv #Get Current User Privileges
whoami /groups #What groups am I in?
net user #Get list of all users
net localgroup #Get list of all local groups
net localgroup <groupname> #Get details of the group, including members
net accounts #Get Password Policy and Other account info
```
## Process Communication
### Access Tokens
Used to describe the security context of a process or thread

