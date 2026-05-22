# Domain Information (Passive Recon)

Before touching the target directly, you can learn a great deal **passively** — by querying public sources rather than the target's own infrastructure. This is part of Layer 1 (Internet Presence) enumeration: identifying domains, subdomains, and the third-party services a company depends on, all without sending a single packet to the target.

## Useful Passive Sources

- **[crt.sh](https://crt.sh)** — searches Certificate Transparency logs. Because every TLS certificate is logged publicly, this reveals subdomains the company may not have intended to expose.
- **[Shodan](https://www.shodan.io)** — indexes internet-connected devices and their banners, letting you find a company's externally exposed services.

## Reading DNS Records for Intelligence

Running `dig any` against a domain returns the records, but the *interpretation* is where the value lies:

```shell
$ dig any inlanefreight.com

inlanefreight.com.  300    IN  A     10.129.27.33
inlanefreight.com.  300    IN  A     10.129.95.250
inlanefreight.com.  3600   IN  MX    1 aspmx.l.google.com.
inlanefreight.com.  21600  IN  NS    ns.inwx.net.
inlanefreight.com.  3600   IN  TXT   "MS=ms92346782372"
inlanefreight.com.  21600  IN  TXT   "atlassian-domain-verification=..."
inlanefreight.com.  3600   IN  TXT   "google-site-verification=..."
inlanefreight.com.  3600   IN  TXT   "logmein-verification-code=..."
inlanefreight.com.  300    IN  TXT   "v=spf1 include:mailgun.org include:_spf.google.com ..."
inlanefreight.com.  21600  IN  SOA   ns.inwx.net. hostmaster.inwx.net. ...
```

### What Each Record Tells You

- **A records** — IP addresses pointing to a (sub)domain. Note any addresses you didn't already know.
- **MX records** — the mail servers. Here mail is handled by Google, so note it and move on.
- **NS records** — the nameservers. Most hosting providers use their own, so these often identify the hosting provider.
- **TXT records** — frequently the richest source. They hold verification keys for third-party providers and mail-security records (SPF, DMARC, DKIM).

## Turning TXT Records into a Provider Map

The verification strings reveal which external services the company uses. Each one is a potential avenue:

| Provider | What it implies / why it matters |
| --- | --- |
| **Atlassian** | Software development & collaboration (Jira, Confluence) — potential exposed boards or wikis. |
| **Google Gmail** | Email via Google; possibly open Google Drive folders or links. |
| **LogMeIn** | Centralized remote access. Compromising an admin here (e.g. via password reuse) can yield access to *all* connected systems. |
| **Mailgun** | Email APIs, SMTP relays, webhooks — watch for API endpoints testable for IDOR, SSRF, etc. |
| **Outlook / Office 365** | Document management; often paired with OneDrive and Azure storage. Azure file storage uses SMB and can be very interesting. |
| **INWX** | A hosting/domain registrar. The `MS=` TXT value is often similar to the login username/ID for the management platform. |

> **The principle in action:** None of this required attacking the target. By reasoning about *what we see* (verification strings) and *what it implies* (the services behind them), you build a picture of the company's attack surface entirely from public data.
