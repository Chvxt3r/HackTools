# Footprinting
## Intro
### Methodology
#### Enumeration Layers
|Level|Layers|
|-----|------|
|`Infrastructure`|Internet Presence, Gateway|
|`Host-Based`|Accessible Services, Processes|
|`OS-Based`|Privileges, OS Setup|
#### 6 Layers of Footprinting
|Layer|Description|Information Categories|
|-----|-----------|----------------------|
|`1. Internet Presence`|Identification of internet presence and externally accessible infrastructure.|Domains, Subdomains, vHosts, ASN, Netblocks, IP Addresses, Cloud Instances, Security Measures|
|`2. Gateway`|Identify the possible security measures to protect the company's external and internal infrastructure.|Firewalls, DMZ, IPS/IDS, EDR, Proxies, NAC, Network Segmentation, VPN, Cloudflare|
|`3. Accessible Services`|Identify accessible interfaces and services that are hosted externally or internally.|Service Type, Functionality, Configuration, Port, Version, Interface|
|`4. Processes`|Identify the internal processes, sources, and destinations associated with the services.|PID, Processed Data, Tasks, Source, Destination|
|`5. Privileges`|Identification of the internal permissions and privileges to the accessible services.|Groups, Users, Permissions, Restrictions, Environment|
|`6. OS Setup`|	Identification of the internal components and systems setup.|OS Type, Patch Level, Network config, OS Environment, Configuration files, sensitive private files|

* Internet Presence: Identify all possible targets and interfaces that can be tested.
* Gateway: Understand what we are dealing with and what we have to watch out for.
* Accessible Services: Understand the reason and functionality of the target and how to communicate with and exploit it.
* Processes: Understand the processes running on the machine and the dependencies between them.
* Priviliges: Identify and understand what and what is not possible with a given set of priviliges.
* OS Setup: Understand how the admin's manage the environment and what useful/sensitive information we can get from them.

## Infrastructure Based Enumeration
### Domain Information
#### Get an understanding of the company and it's services
* What does the company do?
* What technologies does the target company employ to accomplish it's mission?
* 
#### Online Presence
> Online presense can include the website, DNS records, basically any available information regarding the target's infrastructure.  
* Review the actual certificate in your browser.
* [crt.sh](https://crt.sh/) - Review the `SSL Cert` for the domain.
```bash
# Quickly get and filter out just the subdomains from a crt.sh query.
curl -s https://crt.sh/\?q\=inlanefreight.com\&output\=json | jq . | grep name | cut -d":" -f2 | grep -v "CN=" | cut -d'"' -f2 | awk '{gsub(/\\n/,"\n");}1;' | sort -u
```
* Look for company-hosted servers.
```bash
# We can use our list from the previous step
for i in $(cat subdomainlist);do host $i | grep "has address" | grep inlanefreight.com | cut -d" " -f1,4;done
```
* Gathering info with [shodan](https://www.shodan.io/)(Requires an API key)
```bash
# Reduce our list to just company hosted servers
for i in $(cat subdomainlist);do host $i | grep "has address" | grep inlanefreight.com | cut -d" " -f4 >> ip-addresses.txt;done
# Run our new list through Shodan to get more info
for i in $(cat ip-addresses.txt);do shodan host $i;done
```
