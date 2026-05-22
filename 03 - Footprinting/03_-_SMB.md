# SMB

The **Server Message Block (SMB)** protocol shares files, printers, and other resources across a network. An SMB server exposes arbitrary parts of its local filesystem as **shares**, so the hierarchy a client sees is partially independent of the server's actual structure. Access is governed by **Access Control Lists (ACLs)**, which can grant fine-grained rights (`execute`, `read`, `full access`) per user or group, and are defined at the share level rather than mirroring local filesystem permissions.

## Samba and CIFS

**Samba** is the Unix/Linux implementation of the SMB server. It implements the **Common Internet File System (CIFS)** — a dialect of SMB — allowing Unix hosts to communicate with Windows systems. CIFS aligns primarily with **SMB version 1**.

- CIFS / older NetBIOS connections use TCP ports **137, 138, and 139**.
- Modern CIFS over direct TCP uses port **445** exclusively.

| SMB Version | Supported By | Key Features |
| --- | --- | --- |
| CIFS | Windows NT 4.0 | Communication via NetBIOS interface |
| SMB 1.0 | Windows 2000 | Direct connection via TCP |
| SMB 2.0 | Windows Vista / Server 2008 | Performance upgrades, improved signing, caching |
| SMB 2.1 | Windows 7 / Server 2008 R2 | Locking mechanisms |
| SMB 3.0 | Windows 8 / Server 2012 | Multichannel, end-to-end encryption, remote storage |
| SMB 3.0.2 | Windows 8.1 / Server 2012 R2 | — |
| SMB 3.1.1 | Windows 10 / Server 2016 | Integrity checking, AES-128 encryption |

### Workgroups and NetBIOS

Each host on an SMB network participates in a **workgroup** — a name identifying a collection of computers and their shared resources. IBM's **NetBIOS** API lets applications connect and share data. When a machine comes online it registers a name, either reserving its own hostname or using a **NetBIOS Name Server (NBNS)**, later enhanced into **WINS (Windows Internet Name Service)**.

## Default Configuration (`/etc/samba/smb.conf`)

| Setting | Description |
| --- | --- |
| `[sharename]` | The name of the network share. |
| `workgroup = WORKGROUP/DOMAIN` | Workgroup shown when clients query. |
| `path = /path/here/` | Directory the user is given access to. |
| `server string = STRING` | String shown when a connection is initiated. |
| `unix password sync = yes` | Sync the UNIX password with the SMB password. |
| `usershare allow guests = yes` | Allow unauthenticated users to access defined shares. |
| `map to guest = bad user` | What to do when a login doesn't match a valid UNIX user. |
| `browseable = yes` | Show this share in the list of available shares. |
| `guest ok = yes` | Allow connecting without a password. |
| `read only = yes` | Allow read-only access. |
| `create mask = 0700` | Permissions for newly created files. |

## Dangerous Settings

Loose Samba settings can hand you read/write access, guest logins, or even script execution on login.

| Setting | Description |
| --- | --- |
| `browseable = yes` | Allow listing available shares. |
| `read only = no` | Allow creating and modifying files. |
| `writable = yes` | Allow users to create and modify files. |
| `guest ok = yes` | Allow connecting without a password. |
| `enable privileges = yes` | Honor privileges assigned to a specific SID. |
| `create mask = 0777` | Permissions assigned to newly created files. |
| `directory mask = 0777` | Permissions assigned to newly created directories. |
| `logon script = script.sh` | Script executed on user login. |
| `magic script = script.sh` | Script executed when the script session closes. |
| `magic output = script.out` | Where the magic script's output is stored. |

## Footprinting the Service

### rpcclient

`rpcclient` queries the server over MS-RPC. A null session (`-U ""`) often works against misconfigured servers:

```shell
$ rpcclient -U "" 10.129.14.128
Enter WORKGROUP\'s password:
rpcclient $>
```

| Query | Description |
| --- | --- |
| `srvinfo` | Server information. |
| `enumdomains` | Enumerate all deployed domains. |
| `querydominfo` | Domain, server, and user information. |
| `netshareenumall` | Enumerate all available shares. |
| `netsharegetinfo <share>` | Information about a specific share. |
| `enumdomusers` | Enumerate all domain users. |
| `queryuser <RID>` | Information about a specific user. |

### Brute Forcing User RIDs

Even without a user list, you can sweep through Relative Identifiers (RIDs) to discover account names. RIDs are predictable (e.g. `0x3e8` = 1000 is typically the first normal user):

```shell
$ for i in $(seq 500 1100); do \
    rpcclient -N -U "" 10.129.14.128 -c "queryuser 0x$(printf '%x\n' $i)" \
    | grep "User Name\|user_rid\|group_rid" && echo ""; \
  done

        User Name   :   sambauser
        user_rid :      0x1f5
        group_rid:      0x201

        User Name   :   mrb3n
        user_rid :      0x3e8
        group_rid:      0x201

        User Name   :   cry0l1t3
        user_rid :      0x3e9
        group_rid:      0x201
```

### samrdump.py (Impacket)

An alternative that dumps users and their attributes:

```shell
$ samrdump.py 10.129.14.128

Found domain(s):
 . DEVSMB
 . Builtin
[*] Looking up users in domain DEVSMB
Found user: mrb3n, uid = 1000
Found user: cry0l1t3, uid = 1001
mrb3n (1000)/PasswordLastSet: 2021-09-22 17:47:59
cry0l1t3 (1001)/FullName: cry0l1t3
[*] Received 2 entries.
```

### SMBmap

Maps shares and shows your permission level on each:

```shell
$ smbmap -H 10.129.14.128

        Disk        Permissions     Comment
        ----        -----------     -------
        print$      NO ACCESS       Printer Drivers
        home        NO ACCESS       INFREIGHT Samba
        dev         NO ACCESS       DEVenv
        notes       NO ACCESS       CheckIT
        IPC$        NO ACCESS       IPC Service (DEVSM)
```

### CrackMapExec

Enumerate shares with empty credentials, and note the SMB version and signing status it reports:

```shell
$ crackmapexec smb 10.129.14.128 --shares -u '' -p ''

SMB  10.129.14.128  445  DEVSMB  [*] Windows 6.1 (name:DEVSMB) (signing:False) (SMBv1:False)
SMB  10.129.14.128  445  DEVSMB  [+] Enumerated shares
SMB  10.129.14.128  445  DEVSMB  Share    Permissions  Remark
SMB  10.129.14.128  445  DEVSMB  notes    READ,WRITE   CheckIT
```

### enum4linux-ng

A comprehensive automated enumeration tool (`-A` runs all checks):

```shell
$ ./enum4linux-ng.py 10.129.14.128 -A
[+] SMB is accessible on 445/tcp
[+] SMB over NetBIOS is accessible on 139/tcp
----SNIP----
```

> **Methodology note:** Use more than one tool. Differences in how tools are written mean they sometimes surface different information, so cross-check findings manually rather than trusting any single automated result.
