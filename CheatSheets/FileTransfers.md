# File Transfers

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
b. Filess = Download and execute without saving the file  
## Transferring with Code
## Misc File Transfer Methods
## Catching files over HTTP/S
