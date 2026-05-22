# Linux Remote Management Protocols

This section covers three Linux remote-management services: **SSH**, **Rsync**, and the legacy **R-services**.

---

# SSH

**Secure Shell (SSH)** provides encrypted remote login and command execution, listening on **TCP port 22** by default. It's secure when configured well — but several settings weaken it.

## Dangerous Settings

| Setting | Description |
| --- | --- |
| `PasswordAuthentication yes` | Allows password-based authentication (brute-forceable). |
| `PermitEmptyPasswords yes` | Allows empty passwords. |
| `PermitRootLogin yes` | Allows direct root login. |
| `Protocol 1` | Uses an outdated, insecure encryption version. |
| `X11Forwarding yes` | Allows X11 forwarding for GUI apps. |
| `AllowTcpForwarding yes` | Allows TCP port forwarding. |
| `PermitTunnel` | Allows tunneling. |
| `DebianBanner yes` | Displays a specific banner when logging in. |

## Footprinting the Service

### ssh-audit

`ssh-audit` enumerates supported algorithms, the software version, and known weaknesses:

```shell
$ git clone https://github.com/jtesta/ssh-audit.git && cd ssh-audit
$ ./ssh-audit.py 10.129.14.132

(gen) banner: SSH-2.0-OpenSSH_8.2p1 Ubuntu-4ubuntu0.3
(gen) software: OpenSSH 8.2p1
(gen) compatibility: OpenSSH 7.4+, Dropbear SSH 2018.76+
```

### Forcing an Authentication Method

To confirm whether password auth is allowed, force it and read the verbose output:

```shell
$ ssh -v cry0l1t3@10.129.14.132 -o PreferredAuthentications=password

debug1: Authentications that can continue: publickey,password,keyboard-interactive
debug1: Next authentication method: password
cry0l1t3@10.129.14.132's password:
```

> **Using a recovered private key:** When you obtain an `id_rsa` (from FTP, NFS, an email, etc.), save it, set strict permissions, and connect:
> ```shell
> $ chmod 600 id_rsa          # SSH refuses keys readable by others
> $ ssh -i id_rsa user@target
> ```
> If the key is passphrase-protected, crack it: `ssh2john id_rsa > hash.txt` then `john --wordlist=rockyou.txt hash.txt`.

---

# Rsync

**Rsync** is a fast tool for copying files locally and remotely. It uses **port 873** by default and can also tunnel over SSH.

## Scanning for Rsync

```shell
$ sudo nmap -sV -p 873 127.0.0.1

PORT    STATE SERVICE VERSION
873/tcp open  rsync   (protocol version 31)
```

## Probing for Accessible Shares

Connect with netcat and request the module list:

```shell
$ nc -nv 127.0.0.1 873

@RSYNCD: 31.0
@RSYNCD: 31.0
#list
dev             Dev Tools
@RSYNCD: EXIT
```

## Enumerating an Open Share

List a share's contents without downloading:

```shell
$ rsync -av --list-only rsync://127.0.0.1/dev

drwxr-xr-x  48  .
-rw-r--r--   0  build.sh
-rw-r--r--   0  secrets.yaml      <- worth investigating
drwx------  54  .ssh              <- likely contains SSH keys
```

> Pull everything down with `rsync -av rsync://127.0.0.1/dev`. If Rsync runs over SSH, add `-e ssh` (or `-e "ssh -p2222"` for a non-standard port).

---

# R-Services

Legacy Berkeley **R-services** span ports **512, 513, and 514**, accessed via a suite of `r-commands`. They are dangerously trusting — authentication can be bypassed entirely by trusted entries in `/etc/hosts.equiv` and `.rhosts`.

The suite: `rcp`, `rexec`, `rlogin`, `rsh`, `rstat`, `ruptime`, `rwho`.

| Command | Daemon | Port | Protocol | Description |
| --- | --- | --- | --- | --- |
| `rcp` | `rshd` | 514 | TCP | Copies files between systems (like `cp`, but **no overwrite warning**). |
| `rsh` | `rshd` | 514 | TCP | Opens a remote shell with no login procedure; trusts `/etc/hosts.equiv` and `.rhosts`. |
| `rexec` | `rexecd` | 512 | TCP | Runs shell commands remotely; auth over an **unencrypted** socket, overridden by trust files. |
| `rlogin` | `rlogind` | 513 | TCP | Remote login (like telnet, Unix-only); auth overridden by trust files. |

## Scanning for R-Services

```shell
$ sudo nmap -sV -p 512,513,514 10.0.17.2

PORT    STATE SERVICE    VERSION
512/tcp open  exec?
513/tcp open  login?
514/tcp open  tcpwrapped
```

> **Why R-services are dangerous:** The trust-file model means that if a host appears in `/etc/hosts.equiv` or a user's `.rhosts`, it can log in or run commands with **no password at all**. Finding these services open is a strong lead toward unauthenticated access.
