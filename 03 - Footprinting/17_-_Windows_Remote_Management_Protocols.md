# Windows Remote Management Protocols

This section covers two ways to manage Windows hosts remotely: **WinRM** and **WMI**.

---

# WinRM

**Windows Remote Management (WinRM)** is Microsoft's built-in, command-line-based remote management protocol. It relies on two TCP ports:

- **5985** — HTTP
- **5986** — HTTPS

A companion component, **Windows Remote Shell (WinRS)**, lets you execute arbitrary commands on the remote system.

## Footprinting the Service

Scan the two WinRM ports. The Microsoft HTTPAPI banner is the giveaway:

```shell
$ nmap -sV -sC 10.129.201.248 -p5985,5986 --disable-arp-ping -n

PORT     STATE SERVICE VERSION
5985/tcp open  http    Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

## Connecting with evil-winrm

With valid credentials, `evil-winrm` gives you an interactive PowerShell session:

```shell
$ evil-winrm -i 10.129.201.248 -u Cry0l1t3 -p P455w0rD!
```

> **Why WinRM matters:** Unlike RDP, WinRM gives you a fully scriptable shell with no GUI overhead. A single valid credential pair turns into remote command execution — making it one of the most direct paths to a foothold on a Windows host.

---

# WMI

**Windows Management Instrumentation (WMI)** is Microsoft's infrastructure for management data and operations. WMI communication **initializes on TCP port 135**, then moves to a randomly assigned port once the connection is established.

## Footprinting the Service

Impacket's `wmiexec.py` executes commands over WMI. Here it runs `hostname` on the target:

```shell
$ wmiexec.py Cry0l1t3:"P455w0rD!"@10.129.201.248 "hostname"

[*] SMBv3.0 dialect used
ILF-SQL-01
```

> **WMI vs WinRM:** Both achieve remote command execution with valid credentials. WMI's use of port 135 (plus a dynamic high port) is worth remembering — if you see 135 open on a Windows host and you have credentials, `wmiexec.py` is an immediate option for command execution and confirming the hostname.
