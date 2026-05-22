# NFS

The **Network File System (NFS)** serves the same purpose as SMB — sharing files across a network — but originates from the Unix world. Its defining weakness is its trust model: NFS has **no built-in authentication or authorization** mechanism of its own.

| Version | Features |
| --- | --- |
| NFSv2 | Older but widely supported; originally operated entirely over UDP. |
| NFSv3 | More features (variable file size, better error reporting); not fully compatible with NFSv2 clients. |
| NFSv4 | Adds Kerberos, works through firewalls and over the Internet, drops the portmapper requirement, supports ACLs, and is the first **stateful** version. |

## How NFS "Authentication" Works

Authentication is entirely delegated to the **RPC protocol**, and authorization is derived from filesystem information. The server translates the client's user info into filesystem permissions using **UNIX UID/GID and group memberships**.

> **The core flaw:** The client and server do not need to share the same UID/GID-to-user mappings, and the server performs no further checks. If you can present a UID that matches a file's owner, you get that owner's access. This is why NFS should only ever be used on trusted networks — and why it's so useful to a pentester.

## Default Configuration (`/etc/exports`)

The `/etc/exports` file is the access control list defining which directories are shared and to whom:

```shell
$ cat /etc/exports
# Example for NFSv2 and NFSv3:
# /srv/homes    hostname1(rw,sync,no_subtree_check) hostname2(ro,sync,no_subtree_check)
# Example for NFSv4:
# /srv/nfs4     gss/krb5i(rw,sync,fsid=0,crossmnt,no_subtree_check)
```

| Option | Description |
| --- | --- |
| `rw` | Read and write permissions. |
| `ro` | Read-only permissions. |
| `sync` | Synchronous data transfer (slower, safer). |
| `async` | Asynchronous data transfer (faster). |
| `secure` | Ports above 1024 will *not* be used. |
| `insecure` | Ports above 1024 *will* be used. |
| `no_subtree_check` | Disables checking of subdirectory trees. |
| `root_squash` | Maps root's UID/GID 0 to anonymous, preventing root from accessing files on the mount. |

Sharing a directory and reloading the service:

```shell
root@nfs:~# echo '/mnt/nfs  10.129.14.0/24(sync,no_subtree_check)' >> /etc/exports
root@nfs:~# systemctl restart nfs-kernel-server
root@nfs:~# exportfs
/mnt/nfs    10.129.14.0/24
```

## Dangerous Settings

| Option | Description |
| --- | --- |
| `rw` | Read and write permissions. |
| `insecure` | Allows ports above 1024 — lets unprivileged client processes connect. |
| `nohide` | Exports filesystems mounted beneath an exported directory. |
| `no_root_squash` | **Critical:** files created by root keep UID/GID 0, enabling privilege escalation via NFS. |

> **Why `no_root_squash` is dangerous:** With it enabled, you can create a root-owned SUID binary on the share from your attacking machine, then execute it on the target to gain root.

## Footprinting the Service

NFS relies on TCP ports **111** (rpcbind) and **2049** (nfs). The `nfs*` Nmap scripts pull a wealth of information — exported shares, file listings, permissions, and RPC program mappings:

```shell
$ sudo nmap --script nfs* 10.129.14.128 -sV -p111,2049

PORT     STATE SERVICE VERSION
111/tcp  open  rpcbind 2-4 (RPC #100000)
| nfs-ls: Volume /mnt/nfs
|   access: Read Lookup NoModify NoExtend NoDelete NoExecute
| PERMISSION  UID    GID    SIZE  FILENAME
| rw-r--r--   0      0      1872  id_rsa
| rw-r--r--   0      0      348   id_rsa.pub
| rw-r--r--   0      0      0     nfs.share
|_
| nfs-showmount:
|_  /mnt/nfs 10.129.14.0/24
| nfs-statfs:
|   Filesystem  1K-blocks   Used       Available   Use%
|_  /mnt/nfs    30313412.0  8074868.0  20675664.0  29%
```

### List Available Shares

```shell
$ showmount -e 10.129.14.128

Export list for 10.129.14.128:
/mnt/nfs 10.129.14.0/24
```

### Mounting a Share

Create a local mount point first, then mount. The `nolock` option disables file locking (avoids errors against some servers):

```shell
$ mkdir target-NFS
$ sudo mount -t nfs 10.129.14.128:/ ./target-NFS/ -o nolock
$ cd target-NFS
$ tree .
.
└── mnt
    └── nfs
        ├── id_rsa
        ├── id_rsa.pub
        └── nfs.share
```

When finished, always unmount cleanly:

```shell
$ cd ~
$ sudo umount ./target-NFS
```

> **Common gotchas:**
> - NFS paths are **case-sensitive** — `/TechSupport` ≠ `/Techsupport`.
> - The mount point directory must exist *before* you mount onto it.
> - A UID like `4294967294` in listings is the squashed `nobody` value (`-2` as unsigned). Files with `rwx------` owned by it may need `sudo` to read.
