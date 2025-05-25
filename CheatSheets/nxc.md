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
## SQL
* Run commands
  ```bash
  nxc mssql <hname> -u <user> -p <password> -x $command
  ```
