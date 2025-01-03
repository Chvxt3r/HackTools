# Cross-Site Scripting (XSS)

## Stored XSS (Persistent XSS)

### Testing
User input is shown on the page

Test Payloads

```html
<script>alert(window.origin)</script> # This will pop up a window shoing the source URL for the popup window(iframes)
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
<script>alert(window.origin)</script> # This will pop up a window shoing the source URL for the popup window(iframes)
<script>print()</script> # Will pop up the printer dialog
<plaintext> # Should render the rest of the page as plain text.
```
Payloads will NOT be rendered by the browser on execution

Targeting
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
