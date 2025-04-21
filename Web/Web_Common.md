# Common Web Tools

## Nmap
```bash
nmap -p- -sT -sV -A $IP
nmap -p- -sC -sV $IP --open
nmap -p- --script=vuln $IP
```
## Nmap-Methods
```bash
nmap  --script http-methods --script-args http-methods.url-path='/website' 
      --script smb-enum-shares
nmap -p80,443 --script=http-methods  --script-args http-methods.url-path='/directory/goes/here'
```
## WPScan & SSL
```bash
wpscan --url $URL --disable-tls-checks --enumerate p --enumerate t --enumerate u
```
## WPScan Brute Forcing
```bash
wpscan --url $URL --disable-tls-checks -U users -P /usr/share/wordlists/rockyou.txt
```
## WPScan Aggressive Plugin Detection
```bash
wpscan --url $URL --enumerate p --plugins-detection aggressive
```
## Nikto with SSL and Evasion
```bash
nikto --host $IP -ssl -evasion 1
```
## GoBuster directory
```bash
gobuster dir -u $URL -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-medium-directories.txt -k -t 30
```
## GoBuster Files
```bash
gobuster dir -u $URL -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-medium-files.txt -k -t 30
```
## GoBuster Subdomain brute forcing
```bash
gobuster dns -d domain.org -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt -t 30
```
**WARNING: Make sure any DNS name you find resolves to an in-scope address before you test it**
## Wfuzz XSS Fuzzing
```bash
wfuzz -c -z file,/usr/share/wordlists/seclists/Fuzzing/XSS/XSS-BruteLogic.txt "$URL"
wfuzz -c -z file,/usr/share/wordlists/seclists/Fuzzing/XSS/XSS-Jhaddix.txt "$URL"
```
## Wfuzz - Authenticated Fuzzing Directories
```bash
wfuzz -c -z file,/usr/share/wordlists/seclists/Discovery/Web-Content/raft-medium-directories.txt --hc 404 -d "SESSIONID=value" "$URL"
```
## Wfuzz - Authenticated Fuzzing Files
```bash
wfuzz -c -z file,/usr/share/wordlists/seclists/Discovery/Web-Content/raft-medium-files.txt --hc 404 -d "SESSIONID=value" "$URL"
```
## Wfuzz Directories
```bash
wfuzz -c -z file,/usr/share/wordlists/seclists/Discovery/Web-Content/raft-large-directories.txt --hc 404 "$URL"
```
## Wfuzz Files
```bash
wfuzz -c -z file,/usr/share/wordlists/seclists/Discovery/Web-Content/raft-large-files.txt --hc 404 "$URL"
|
LARGE WORDS:
wfuzz -c -z file,/usr/share/wordlists/seclists/Discovery/Web-Content/raft-large-words.txt --hc 404 "$URL"
|
USERS:
wfuzz -c -z file,/usr/share/wordlists/seclists/Usernames/top-usernames-shortlist.txt --hc 404,403 "$URL"
```
## SQLMap
```bash
sqlmap -u $URL --threads=2 --time-sec=10 --level=2 --risk=2 --technique=T --force-ssl
sqlmap -u $URL --threads=2 --time-sec=10 --level=4 --risk=3 --dump
```
