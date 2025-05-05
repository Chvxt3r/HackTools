# Living Off the Land

## Links
[LOLBins](https://lolbas-project.github.io/) List of Windows Binaries that can be used to bypass security
[GTFOBins](https://gtfobins.github.io/) Curated list of Unix/Linux binaries that can be used to bypass security restrictions
## Linux
### Common Scripts
> Don't store scripts in common directories /tmp works fabulously
* **[Lin Enum](https://github.com/rebootuser/LinEnum)**  
  ```bash
  chmod +x LinEnum.sh #If Needed
  ./LinEnum.sh
  ```
* **[Linux Exploit Suggester](https://github.com/The-Z-Labs/linux-exploit-suggester)**  
  ```bash
  chmod +x les.sh #If needed
  ./les.sh
  ```
### Grep
* **Search contents of files/directories recursively**    
```bash
grep -R "<search term>" <PATH>
```
* **Search contents of files/directories recursively for specific term**  
  ```bash
  #This can be quite involved if doing it from root as shown here, try and 
  #narrow it down some. For example: /etc, /var/www/html, /var/www/html/wp-config
  grep --color=auto -rnw '/' -ie "Password" --color=always 2> /dev/null
  ```
* **Extract IP's from a text file**  
  ```bash
  grep -o '[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}' nmapfile.txt
  ```
### Informational
Current Users Path
```bash
echo $PATH
```
Current Users Environment
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
### Files
Find all hidden files
```bash
find / -type f -name ".*" -exec ls -l {} \; 2>/dev/null | grep <username>
```
Find History Files
```bash
find / -type f \( -name *_hist -o name *_history\) -exec ls -l {} \; 2>/dev/null
```
Find Configuration Files
```bash
find / -type f\( -name *.conf -o -name *.config\) -exec ls -l {} \; 2>/dev/null
```
### Packages
List installed Packages
```bash
apt list --installed | tr "/" " " | cut -d" " -f1,3 | sed 's/[0-9]://g' | tee -a installed_pkgs.list
```
List Binaries
```bash
ls -l /bin /usr/bin/ /usr/sbin/
```
### Credential Hunting
Web/DB Credentials
```bash
cat <Config File> | grep 'DB_USER\|DB_PASSWORD' # Use on any config files you find in /var
```

