# MSSQL

**Microsoft SQL Server (MSSQL)** is Microsoft's closed-source relational database management system, originally written for Windows. It's popular for applications built on the **.NET framework** thanks to strong native support. While Linux and macOS versions exist, you'll most often encounter MSSQL on Windows targets. It listens on **TCP port 1433** by default.

## MSSQL Clients

**SQL Server Management Studio (SSMS)** is the primary admin client. Because it's a *client-side* application, it can be installed on any machine an admin manages the database from — not just the server. This means a compromised workstation with SSMS and **saved credentials** can hand you database access.

Other clients include: `mssql-cli`, SQL Server PowerShell, HeidiSQL, SQLPro, and **Impacket's `mssqlclient.py`**. Pentesters favour `mssqlclient.py` because Impacket ships on most pentesting distros:

```shell
$ locate mssqlclient
/usr/bin/impacket-mssqlclient
/usr/share/doc/python3-impacket/examples/mssqlclient.py
```

## Default System Databases

Understanding these helps you recognize which databases are *non-default* (i.e. created by the app/admin):

| Database | Description |
| --- | --- |
| `master` | Tracks all system information for the SQL Server instance. |
| `model` | Template for every new database; changes here propagate to new databases. |
| `msdb` | Used by SQL Server Agent to schedule jobs and alerts. |
| `tempdb` | Stores temporary objects. |
| `resource` | Read-only database containing system objects. |

## Default Configuration

When configured for network access, the SQL service typically runs as `NT SERVICE\MSSQLSERVER`. Client connections can use **Windows Authentication**, and **encryption is not enforced** by default.

> **Windows Authentication explained:** With this setting, the underlying Windows OS processes the login — checking the local **SAM database** or the **domain controller (Active Directory)** before granting access. Great for auditing, but a compromised account can lead to privilege escalation and lateral movement across the domain.

## Dangerous Settings

- MSSQL clients connecting **without encryption**.
- **Self-signed certificates** (which can be spoofed) when encryption *is* used.
- The use of **named pipes**.
- **Weak or default `sa` credentials** — admins often forget to disable this account.

## Footprinting the Service

### Nmap Script Scan

Target port 1433 with the `ms-sql-*` scripts:

```shell
$ sudo nmap --script ms-sql-info,ms-sql-ntlm-info,ms-sql-empty-password \
    -sV -p 1433 10.129.201.248

| ms-sql-ntlm-info:
|   Target_Name: SQL-01
|   NetBIOS_Computer_Name: SQL-01
|   DNS_Computer_Name: SQL-01
|_  Product_Version: 10.0.17763
| ms-sql-info:
|     Version: Microsoft SQL Server 2019 RTM (15.00.2000.00)
|     TCP port: 1433
|_    Named pipe: \\10.129.201.248\pipe\sql\query
```

### Metasploit – mssql_ping

The `mssql_ping` auxiliary scanner returns the server name, instance, version, and named pipe:

```shell
msf6 > use auxiliary/scanner/mssql/mssql_ping
msf6 auxiliary(scanner/mssql/mssql_ping) > set rhosts 10.129.201.248
msf6 auxiliary(scanner/mssql/mssql_ping) > run

[+]    ServerName      = SQL-01
[+]    InstanceName    = MSSQLSERVER
[+]    Version         = 15.0.2000.5
[+]    tcp             = 1433
```

### Connecting with mssqlclient.py

With valid credentials you can connect and run **T-SQL (Transact-SQL)** directly.

> **`-windows-auth` is the key flag:** If the account is a *Windows* account (validated against the SAM/AD) rather than a native SQL login, you **must** add `-windows-auth`. Without it, Impacket attempts SQL authentication and the server rejects it with "Login failed."

```shell
$ impacket-mssqlclient Administrator@10.129.201.248 -windows-auth

[*] Encryption required, switching to TLS
SQL> select name from sys.databases
```

---

## Module Answers

> **1. List the hostname of the MSSQL server.**
> **`ILF-SQL-01`**
>
> Found via `mssql_ping` in Metasploit (`ServerName = ILF-SQL-01`).

> **2. Connect with `backdoor:Password1` and list the non-default database.**
> **`Employees`**
>
> `backdoor` is a **Windows** account, so the `-windows-auth` flag is required:
> ```shell
> $ impacket-mssqlclient backdoor:Password1@10.129.230.249 -windows-auth
> SQL (ILF-SQL-01\backdoor  dbo@master)> select name from sys.databases
> name
> ---------
> master
> tempdb
> model
> msdb
> Employees     <- the non-default database
> ```
