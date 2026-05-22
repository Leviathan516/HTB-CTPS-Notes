# IMAP / POP3

These two protocols let clients **retrieve** email from a server (whereas SMTP *sends* it).

- **IMAP (Internet Message Access Protocol)** — manages email *online, directly on the server*, and supports folder structures. Connects on port **143** (or **993** for TLS).
- **POP3 (Post Office Protocol)** — typically downloads mail to the client. Connects on port **110** (or **995** for TLS).

The higher ports (**993** and **995**) wrap the connection in TLS/SSL.

## IMAP Commands

Commands are prefixed with a tag (here `1`) that the server echoes in its reply.

| Command | Description |
| --- | --- |
| `1 LOGIN username password` | Log in. |
| `1 LIST "" *` | List all directories. |
| `1 CREATE "INBOX"` | Create a mailbox with the given name. |
| `1 DELETE "INBOX"` | Delete a mailbox. |
| `1 RENAME "ToRead" "Important"` | Rename a mailbox. |
| `1 LSUB "" *` | List subscribed/active mailboxes. |
| `1 SELECT INBOX` | Select a mailbox so messages can be accessed. |
| `1 UNSELECT INBOX` | Exit the selected mailbox. |
| `1 FETCH <ID> all` | Retrieve data for a message. |
| `1 CLOSE` | Remove all messages flagged `Deleted`. |
| `1 LOGOUT` | Close the connection. |

## POP3 Commands

| Command | Description |
| --- | --- |
| `USER username` | Identify the user. |
| `PASS password` | Authenticate with the password. |
| `STAT` | Number of saved emails. |
| `LIST` | Number and size of all emails. |
| `RETR id` | Deliver the email with the given ID. |
| `DELE id` | Delete the email with the given ID. |
| `CAPA` | Display server capabilities. |
| `RSET` | Reset transmitted information. |
| `QUIT` | Close the connection. |

## Dangerous Settings

These Dovecot debug/verbose options can leak credentials straight into logs:

| Setting | Description |
| --- | --- |
| `auth_debug` | Enables all authentication debug logging. |
| `auth_debug_passwords` | Logs submitted passwords and the auth scheme. |
| `auth_verbose` | Logs failed authentication attempts and reasons. |
| `auth_verbose_passwords` | Logs (possibly truncated) passwords used for auth. |
| `auth_anonymous_username` | Username used for ANONYMOUS SASL logins. |

## Footprinting the Service

### Nmap

Scan all four mail-retrieval ports. The scripts pull capabilities and the embedded TLS certificate — which often leaks the internal hostname (`commonName`):

```shell
$ sudo nmap 10.129.14.128 -sV -p110,143,993,995 -sC

PORT    STATE SERVICE  VERSION
110/tcp open  pop3     Dovecot pop3d
143/tcp open  imap     Dovecot imapd
993/tcp open  ssl/imap Dovecot imapd
| ssl-cert: Subject: commonName=mail1.inlanefreight.htb/organizationName=Inlanefreight/...
995/tcp open  ssl/pop3 Dovecot pop3d
```

> The certificate's `commonName=mail1.inlanefreight.htb` is a free hostname/domain disclosure — note it.

### Interacting Over TLS

Connect to the encrypted IMAP port with `openssl` to issue commands manually:

```shell
$ openssl s_client -connect 10.129.88.93:imaps
```

Once connected, you can log in and read mail directly — for example, after recovering credentials elsewhere:

```text
1 LOGIN username password
1 LIST "" *
1 SELECT INBOX
1 FETCH 1 BODY[]
```

> **Why this matters in practice:** Mailboxes frequently contain exactly the secrets you're hunting — password reset emails, configuration snippets, or (as in the Hard lab) a full SSH private key pasted into a message body. Once you have IMAP credentials, always read the inbox.
