# HashCat

Hitting "s" while running shows status

## Identification
### Hashcat Examples
```bash
hashcat --example-hashes
```
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

## Hashcat Cracking
### Hashcat Installation
```bash
sudo apt install hashcat
```
### Hashcat Benchmarking
```bash
# Specific mode
hashcat -b -m 0

# All Modes
hashcat -b
```

### Dictionary Attack
```bash
hashcat -a 0 -m <hash type> <hashfile> <wordlist>
```

### Combination Attack
Takes 2 lists as input and combines them
```bash
# View the output of combined wordlists
hashcat -a 1 --stdout <wordlist1> <wordlist2>
```

```bash
hashcat -a 1 -m <hash type> <hash file> <wordlist1> <wordlist2>
```

### Mask Attack
Used to generate words matching a specific Pattern
?l - Lower-case ASCII letters (a-z)  
?u - Upper-case ASCII letters (A-Z)  
?d - digits (0-9)  
?h - 0123456789abcdef  
?H - 0123456789ABCDEF  
?s - Special Characters (<space> !"#$%&'()*+,-./:;<=>?@[]^_`{})  
?a - ?l?u?d?s  
?b - 0x00 -0xff  

Can be combined with options "-1" to "-4" which can be used for custom placeholders

Example
```bash
hashcat -a 3 -m 0 md5_mask_example_hash -1 01 'ILFREIGHT?l?l?l?l?l20?1?d'
# Notice the placeholder "-1" maps to "?1" and is specified to be either 0 or 1
```
--increment can be used to increment the mask length automatically, and --increment-max flag used to cap the increment

### Hybrid Mode
Useful for when you have a general idea of the orgs password policy. Hybrid mode allows you to append or prepend a  
mask to words in a wordlist.
Append - Mode 6
Prepend - Mode 7

Example
```bash
# Append
hashcat -a 6 -m 0 -1 01 <hashfile> <wordlist> '20?1?d'

# Prepend
hashcat -a 7 -m 0 <hashfile> -1 01 '20?1?d' <wordlist>
```
## Custom Wordlists
### Crunch
```bash
crunch <min length> <max length> <charset> -t <pattern> -o <outfile>
```
Crunch Patterns
@ - Lower case Characters  
, - Upper Case Characters  
% - Numbers  
^ - Symbols  

-d - Specify the amount of repetition

Example
```bash
crunch 17 17 -t ILFREIGHT201%@@@@ -o wordlist
# Result = min/max 17 char's, ILFREIGHT2010aaaa
```
### CUPP
Use - i to run in interactive mode
```bash
python3 cupp.py -i
```
### CeWL
Used to scrape possible passwords form websites
Usage:
```bash
cewl -d <spider depth> -m <min word length> -w <output wordlist> <url of website>
```
Useful options
-m - Alter the length of outputted words  
-e - extract emails  

Example:
```bash
cewl -d 5 -m 8 -e http://performanceltg.com/blog -w wordlist.txt
```

