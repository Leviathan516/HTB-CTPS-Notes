# DNS

The **Domain Name System (DNS)** maps human-readable names to IP addresses, but it stores far more than that. DNS also holds information about a domain's mail servers, name servers, and various verification and security records — all of which are valuable during enumeration. DNS runs on port **53** (UDP for queries, TCP for zone transfers and large responses).

## Types of DNS Servers

| Server Type | Description |
| --- | --- |
| **DNS Root Server** | Responsible for the top-level domains (TLDs). Only queried as a last resort. Coordinated by ICANN; there are **13** root servers worldwide. |
| **Authoritative Nameserver** | Holds authority for a specific zone and gives binding answers for its area of responsibility. |
| **Non-authoritative Nameserver** | Not responsible for a zone; gathers zone information via recursive or iterative queries. |
| **Caching DNS Server** | Caches information from other nameservers for a period set by the authoritative server. |
| **Forwarding Server** | Forwards DNS queries to another DNS server — nothing more. |
| **Resolver** | Performs name resolution locally on a computer or router; not authoritative. |

## DNS Record Types

Each record type serves a different function. The records that aren't `A`/`AAAA` are often the most interesting for enumeration.

| Record | Description |
| --- | --- |
| `A` | Returns the IPv4 address for a domain. |
| `AAAA` | Returns the IPv6 address for a domain. |
| `MX` | Returns the responsible mail servers. |
| `NS` | Returns the domain's nameservers. |
| `TXT` | Free-form text; used for Google/SSL verification, and SPF/DMARC/DKIM mail-security records. |
| `CNAME` | An alias pointing one domain name at another. |
| `PTR` | Reverse lookup — converts an IP address back into a domain name. |
| `SOA` | Start of Authority; zone information and the admin's email contact. |

## Dangerous Settings

These directives control who can query and — critically — who can pull entire zones. An open `allow-transfer` is a classic finding.

| Option | Description |
| --- | --- |
| `allow-query` | Which hosts may send requests to the DNS server. |
| `allow-recursion` | Which hosts may send recursive requests. |
| `allow-transfer` | Which hosts may receive **zone transfers** — if set too broadly, leaks every record in the zone. |
| `zone-statistics` | Collects statistical data of zones. |

## Footprinting the Service

The `dig` tool is the workhorse for DNS enumeration. Point it at the target's nameserver with `@<IP>`.

**Query nameservers (NS records):**

```shell
dig ns inlanefreight.htb @10.129.14.128
```

**Query the server version** (can reveal the BIND version, useful for matching known CVEs):

```shell
dig CH TXT version.bind 10.129.120.85
```

**Query everything (ANY):**

```shell
dig any inlanefreight.htb @10.129.14.128
```

> **Why `ANY` is your first move:** A single `ANY` query returns A, MX, NS, TXT, and SOA records all at once, giving you a quick map of the domain's infrastructure and the third-party services it relies on. The next step is usually attempting a zone transfer (`dig axfr inlanefreight.htb @<NS>`) to see whether the server leaks its full zone.
