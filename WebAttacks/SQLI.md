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
