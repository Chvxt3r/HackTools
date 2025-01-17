# SQL Injection (SQLI)
## MySQL
### Basic Connection
```bash
mysql -u <user> -h <hostname or IP> -P <port> -p<password>
#note that there is no blank space between the "-p" and the password. If "-p" is included and left blank, the system
#will prompt for a password. Useful for complex characters in passwords.
```
### Basic Commands
MySQL expects command line query's to be terminated with a semicolon (;)

Create a database
```bash
CREATE DATABASE users;
```
List DB's
```bash
SHOW DATABASES;
```

Use a specific database
```bash
USE users;
```
Show Tables in a DB
```bash
SHOW TABLES;
```
List Table structure w/ fields and datatypes
```bash
DESCRIBE table;
```
Create Table
```bash
mysql> CREATE TABLE logins (
    ->     id INT,
    ->     username VARCHAR(100),
    ->     password VARCHAR(100),
    ->     date_of_joining DATETIME
    ->     );
Query OK, 0 rows affected (0.03 sec)
```

INSERT Statement
```bash
INSERT INTO table_name VALUES (column1_value, column2_value, column3_value, ...);
# Skipping columns with the 'NOT NULL' constraint will result in an error
```
```bash
INSERT INTO logins(username, password) Values('administrator', 'password');
# Example: Inserting a username and password into logins by defining the columns to be inserted into,
# specifically (username, password)
```

SELECT Statement
```bash
SELECT * FROM table_name;
# Selects every column from the specified table
```
```bash
SELECT column1, column2, FROM table_name;
# Selects only column1 and column2 from the specified table
```
DROP Statement
```bash
DROP TABLE table_name;
# Permanently and complete deletes the specified table. 
```
ALTER Statement
```bash
ALTER TABLE table_name ADD newColumn INT;
# Adds a new column named new Column to the specified table_name
```
```bash
ALTER TABLE table_name RENAME COLUMN newColumn TO newerColumn;
# Renames a newColumn to newColumn
```
```bash
ALTER TABLE table_name MODIFY newerColumn DATE;
# Changes the columns datatype from "INT" specified above to "DATE"
```
```bash
ALTER TABLE table_name DROP newerColumn;
# Permanently Drops a column from the table
```
UPDATE Statement
```bash
UPDATE table_name SET column1=newvalue1, column2=newvalue2, ... WHERE <condition>;
# Updates specific records in a table based on a condition
```
```bash
UPDATE logins SET password = 'change_password' WHERE id > 1;
# Example: sets every account with an id > 1 to new password 'change_password'
```

