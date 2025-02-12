# HashCat


## Identification
### HashID
Installation
```bash
## Install with apt
sudo apt install hashid
## Install with PIP
pipx install hashid
```
Usage  
Hashes can be supplied as arguments or using a file
```bash
# Arguments
hashid '$apr1$71850310$gh9m4xcAn3MGxogwX/ztb.'

# File
hashid hashes.txt
```
Use '-m' flag to have hashid provide the hashcat mode  

