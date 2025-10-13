# NXC Cheatsheet

## General

## Non-Credentialed enumeration
### Basic SMB Recon
* Scan for live targets
```bash
nxc smb 172.16.10.0/24
```
* Scan for smb signing disabled and generate a relay list
```bash
nxc smb 172.16.10.0/24 --gen-relay-list relaylist.txt
```

### Null/Anon Sessions
* Enumerate password policy
```bash
nxc smb 172.16.10.10 -u '' -p '' --pass-pol
```
* Export password policy to json and clean up
```bash
nxc smb 172.16.10.10 -u '' -p '' --pass-pol --export passpol.txt

# Clean up the exported json
sed -i "s/'/\"/g" passpol.txt
```
* Enumerate users
```bash
nxc smb 172.16.10.10 -u '' -p '' --users --export users.txt

# Clean up the exported list
sed -i "s/'/\"/g" users.txt
jq -r '.[]' users.txt > userslist.txt
```bash
* Enumerate users through RID brute force
> By default, --rid-brute only tries 4000 RIDs. Specify an upper limit with --rid-brute [max rid]  
```bash
nxc smb 172.16.10.10 -u '' -p '' --rid-brute
```
* Enumerating shares
```bash
nxc smb $IP -u '' -p '' --shares
```
## Credentialed enumeration

## Admin Credentialed enumeration

## Remote Shell

## Modules

## Misc

## Database
