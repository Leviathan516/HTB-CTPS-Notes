# SMTP

The **Simple Mail Transfer Protocol (SMTP)** handles sending and relaying email. By default it listens on port **25**, though modern servers also use **587** (submission). Its relevance to enumeration is twofold: certain commands let you **verify which usernames exist** on a system, and misconfigured servers can act as **open relays**.

## Key Concepts

- **ESMTP and SMTP-Auth** — Modern servers use Extended SMTP, which supports authentication. You must log in before sending, which helps block anonymous spam.
- **MSA vs MTA** — The Mail Submission Agent handles *submission* and validates the sender; the Mail Transfer Agent handles *transfer and delivery*.
- **The open relay problem** — If an MSA/MTA forwards mail for anyone without authentication, spammers can abuse it to send bulk mail. Correct configuration and SMTP-Auth prevent this.

## SMTP Commands

| Command | Description |
| --- | --- |
| `AUTH PLAIN` | Service extension used to authenticate the client. |
| `HELO` | Client identifies itself by name and starts the session. |
| `MAIL FROM` | Names the email sender. |
| `RCPT TO` | Names the email recipient. |
| `DATA` | Begins transmission of the email body. |
| `RSET` | Aborts the current transmission but keeps the connection open. |
| `VRFY` | Checks whether a mailbox exists. |
| `EXPN` | Also checks whether a mailbox is available for messaging. |
| `NOOP` | Requests a response to keep the connection from timing out. |
| `QUIT` | Terminates the session. |

> **The enumeration value of `VRFY` and `EXPN`:** These commands were designed to confirm valid mailboxes — which means they confirm valid *usernames*. A server that answers them reveals which accounts exist, giving you a target list for password attacks against other services.

## Footprinting the Service

### Nmap

The default `smtp-commands` script issues `EHLO` and lists every command the server supports — including whether `VRFY` is available:

```shell
$ sudo nmap 10.129.14.128 -sC -sV -p25

PORT   STATE SERVICE VERSION
25/tcp open  smtp    Postfix smtpd
|_smtp-commands: mail1.inlanefreight.htb, PIPELINING, SIZE 10240000, VRFY, ETRN, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8, CHUNKING
```

> Note `mail1.inlanefreight.htb` in the output — the banner leaks the internal hostname/domain, and `VRFY` is listed as supported.

### Enumerating Users with smtp-user-enum

When `VRFY` is available, automate username discovery against a wordlist:

```shell
$ smtp-user-enum -M VRFY -U wordlist.txt -t 10.129.88.93 -m 60 -w 20

Mode ..................... VRFY
Username count ........... 101
Target TCP port .......... 25

######## Scan started ########
10.129.88.93: robin exists
######## Scan completed ########
1 results.
```

- `-M VRFY` — use the VRFY method.
- `-U wordlist.txt` — the list of usernames to test.
- `-t` — target IP.
- `-m 60` — number of worker processes; `-w 20` — query timeout in seconds.

The result (`robin exists`) is a confirmed valid account — feed it into credential attacks against SSH, IMAP/POP3, or other services on the same host.
