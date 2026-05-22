# FTP

The **File Transfer Protocol (FTP)** transfers files between a client and server over the network. It typically listens on **TCP port 21** (control channel), with data transferred on port 20 (active mode) or a high random port (passive mode). FTP is plaintext by default, which makes it a frequent source of leaked credentials and exposed files during enumeration.

`vsFTPd` is one of the most common Linux FTP servers. Its behaviour is driven entirely by its config file, so understanding that file tells you what the server will and won't allow.

## vsFTPd Config File (`/etc/vsftpd.conf`)

| Setting | Description |
| --- | --- |
| `listen=NO` | Run from inetd, or as a standalone daemon? |
| `listen_ipv6=YES` | Listen on IPv6? |
| `anonymous_enable=NO` | Enable anonymous access? |
| `local_enable=YES` | Allow local users to log in? |
| `dirmessage_enable=YES` | Display messages when users enter certain directories? |
| `use_localtime=YES` | Use local time? |
| `xferlog_enable=YES` | Log uploads/downloads? |
| `connect_from_port_20=YES` | Connect from port 20? |
| `secure_chroot_dir=/var/run/vsftpd/empty` | Name of an empty directory. |
| `pam_service_name=vsftpd` | The PAM service vsftpd will use. |
| `rsa_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem` | Location of the RSA certificate for SSL connections. |
| `rsa_private_key_file=/etc/ssl/private/ssl-cert-snakeoil.key` | Location of the RSA private key. |
| `ssl_enable=NO` | Enable SSL? |

### The `/etc/ftpusers` File

This file **denies** specific users access to FTP, even if they exist as valid Linux accounts. In the example below, `guest`, `john`, and `kevin` cannot log in via FTP:

```shell
$ cat /etc/ftpusers
guest
john
kevin
```

> **Enumeration angle:** If you can read this file, you learn which accounts the admin considered sensitive enough to block — which is itself a hint about valuable usernames.

## Dangerous Settings

These options weaken the server and are worth hunting for. Anonymous write access in particular can lead directly to file upload and potential code execution.

| Setting | Description |
| --- | --- |
| `anonymous_enable=YES` | Allow anonymous login. |
| `anon_upload_enable=YES` | Allow anonymous users to upload files. |
| `anon_mkdir_write_enable=YES` | Allow anonymous users to create directories. |
| `no_anon_password=YES` | Don't ask anonymous users for a password. |
| `anon_root=/home/username/ftp` | Root directory for anonymous users. |
| `write_enable=YES` | Allow `STOR`, `DELE`, `RNFR`, `RNTO`, `MKD`, `RMD`, `APPE`, `SITE`. |

## Detailed Output Settings

| Setting | Description |
| --- | --- |
| `dirmessage_enable=YES` | Show a message when entering a new directory. |
| `chown_uploads=YES` | Change ownership of anonymously uploaded files. |
| `chown_username=username` | User given ownership of anonymously uploaded files. |
| `local_enable=YES` | Enable local users to log in. |
| `chroot_local_user=YES` | Place local users into their home directory. |
| `chroot_list_enable=YES` | Use a list of users to chroot into their home directory. |

| Setting | Description |
| --- | --- |
| `hide_ids=YES` | Display all user/group info in listings as `ftp`. |
| `ls_recurse_enable=YES` | Allow recursive listings. |

## Service Interaction

Connect manually to read the banner and interact with the service directly. The banner often reveals the server software, version, and sometimes the hostname or internal domain:

```shell
# Plain connection
nc -nv 10.129.14.136 21
nc 10.129.14.136 21

# If the server uses SSL/TLS
openssl s_client -connect 10.129.14.136:21 -starttls ftp
```

## Downloading Everything at Once

When anonymous access is allowed, mirror the entire share in one command rather than pulling files one by one:

```shell
wget -m --no-passive ftp://anonymous:anonymous@10.129.14.136
```

- `-m` enables mirror mode (recursive, preserves timestamps).
- `--no-passive` forces active mode, which sidesteps the common passive-mode data-channel failures you hit over VPNs or restrictive firewalls.

> **Practical tip:** If an interactive `get` hangs with a "No control connection" / "Not connected" error, the data channel is being blocked. Toggle `passive` off inside the `ftp>` prompt, or just switch to `wget`/`curl`, which negotiate the mode more gracefully.
