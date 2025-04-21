# Common Linux PrivEsc Recon

## Host Enum

### First Steps
```bash
whoami
id
hostname
ip -a
sudo -l
```
### Find Users & User Info
```bash
cat /etc/passwd
grep "sh$" /etc/passwd # Find only users with login shells, and which shell they have.
```
Current Users Path
```bash
echo $PATH
```
Current Users Env
```bash
env
```
Bash History
```bash
history
```
### System Info
OS Release
```bash
/etc/os-release
```
Current Kernel Version
```bash
uname -a
cat /proc/version
```
Available login shells
```bash
cat /etc/shells
```
Current Open Ports and Services
```bash
netstat -ltp
ss -ta # Little bit more detail
```
Current Processes
```bash
ps aux
```
Cron Jobs
```bash
ls -la /etc/cron.daily
```
File Systems and Drives
```bash
lsblk
cat /etc/fstab
cat /etc/fstab | grep <password/username/credential/etc>
```
Find Writeable Directories
```bash
find / -path /proc -prune -o -type d -perm -o+w 2>/dev/null
```
Find Writeable Files
```bash
find / -path /proc -prune -o -type d -perm -o+w 2>/dev/null
```
