# HTB Academy — Footprinting Module Notes

Polished, reorganized study notes for the Footprinting module, with brief explanations of the *why* behind each technique. Lab walkthroughs preserve every credential and answer.

## Foundations
- [01 — Enumeration Principles](01_-_Enumeration_Principles.md)
- [15 — Enumeration Methodology](15_-_Enumeration_Methodology.md)

## Passive Recon
- [05 — DNS](05_-_DNS.md)
- [06 — Domain Information](06_-_Domain_Information.md)
- [07 — Cloud Resources](07_-_Cloud_Resources.md)

## File & Share Services
- [02 — FTP](02_-_FTP.md)
- [03 — SMB](03_-_SMB.md)
- [04 — NFS](04_-_NFS.md)

## Mail Services
- [08 — SMTP](08_-_SMTP.md)
- [09 — IMAP / POP3](09_-_IMAP_POP3.md)

## Databases
- [11 — MySQL](11_-_MySQL.md)
- [12 — MSSQL](12_-_MSSQL.md)
- [13 — Oracle TNS](13_-_Oracle_TNS.md)

## Management & Monitoring
- [10 — SNMP](10_-_SNMP.md)
- [14 — IPMI](14_-_IPMI.md)
- [16 — Linux Remote Management (SSH, Rsync, R-services)](16_-_Linux_Remote_Management_Protocols.md)
- [17 — Windows Remote Management (WinRM, WMI)](17_-_Windows_Remote_Management_Protocols.md)

## Lab Walkthroughs
- [18 — Footprinting Lab: Easy](18_-_Footprinting_Lab_-_Easy.md)
- [19 — Footprinting Lab: Medium](19_-_Footprinting_Lab_-_Medium.md)
- [20 — Footprinting Lab: Hard](20_-_Footprinting_Lab_-_Hard.md)

---

## Quick Reference — Default Ports

| Service | Port(s) |
| --- | --- |
| FTP | 21 (control), 20 (active data) |
| SSH | 22 |
| SMTP | 25, 587 |
| DNS | 53 |
| POP3 | 110, 995 (TLS) |
| RPC (NFS/SMB) | 111 |
| IMAP | 143, 993 (TLS) |
| SNMP | 161 (UDP) |
| SMB / CIFS | 445 (139, 137/138 for NetBIOS) |
| IPMI | 623 (UDP) |
| Rsync | 873 |
| R-services | 512, 513, 514 |
| MSSQL | 1433 |
| Oracle TNS | 1521 |
| NFS | 2049 |
| MySQL | 3306 |
| RDP | 3389 |
| WinRM | 5985 (HTTP), 5986 (HTTPS) |
| WMI | 135 (then random) |

## Recurring Themes Across the Labs
1. **Credential chaining** — one leaked credential unlocks another service, which leaks the next.
2. **Credential reuse** — always try recovered credentials against *every* other service on the host.
3. **Scan UDP too** — the Hard lab's entire path opened up only through SNMP on 161/udp.
4. **Read the leftovers** — empty-looking shares, certificates, banners, and email bodies all hide secrets.
5. **`chmod 600`** any recovered private key before SSH will use it.
