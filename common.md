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
gobuster dir -u $URL -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-medium-directories.txt -k -t 30
```
### Gobuster files
```bash
gobuster dir -u $URL -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-medium-files.txt -k -t 30
```

### Gobuster for SubDomain brute forcing:
```bash
gobuster dns -d domain.org -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt -t 30
```
"WARNING: Make sure any DNS name you find resolves to an in-scope address before you test it"

### Extract IPs from a text file.
```bash
grep -o '[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}' nmapfile.txt
```

### Wfuzz XSS Fuzzing
```bash
wfuzz -c -z file,/usr/share/wordlists/seclists/Fuzzing/XSS/XSS-BruteLogic.txt "$URL"
wfuzz -c -z file,/usr/share/wordlists/seclists/Fuzzing/XSS/XSS-Jhaddix.txt "$URL"
```
### COMMAND INJECTION WITH POST DATA
```bash
wfuzz -c -z file,/usr/share/wordlists/seclists/Fuzzing/command-injection-commix.txt -d "doi=FUZZ" "$URL"
```

### Test for Parameter Existence!
```bash
wfuzz -c -z file,/usr/share/wordlists/seclists/Discovery/Web-Content/burp-parameter-names.txt "$URL"
```

### AUTHENTICATED FUZZING DIRECTORIES
```bash
wfuzz -c -z file,/usr/share/wordlists/seclists/Discovery/Web-Content/raft-medium-directories.txt --hc 404 -d "SESSIONID=value" "$URL"
```

### AUTHENTICATED FILE FUZZING:
```bash
wfuzz -c -z file,/usr/share/wordlists/seclists/Discovery/Web-Content/raft-medium-files.txt --hc 404 -d "SESSIONID=value" "$URL"
```

### FUZZ Directories:
```bash
wfuzz -c -z file,/usr/share/wordlists/seclists/Discovery/Web-Content/raft-large-directories.txt --hc 404 "$URL"
```

### FUZZ FILES:
```bash
wfuzz -c -z file,/usr/share/wordlists/seclists/Discovery/Web-Content/raft-large-files.txt --hc 404 "$URL"
|
LARGE WORDS:
wfuzz -c -z file,/usr/share/wordlists/seclists/Discovery/Web-Content/raft-large-words.txt --hc 404 "$URL"
|
USERS:
wfuzz -c -z file,/usr/share/wordlists/seclists/Usernames/top-usernames-shortlist.txt --hc 404,403 "$URL"
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
smtp-user-enum -M VRFY -U /usr/share/wordlists/seclists/Usernames/xato-net-10-million-usernames.txt -t $IP
smtp-user-enum -M EXPN -U /usr/share/wordlists/seclists/sr/share/wordlists/seclists/Usernames/xato-net-10-million-usernames.txt -t $IP
smtp-user-enum -M RCPT -U /usr/share/wordlists/seclists/Usernames/xato-net-10-million-usernames.txt -t $IP
smtp-user-enum -M EXPN -U /usr/share/wordlists/seclists/Usernames/xato-net-10-million-usernames.txt -t $IP
```

## Command Execution Verification - [Ping check]
```bash
sudo tcpdump -i any icmp -c10
```
```bash
#Check Network
netdiscover -r 0.0.0.0/24
```
```bash
#INTO OUTFILE D00R
SELECT “” into outfile “/var/www/WEROOT/backdoor.php”;
```
## LFI?
### PHP Filter Checks.
php://filter/convert.base64-encode/resource=

UPLOAD IMAGE?
GIF89a1

Chuxtr's rip-off of S1ren's common implies that you have seclists installed in the default Kali location '/usr/share/wordlists/seclists/'
https://github.com/danielmiessler/SecLists


