# Common Tools
The most common tools I use, mostly for recon.

## Searches from Linux
### Grep
* Search recursively the contents of files
  ```bash
  grep -R "<search term>" <PATH>
  ```
* Search a file for any IP addresses
  ```bash
  grep -oE '((1?[0-9][0-9]?|2[0-4][0-9]|25[0-5])\.){3}(1?[0-9][0-9]?|2[0-4][0-9]|25[0-5])' FILE
  ```
### sed

## Time Tools
### Faketime
> Useful for kerberos when your time is off
Install libfaketime
* Sync your time with a destination time server for once command only
  ```bash
  faketime "$(ntpdate -q <ip of timesource> | cut -d'' -f 1,2)" <command>
  ```
* Add a set amount of time to your command
  ```bash
  faketime -f '+7h' <command>
  ```
  
