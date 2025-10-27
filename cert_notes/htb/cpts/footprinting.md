# Footprinting
## Intro
### Methodology
#### Enumeration Layers
|Level|Layers|
|-----|------|
|`Infrastructure`|Internet Presence, Gateway|
|`Host-Based`|Accessible Services, Processes|
|`OS-Based`|Privileges, OS Setup|
#### 6 Layers of Footprinting
|Layer|Description|Information Categories|
|-----|-----------|----------------------|
|`1. Internet Presence`|Identification of internet presence and externally accessible infrastructure.|Domains, Subdomains, vHosts, ASN, Netblocks, IP Addresses, Cloud Instances, Security Measures|
|`2. Gateway`|Identify the possible security measures to protect the company's external and internal infrastructure.|Firewalls, DMZ, IPS/IDS, EDR, Proxies, NAC, Network Segmentation, VPN, Cloudflare|
|`3. Accessible Services`|Identify accessible interfaces and services that are hosted externally or internally.|Service Type, Functionality, Configuration, Port, Version, Interface|
|`4. Processes`|Identify the internal processes, sources, and destinations associated with the services.|PID, Processed Data, Tasks, Source, Destination|
|`5. Privileges`|Identification of the internal permissions and privileges to the accessible services.|Groups, Users, Permissions, Restrictions, Environment|
|`6. OS Setup`|	Identification of the internal components and systems setup.|OS Type, Patch Level, Network config, OS Environment, Configuration files, sensitive private files|

* Internet Presence: Identify all possible targets and interfaces that can be tested.
* Gateway: Understand what we are dealing with and what we have to watch out for.
* Accessible Services: Understand the reason and functionality of the target and how to communicate with and exploit it.
* Processes: Understand the processes running on the machine and the dependencies between them.
* Priviliges: Identify and understand what and what is not possible with a given set of priviliges.
* OS Setup: Understand how the admin's manage the environment and what useful/sensitive information we can get from them.

## Infrastructure Based Enumeration
### Domain Information
#### Get an understanding of the company and it's services
* What does the company do?
* What technologies does the target company employ to accomplish it's mission?
* 
#### Online Presence
> Online presense can include the website, DNS records, basically any available information regarding the target's infrastructure.  
* Review the actual certificate in your browser.
* [crt.sh](https://crt.sh/) - Review the `SSL Cert` for the domain.
```bash
# Quickly get and filter out just the subdomains from a crt.sh query.
curl -s https://crt.sh/\?q\=inlanefreight.com\&output\=json | jq . | grep name | cut -d":" -f2 | grep -v "CN=" | cut -d'"' -f2 | awk '{gsub(/\\n/,"\n");}1;' | sort -u
```
* Look for company-hosted servers.
```bash
# We can use our list from the previous step
for i in $(cat subdomainlist);do host $i | grep "has address" | grep inlanefreight.com | cut -d" " -f1,4;done
```
* Gathering info with [shodan](https://www.shodan.io/)(Requires an API key)
```bash
# Reduce our list to just company hosted servers
for i in $(cat subdomainlist);do host $i | grep "has address" | grep inlanefreight.com | cut -d" " -f4 >> ip-addresses.txt;done
# Run our new list through Shodan to get more info
for i in $(cat ip-addresses.txt);do shodan host $i;done
```
* DNS Records
```bash
dig any [domain]
```
### Cloud Resources
#### Google Dorks for cloud resources
* Google searchs of `inurl:` and `intext:`
* Amazon - `intext:[search term] inurl:amazonaws.com`
* Azure - `intext:[search term] inurl:blob.core.windows.net`
#### Code Review
* Review the code of the website and any downloadable files for pointers to cloud storage
#### [GrayHatWarfare](https://buckets.grayhatwarfare.com/)
* useful for search AWS/Azure/Google buckets and their contents
* make sure to search for credentials (SSH keys, admin portals, etc)
### Staff
#### LinkedIn
* Review job posting to learn about infrastructure
* Review photo's for badge information and inadvertant information disclosure on screens, TV's, Signs, etc.
#### Github
* If you can track down an employee or company github, review the code for emails, JWT tokens, SSH keys, etc.
## Host Based Enumeration
### FTP
* Check for anonymous sessions
* Active - Client initiates the connection on port 21 and then informs the sever which port to reply on.
* Passive - Server advertises the port the client should connect on (Great for firewall evasion)
#### TFTP
* Provides simple transfer of files
* Does not provide any kind of client authentication
* TFTP does not have a directory list function. 
#### Default Configurations
* vsFTPd default settings file
```bash
cat /etc/vsftpd.conf | grep -v "#"
```
* FTPUsers - People in this file are not allowed to use FTP
```bash
cat /etc/ftpusers
```
#### Dangerous Settings
* Check for these `optional` configuration settings  

|Setting|Description|
|-------|-----------|
|`anonymous_enable=YES`|Allowing anonymous login?|
|`anon_upload_enable=YES`|Allowing anonymous to upload files?|
|`anon_mkdir_write_enable=YES`|Allowing anonymous to create new directories?|
|`no_anon_password=YES`|Do not ask anonymous for password?|
|`anon_root=/home/username/ftp`|Directory for anonymous.|
|`write_enable=YES`|Allow the usage of FTP commands: STOR, DELE, RNFR, RNTO, MKD, RMD, APPE, and SITE?|

#### FTP Commands
|Command|Description|
|-------|-----------|
|`status`|Shows some of the settings of the server|
|`debug`|Turns on/off debugging|
|`trace`|Turns on/off packet tracing|
|`ls`|Lists the contents of the directory w/ permissions and RIDS|
|`ls -R`|Recursive Listing|
|`get`|Download a file|
|`wget -m --no-passive ftp://anonymous:anonymous@[IP]`|Download all the files we have access to|
|`put`|Upload a file|

#### Nmap FTP Scripts
```bash
find / -type f -name ftp* 2>/dev/null | grep scripts

/usr/share/nmap/scripts/ftp-syst.nse
/usr/share/nmap/scripts/ftp-vsftpd-backdoor.nse
/usr/share/nmap/scripts/ftp-vuln-cve2010-4221.nse
/usr/share/nmap/scripts/ftp-proftpd-backdoor.nse
/usr/share/nmap/scripts/ftp-bounce.nse
/usr/share/nmap/scripts/ftp-libopie.nse
/usr/share/nmap/scripts/ftp-anon.nse
/usr/share/nmap/scripts/ftp-brute.nse
```
#### Service Interaction
```bash
# netcat
nc -nv 10.129.14.136 21

# telnet
telnet 10.129.14.136 21

# openssl (May provide additional useful information)
openssl s_client -connect 10.129.14.136:21 -starttls ftp

CONNECTED(00000003)                                                                                      
Can't use SSL_get_servername                        
depth=0 C = US, ST = California, L = Sacramento, O = Inlanefreight, OU = Dev, CN = master.inlanefreight.htb, emailAddress = admin@inlanefreight.htb
verify error:num=18:self signed certificate
verify return:1

depth=0 C = US, ST = California, L = Sacramento, O = Inlanefreight, OU = Dev, CN = master.inlanefreight.htb, emailAddress = admin@inlanefreight.htb
verify return:1
---                                                 
Certificate chain
 0 s:C = US, ST = California, L = Sacramento, O = Inlanefreight, OU = Dev, CN = master.inlanefreight.htb, emailAddress = admin@inlanefreight.htb
 
 i:C = US, ST = California, L = Sacramento, O = Inlanefreight, OU = Dev, CN = master.inlanefreight.htb, emailAddress = admin@inlanefreight.htb
---
 
Server certificate

-----BEGIN CERTIFICATE-----

MIIENTCCAx2gAwIBAgIUD+SlFZAWzX5yLs2q3ZcfdsRQqMYwDQYJKoZIhvcNAQEL
...SNIP...
```
### SMB
#### Samba (SMB/CIFS for linux)
##### Default Configuration
```bash
cat /etc/samba/smb.conf | grep -v "#\|\;" 

[global]
   workgroup = DEV.INFREIGHT.HTB
   server string = DEVSMB
   log file = /var/log/samba/log.%m
   max log size = 1000
   logging = file
   panic action = /usr/share/samba/panic-action %d

   server role = standalone server
   obey pam restrictions = yes
   unix password sync = yes

   passwd program = /usr/bin/passwd %u
   passwd chat = *Enter\snew\s*\spassword:* %n\n *Retype\snew\s*\spassword:* %n\n *password\supdated\ssuccessfully* .

   pam password change = yes
   map to guest = bad user
   usershare allow guests = yes

[printers]
   comment = All Printers
   browseable = no
   path = /var/spool/samba
   printable = yes
   guest ok = no
   read only = yes
   create mask = 0700

[print$]
   comment = Printer Drivers
   path = /var/lib/samba/printers
   browseable = yes
   read only = yes
   guest ok = no
```
##### Invidvidual Share Settings
|Settings|Description|
|--------|-----------|
|`[sharename]`|The name of the network share.|
|`workgroup = WORKGROUP/DOMAIN`|Workgroup that will appear when clients query.|
|`path = /path/here/`|The directory to which user is to be given access.|
|`server string = STRING`|The string that will show up when a connection is initiated.|
|`unix password sync = yes`|Synchronize the UNIX password with the SMB password?|
|`usershare allow guests = yes`|Allow non-authenticated users to access defined share?|
|`map to guest = bad user`|What to do when a user login request doesn't match a valid UNIX user?|
|`browseable = yes`|Should this share be shown in the list of available shares?|
|`guest ok = yes`|Allow connecting to the service without using a password?|
|`read only = yes`|Allow users to read files only?|
|`create mask = 0700`|What permissions need to be set for newly created files?|

##### Dangerous Settings
|Settings|Description|
|--------|-----------|
|`browseable = yes`|Allow listing available shares in the current share?|
|`read only = no`|Forbid the creation and modification of files?|
|`writable = yes`|Allow users to create and modify files?|
|`guest ok = yes`|Allow connecting to the service without using a password?|
|`enable privileges = yes`|Honor privileges assigned to specific SID?|
|`create mask = 0777`|What permissions must be assigned to the newly created files?|
|`directory mask = 0777`|What permissions must be assigned to the newly created directories?|
|`logon script = script.sh`|What script needs to be executed on the user's login?|
|`magic script = script.sh`|Which script should be executed when the script gets closed?|
|`magic output = script.out`|Where the output of the magic script needs to be stored?|
##### Sample Share (Add to /etc/samba/smb.conf
```bash
[notes]
    comment = CheckIT
    path = /mnt/notes/

    browseable = yes
    read only = no
    writable = yes
    guest ok = yes

    enable privileges = yes
    create mask = 0777
    directory mask = 0777
```
* After adding a share, restart samba
```bash
sudo systemctl restart smbd
```
### SMB Client
* No Credentials, list all shares
```bash
smbclient -N -L //[IP]
```
* Connect, Anonymous login
```bash
smbclient //[IP]/[sharename]
```
* Download a file
```bash
get [filename]
```
* Get Status of smb
```bash
smbstatus
```
### Footprinting SMB
#### Nmap
```bash
sudo nmap [IP] -sVC -p139,445
```
#### RPCclient
```bash
rpclient -U '' [IP]
```
* RPCclient commands
|Query|Description|
|-----|-----------|
|`srvinfo`|Server information.|
|`enumdomains`|Enumerate all domains that are deployed in the network.|
|`querydominfo`|Provides domain, server, and user information of deployed domains.|
|`netshareenumall`|Enumerates all available shares.|
|`netsharegetinfo [sharename]`|Provides information about a specific share.|
|`enumdomusers`|Enumerates all domain users.|
|`queryuser <RID>`|Provides information about a specific user.|
* Simple bash script to Brute Force RID's
```bash
for i in $(seq 500 1100);do rpcclient -N -U "" 10.129.14.128 -c "queryuser 0x$(printf '%x\n' $i)" | grep "User Name\|user_rid\|group_rid" && echo "";done
```
* Alternate way with Imapcket-Samrdump
```bash
impacket-samrdump [IP]
```
#### SMBMap
```bash
smbmap -H [IP]
```
#### Netexec
```bash
nxc smb [IP] --shares -u '' -p''
```
#### [Enum4Linux-ng](https://github.com/cddmp/enum4linux-ng)
```bash
./enum4linux-ng [IP] -A
```
