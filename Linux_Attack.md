# Linux Recon & Attack

## Enumeration
### Scans
```bash
sudo nmap -p- -sC -sV $IP --open -oA scans/nmap_Initial
```
### Fuzzing
Fuzz Directories
```bash
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-large-directories-lowercase.txt:FUZZ -u http://<Domain or IP>/FUZZ -fs 278
```
Fuzz Subdomains
```bash
ffuf -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt -u http://<Domain or IP> -H "Host:FUZZ.<Domain or IP" -fw 20
```
Fuzz Files
```bash
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-large-files.txt -u http://<Domain or IP>/FUZZ -e .php,.html,.txt -fs 283
```
## Credentialed Enumeration
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
SSH Simple Port forward (Useful for forward to a website only available from target machine)
```bash
ssh -L 8080:localhost:52846 <user>@<target>
```
## PrivEsc
### Environmental
First
```bash
whoami
id
hostname
ip -a
sudo -l
```
Sudo - List User's Priv
```bash
sudo -l
```
Possible Passwords
```bash
cat /etc/passwd
grep "*sh$" /etc/passwd #Find only users with login shells, and which shell they have.
```
Bash History
```bash
history
```
List current Processes
```bash
ps aux
```
Cron Jobs
```bash
ls -la /etc/cron.daily
```
File Systems & Additional Drives
```bash
lsblk
cat /etc/fstab
cat /etc/fstab | grep <password/username/credential/etc>
```
Find Writable Directories
```bash
find / -path /proc -prune -o -type d -perm -o+w 2>/dev/null
```
Find Writable Files
```bash
find / -path /proc -prune -o -type d -perm -o+w 2>/dev/null
```
OS Release
```bash
/etc/os-release
```
Current Users Path
```bash
echo $PATH
```
Current Users Enviornment
```bash
env
```
Current Kernel Version
```bash
uname -a
cat /proc/version
```
List Login Shells
```bash
cat /etc/shells
```
Routing Table
```bash
route
netstat -rn
```
Arp Table
```bash
arp -a
```
Existing Groups
```bash
cat /etc/group
```
Get all members of a group
```bash
getent group sudo
```
Mounted File Systems
```bash
df -h
```
