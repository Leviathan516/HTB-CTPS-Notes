# SNMP

The **Simple Network Management Protocol (SNMP)** monitors and manages network devices, and can also change settings remotely. It exchanges information *and* transmits control commands via agents over **UDP port 161**. Because it often exposes detailed system information — and sometimes credentials — with weak or default access control, it's a rich enumeration target.

## Community Strings

SNMP access (in v1/v2c) is gated by a **community string**, which functions like a password. The defaults `public` (read-only) and `private` (read-write) are extremely common and frequently left unchanged. Because admins can name community strings arbitrarily, discovering them sometimes requires brute forcing.

## Dangerous Settings

| Setting | Description |
| --- | --- |
| `rwuser noauth` | Grants access to the full OID tree with no authentication. |
| `rwcommunity <string> <IPv4>` | Grants full OID-tree access regardless of source address. |
| `rwcommunity6 <string> <IPv6>` | Same as above, over IPv6. |

> **Why these are dangerous:** Read-write access to the OID tree can expose — and let you modify — system configuration, running processes, and more. Even read-only access often leaks usernames, email addresses, software inventories, and occasionally plaintext credentials embedded in process arguments.

## Footprinting the Service

Three tools cover most SNMP enumeration: **snmpwalk** (query OIDs), **onesixtyone** (brute-force community strings), and **braa** (fast bulk OID querying).

### snmpwalk

Walk the OID tree using a known community string (`-c public`, SNMP v2c):

```shell
$ snmpwalk -v2c -c public 10.129.14.128

iso.3.6.1.2.1.1.1.0 = STRING: "Linux htb 5.11.0-34-generic ... x86_64"
iso.3.6.1.2.1.1.4.0 = STRING: "mrb3n@inlanefreight.htb"
iso.3.6.1.2.1.1.5.0 = STRING: "htb"
iso.3.6.1.2.1.1.6.0 = STRING: "Sitting on the Dock of the Bay"
...SNIP...
```

> Already this leaks the OS and kernel version, an admin email (`mrb3n@inlanefreight.htb`), the hostname, and a location string — all useful intelligence.

### onesixtyone

If you don't know the community string, brute-force it with a wordlist (SecLists provides good ones):

```shell
$ sudo apt install onesixtyone
$ onesixtyone -c /opt/useful/seclists/Discovery/SNMP/snmp.txt 10.129.14.128

Scanning 1 hosts, 3220 communities
10.129.14.128 [public] Linux htb 5.11.0-37-generic ... x86_64
```

> The string in brackets (`[public]`) is the discovered community string.

### braa

Once you have a valid community string, `braa` queries large numbers of OIDs quickly:

```shell
$ sudo apt install braa
$ braa public@10.129.14.128:.1.3.6.*      # syntax: <community>@<IP>:<OID>

10.129.14.128:20ms:.1.3.6.1.2.1.1.1.0:Linux htb 5.11.0-34-generic ...
10.129.14.128:20ms:.1.3.6.1.2.1.1.4.0:mrb3n@inlanefreight.htb
10.129.14.128:20ms:.1.3.6.1.2.1.1.5.0:htb
...SNIP...
```

> **In practice (see the Hard lab):** Walking SNMP can reveal process command lines containing credentials — e.g. an OID showing `tom NMds732Js2761`, a username and password passed as script arguments. Always read the full walk carefully; the OID `.1.3.6.1.2.1.25.1.7` (running software / processes) is a frequent source of leaked secrets.
