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
