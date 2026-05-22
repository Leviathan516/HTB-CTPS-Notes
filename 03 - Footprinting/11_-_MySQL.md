# MySQL

**MySQL** is a widely used open-source relational database management system. It listens on **TCP port 3306** by default. During enumeration it's a prime target because it stores application data — and that data often includes credentials, personal information, or other secrets.

## Footprinting the Service

### Nmap

The MySQL Nmap scripts pull version info, capabilities, and attempt account enumeration and brute forcing:

```shell
PORT     STATE SERVICE VERSION
3306/tcp open  mysql   MySQL 8.0.27-0ubuntu0.20.04.1
| mysql-info:
|   Protocol: 10
|   Version: 8.0.27-0ubuntu0.20.04.1
|   Auth Plugin Name: caching_sha2_password
```

### Connecting

Connect with discovered or guessed credentials. **There is no space between `-p` and the password:**

```shell
$ mysql -u robin -probin -h 10.129.99.102 --ssl-verify-server-cert=false
```

## Core SQL Commands

| Command | Description |
| --- | --- |
| `mysql -u <user> -p<password> -h <IP>` | Connect to the server (no space after `-p`). |
| `show databases;` | List all databases. |
| `use <database>;` | Select a database. |
| `show tables;` | List tables in the selected database. |
| `show columns from <table>;` / `describe <table>;` | Show a table's columns. |
| `select * from <table>;` | Show everything in a table. |
| `select * from <table> where <column> = "<string>";` | Filtered search. |

> **Default vs. non-default databases:** Every MySQL server ships with `information_schema`, `mysql`, `performance_schema`, and `sys`. Any database *other* than those four was created by the application or admin — that's where the interesting data lives.

> **Statement terminator:** MySQL won't execute a statement until it sees a `;`. If your prompt changes to `->`, it's waiting for you to finish — type `;` and press Enter, or `\c` to cancel.

## Worked Example

```sql
MySQL [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| customers          |   <- non-default, created by the app
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+

MySQL [(none)]> use customers
MySQL [customers]> show tables;
+---------------------+
| Tables_in_customers |
+---------------------+
| myTable             |
+---------------------+

MySQL [customers]> describe myTable;
+-----------+--------------------+ ... +
| Field     | Type               | ... |
+-----------+--------------------+ ... +
| id        | mediumint unsigned | ... |
| name      | varchar(255)       | ... |
| email     | varchar(255)       | ... |
| ...       | ...                | ... |
| pan       | varchar(255)       | ... |   <- card data!
| cvv       | varchar(255)       | ... |
+-----------+--------------------+ ... +

MySQL [customers]> SELECT email FROM myTable WHERE name='Otto Lang';
+---------------------+
| email               |
+---------------------+
| ultrices@google.htb |
+---------------------+
```

## Dangerous Settings

| Setting | Description |
| --- | --- |
| `user` | The OS user the MySQL service runs as. |
| `password` | Password for the MySQL user. |
| `admin_address` | IP address listening for admin-network TCP/IP connections. |
| `debug` | Current debugging settings. |
| `sql_warnings` | Whether single-row INSERTs produce an info string on warnings. |
| `secure_file_priv` | Limits the effect of data import/export operations (file read/write). |

---

## Module Answers

> **1. Determine the MySQL version in use.** (Format: `MySQL X.X.XX`)
> **`MySQL 8.0.27`**

> **2. Using weak credentials `robin:robin`, what is the email address of customer "Otto Lang"?**
> **`ultrices@google.htb`**
