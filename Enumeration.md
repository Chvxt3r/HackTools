# Enumeration

## Domain Info
### Subdomains via [crt.sh](https://crt.sh)
Gather certificate info and display in json
```bash
curl -s https://crt.sh/\?q\=<domain>\&output\=json | jq .
```
Gather certificate info and filter by unique subdomain
```bash
curl -s https://crt.sh/\?q\=<domain>\&output\=json | jq . | grep name | cut -d":" -f2 | grep -v "CN=" | cut -d'"' -f2 | awk '{gsub(/\\n/,"\n");}1;' | sort -u
```
Find which subdomains are on-site or in-scope
```bash
for i in $(cat <subdomainlist>);do host $i | grep "has address" | grep <domain> | cut -d" " -f1,4;done
```
Run through shodan to find devices with the same IP
```bash
# Get IP addresses and export to a list
for i in $(cat <subdomainlist>);do host $i | grep "has address" | grep <domain> | cut -d" " -f4 >> ip-addresses.txt;done
# Run through shodan for attached devices
for i in $(cat ip-addresses.txt);do shodan host $i;done
```
Get any and all available DNS Records for a domain
```bash
dig any <domain>
```

### Search for Cloud Providers
Google Dorks:  
Amazon  
    intext:domain inurl:amazonaws.com  
Azure  
    intext:domain inurl:blob.core.windows.net  

Infrastructure:  
[domain.glass](https://domain.glass)  
[GrayHatWarfare](https://grayhatwarfare.com)  
** Don't forget to search company name abbreviations **  
** Also look for ssh keys, passwords, etc **  

## Services
### FTP
NMAP fingerprinting
```bash
sudo nmap -sV -p21 -sC -A <IP>
```
Anonymous Login
```bash
ftp <IP>
```
FTP Settings
```bash
status
```
Recursive Listing
```bash
ls -R
```
Download a file
```bash
get <filename>
```
Download ALL files
```bash
wget -m --no-passive ftp://<username>:<password>@<IP>
```
Upload a file
```bash
put <filename>
```
### SMB
Nmap Fingerprinting
```bash
sudo nmap <IP> -sV -sC -p139,445
```
Connect with smbclient
```bash
smbclient //<IP>/share
```
Download a file
```bash
get <filename>
```
### SMB/RPC
Anonymous Connection
```bash
rpcclient -U "" <IP>
```
Server Info
```bash
srvinfo
```
Enumerate domains
```bash
enumdomains
```
Query domain info
```bash
querydominfo
```
Enumerate all shares
```bash
netshareenumall
```
Get info about a specific share
```bash
netsharegetinfo <share>
```
