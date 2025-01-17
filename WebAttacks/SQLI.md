# SQL Injection
### Summary
Injection occurs user-input is put into the SQL query string without properly sanitizing or filtering the input
## Common
### Authentication Bypass
```sql
# Basic Admin Panel Query String
SELECT * FROM logins WHERE username = 'admin' AND password = 'p@ssw0rd';
# This statement returns True based on the "AND" Statement if both the username and password match the same entry in sql,
# and thus allows the login
```
## In-Band
### Summary
Output of both the intended and new query are output directly to the screen and can be directly read.
### Union Based
### Error Based
## Blind
### Summary
Output may not printed, so we'll have to use SQL logic to get the output character by character
### Boolean Based
### Time Based
## Out-of-band
### Summary
No direct access to the output, so we have to direct the output to a remote location, like a DNS record, and retrieve it from there.
