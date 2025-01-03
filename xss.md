# Cross-Site Scripting (XSS)

## Stored XSS (Persistent XSS)

### Testing
User input is shown on the page

Basic Payloads

```html
<script>alert(window.origin)</script> # This will pop up a window shoing the source URL for the popup window(iframes)
<script>print()</script> # Will pop up the printer dialog
<plaintext> # Should render the rest of the page as plain text.
'''

