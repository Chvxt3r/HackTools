# Enumeration

## Domain Info
### Subdomains via [crt.sh](https://crt.sh)
Gather certificate info and display in json
```bash
curl -s https://crt.sh/\?q\=<domain>\&output\=json | jq .
```
Gather certificate info and filter by unique subdomain
```bash
curl -s https://crt.sh/\?q\=<domain>\&output\=json | jq . | grep name | cut -d":" -f2 | grep -v "CN=" | cut -d'"' -f2 | awk '{gsub(/\\n/,"\n");}1;' | sort -u
```
Find which subdomains are on-site or in-scope
```bash
for i in $(cat <subdomainlist>);do host $i | grep "has address" | grep <domain> | cut -d" " -f1,4;done
```
Run through shodan to find devices with the same IP
```bash
# Get IP addresses and export to a list
for i in $(cat <subdomainlist>);do host $i | grep "has address" | grep inlanefreight.com | cut -d" " -f4 >> ip-addresses.txt;done
# Run through shodan for attached devices
for i in $(cat ip-addresses.txt);do shodan host $i;done
```
