# Linux Recon & Attack

## Enumeration
### Scans
```bash
sudo nmap -p- -sC -sV $IP --open -oA scans/nmap_Initial
```

==Fuzz Directories
```bash
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-large-directories-lowercase.txt:FUZZ -u http://<Domain or IP>/FUZZ -fs 278
```
==Fuzz Subdomains
```bash
ffuf -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt -u http://<Domain or IP> -H "Host:FUZZ.<Domain or IP" -fw 20
```
==Fuzz Files
```bash
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-large-files.txt -u http://<Domain or IP>/FUZZ -e .php,.html,.txt -fs 283
```
## Credentials Enumeration
==What's Listening?
```bash
netstat -tulpn
```
### Databases
==MySQL
```bash
mysql -u <user> -p<pass> -h <host> <db>#No space between -p and the password (ex: -pPassword)
```

## Pivots
SSH Socks4 Dynamic
```bash
ssh -D 9050 <user>@<host>
```
Change browser settings to socks4 proxy 127.0.0.1:9050 to route web traffic

Proxychains settings located in /etc/proxychains4.conf
```bash
[ProxyList]
# add proxy here ...
# meanwile
# defaults set to "tor"
socks4  127.0.0.1 9050
```
For app traffic, prepend "proxychains4" before command
Example:
```bash
proxychains4 curl http://<site>.<dom>/<page>
```


