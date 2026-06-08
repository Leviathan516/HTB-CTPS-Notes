# Laudanum

Laudanum, One Webshell to Rule Them All

Laudanum is a repository of ready-made files that can be used to inject onto a victim and receive back access via a reverse shell, run commands on the victim host right from the browser, and more. The repo includes injectable files for many different web application languages to include asp, aspx, jsp, php, and more. This is a staple to have on any pentest. If you are using your own VM, Laudanum is built into Parrot OS and Kali by default. For any other distro, you will likely need to pull a copy down to use. You can get it here. Let's examine Laudanum and see how it works.
Working with Laudanum

The Laudanum files can be found in the /usr/share/laudanum directory. For most of the files within Laudanum, you can copy them as-is and place them where you need them on the victim to run. For specific files such as the shells, you must edit the file first to insert your attacking host IP address to ensure you can access the web shell or receive a callback in the instance that you use a reverse shell. Before using the different files, be sure to read the contents and comments to ensure you take the proper actions.
Laudanum Demonstration

Now that we understand what Laudanum is and how it works, let's look at a web application we have found in our lab environment and see if we can run a web shell. If you wish to follow along with this demonstration, you will need to add an entry into your /etc/hosts file on your attack VM or within Pwnbox for the host we are attacking. That entry should read: <target ip> status.inlanefreight.local. Once this is done, you can play and explore this demonstration as long as you are on the VPN or using Pwnbox.
Move a Copy for Modification

        shellsession
Leviathan516x@htb[/htb]$ cp /usr/share/laudanum/aspx/shell.aspx /home/tester/demo.aspx

Add your IP address to the allowedIps variable on line 59. Make any other changes you wish. It can be prudent to remove the ASCII art and comments from the file. These items in a payload are often signatured on and can alert the defenders/AV to what you are doing.
Modify the Shell for Use

The image shows a code snippet in a text editor. It highlights an array of allowed IP addresses, including "10.10.14.12". A yellow arrow points to this line, indicating its significance.

We are taking advantage of the upload function at the bottom of the status page(Green Arrow) for this to work. Select your shell file and hit upload. If successful, it should print out the path to where the file was saved (Yellow Arrow). Use the upload function. Success prints out where the file went, navigate to it.
Take Advantage of the Upload Function

The image shows a server status page with BIOS, disk, and services information. Several services are marked as "Stopped" in red. A section for importing configuration files is highlighted with a yellow arrow pointing to the file path and a green arrow pointing to the "Upload File" button.

Once the upload is successful, you will need to navigate to your web shell to utilize its functions. The image below shows us how to do it. As seen from the last image, our shell was uploaded to the \\files\ directory, and the name was kept the same. This won't always be the case. You may run into some implementations that randomize filenames on upload that do not have a public files directory or any number of other potential safeguards. For now, we are lucky that's not the case. With this particular web application, our file went to status.inlanefreight.local\\files\demo.aspx and will require us to browse for the upload by using that \ in the path instead of the / like normal. Once you do this, your browser will clean it up in your URL window to appear as status.inlanefreight.local//files/demo.aspx.
Navigate to Our Shell

The image shows a Laundanum ASPX Shell interface with a command input field labeled "cmd /c" and a "Submit Query" button. A green arrow points to the URL "status.inlanefreight.local/files/demo.aspx" in the browser's address bar.

We can now utilize the Laudanum shell we uploaded to issue commands to the host. We can see in the example that the systeminfo command was run.
Shell Success

The image shows a Laundanum ASPX Shell interface displaying system information. It includes details like host name, OS version, manufacturer, system type, processor, memory, and network card information. The command "systeminfo" is executed, and the output is shown under "STDOUT" in the browser window.

Target:
10.129.42.197

Establish a web shell session with the target using the concepts covered in this section. Submit the full path of the directory you land in. (Format: c:\path\you\land\in)
        c:\windows\system32\inetsrv

┌─[✗]─[leviathan@parrot]─[~]
└──╼ $cp /usr/share/laudanum/aspx/shell.aspx ~/demo.aspx
        Make changes to line 59, add tun0 ip 
┌─[leviathan@parrot]─[~]
└──╼ $nano ~/demo.aspx
        Add host to etc/hosts
echo "<target_ip> status.inlanefreight.local" | sudo tee -a /etc/hosts

Upload demo.aspx via the upload function at status.inlanefreight.local
navigate to status.inlanefreight.local//files/demo.aspx

cmd /c : cd
STDOUT:

c:\windows\system32\inetsrv



Where is the Laudanum aspx web shell located on Pwnbox? Submit the full path. (Format: /path/to/laudanum/aspx)
        /usr/share/laudanum/aspx/shell.aspx

┌─[✗]─[leviathan@parrot]─[~]
└──╼ $cd /usr/share/laudanum/aspx
┌─[leviathan@parrot]─[/usr/share/laudanum/aspx]
└──╼ $ls
shell.aspx


# Antak

ASPX Explained

Active Server Page Extended (ASPX) is a file type/extension written for Microsoft's ASP.NET Framework. On a web server running the ASP.NET framework, web form pages can be generated for users to input data. On the server side, the information will be converted into HTML. We can take advantage of this by using an ASPX-based web shell to control the underlying Windows operating system. Let's witness this first-hand by utilizing the Antak Webshell.
Antak Webshell

Antak is a web shell built in ASP.Net included within the Nishang project. Nishang is an Offensive PowerShell toolset that can provide options for any portion of your pentest. Since we are focused on web applications for the moment, let's keep our eyes on Antak. Antak utilizes PowerShell to interact with the host, making it great for acquiring a web shell on a Windows server. The UI is even themed like PowerShell. It's time to dive in and experiment with Antak.
Working with Antak

The Antak files can be found in the /usr/share/nishang/Antak-WebShell directory.

        shellsession
Leviathan516x@htb[/htb]$ ls /usr/share/nishang/Antak-WebShell

antak.aspx  Readme.md

Antak web shell functions like a Powershell Console. However, it will execute each command as a new process. It can also execute scripts in memory and encode commands you send. As a web shell, Antak is a pretty powerful tool.
Antak Demonstration

Now that we understand what Antak is and how it works let's put it to the test against the same web application from the Laudanum section. If you wish to follow along with this demonstration, you will need to add an entry into your /etc/hosts file on your attack VM or within Pwnbox for the host we are attacking. That entry should read: <target ip> status.inlanefreight.local. Once this is done, as long as you are on the VPN or using Pwnbox, you can also play and explore this demonstration.
Move a Copy for Modification

        shellsession
Leviathan516x@htb[/htb]$ cp /usr/share/nishang/Antak-WebShell/antak.aspx /home/administrator/Upload.aspx

Make sure you set credentials for access to the web shell. Modify line 14, adding a user (green arrow) and password (orange arrow). This comes into play when you browse to your web shell, much like Laudanum. This can help make your operations more secure by ensuring random people can't just stumble into using the shell. It can be prudent to remove the ASCII art and comments from the file. These items in a payload are often signatured on and can alert the defenders/AV to what you are doing.
Modify the Shell for Use

The image shows a code snippet with a conditional statement checking if the username is "Disclaimer" and the password is "ForLegitUseOnly". If true, it sets execution visibility and enables it. Green and orange arrows highlight the username and password conditions.

For the sake of demonstrating the tool, we are uploading it to the same status portal we used for Laudanum. That host was a Windows host, so our shell should work just fine with PowerShell. Upload the file and then navigate to the page for use. It will give you a user and password prompt. Remember, with this web application, the files are stored in the \\files\ directory. When you navigate to the upload.aspx file, you should see a prompt as we have below.
Shell Success

The image shows a login form for the Antak Webshell with fields for username and password, both filled with "htb-student". A "Login" button is present below the fields. The URL in the browser's address bar is "status.inlanefreight.local/files/upload.aspx".

As seen in the following image, we will be granted access if our credentials are entered properly.

The image shows the Antak Webshell interface with a blue command prompt area displaying the message: "Welcome to Antak - A Webshell which utilizes PowerShell. Use help for more details. Use clear to clear the screen." Below are buttons labeled "Submit," "Browse," "Upload the File," "Encode and Execute," "Download," "Parse web.config," "Execute SQL Query," and a field for entering a connection string.

Now that we have access, we can utilize PowerShell commands to navigate and take actions against the host. We can issue basic commands from the Antak shell window, upload and download files, encode and execute scripts, and much more (green arrow below). This is an excellent way to utilize a Webshell to deliver us a callback to our command and control platform. We could upload the payload via the Upload function or use a PowerShell one-liner to download and execute the shell for us. If you feel unsure where to start, issue the command help in the prompt window (orange arrow ) below.
Issuing Commands

The image shows a command prompt window listing files and directories in two sections. The first section displays logs in a directory, and the second section lists user directories under "C:\Users". Below the command output, there are buttons labeled "Submit," "Browse," "Upload the File," "Encode and Execute," and "Download." An orange arrow points to the "Submit" button, and a green arrow points to the "Encode and Execute" button.



Where is the Antak webshell located on Pwnbox? Submit the full path. (Format:/path/to/antakwebshell)
        /usr/share/nishang/Antak-WebShell/antak.aspx


Establish a web shell with the target using the concepts covered in this section. Submit the name of the user on the target that the commands are being issued as. In order to get the correct answer you must navigate to the web shell you upload using the vHost name. (Format: ****\****, 1 space)
        iis apppool\status
        
cp /usr/share/nishang/Antak-WebShell/antak.aspx ~/Upload.aspx
    edit file with credentials line 14 
nano ~/Upload.aspx 

upload via status.inlanefreight.local
go to upload location, status.inlanefreight.local/files/Upload.aspx
enter credentials 

type whoami 

PS> whoami
iis apppool\status


# PHP Web Shells

Hypertext Preprocessor or PHP is an open-source general-purpose scripting language typically used as part of a web stack that powers a website. At the time of this writing (October 2021), PHP is the most popular server-side programming language. According to a recent survey conducted by W3Techs, "PHP is used by 78.6% of all websites whose server-side programming language we know".

Let's consider a practical example of filling out the user account and password fields on a login web form.
PHP Login Page

rConfig login page with fields for username and password, 'Remember me' checkbox, and 'Forgot my password' link.

Recall the rConfig server from earlier in this module? It uses PHP. We can see a login.php file. So when we select the login button after filling out the Username and Password field, that information is processed server-side using PHP. Knowing that a web server is using PHP gives us pentesters a clue that we may gain a PHP-based web shell on this system. Let's work through this concept hands-on.
Hands-on With a PHP-Based Web Shell.

Since PHP processes code & commands on the server-side, we can use pre-written payloads to gain a shell through the browser or initiate a reverse shell session with our attack box. In this case, we will take advantage of the vulnerability in rConfig 3.9.6 to manually upload a PHP web shell and interact with the underlying Linux host. In addition to all the functionality mentioned earlier, rConfig allows admins to add network devices and categorize them by vendor. Go ahead and log in to rConfig with the default credentials (admin:admin), then navigate to Devices > Vendors and click Add Vendor.
Vendors Tab

rConfig vendor management page with options to add, edit, or remove vendors, fields for vendor name and logo upload, and save or close buttons.

We will be using WhiteWinterWolf's PHP Web Shell. We can download this or copy and paste the source code into a .php file. Keep in mind that the file type is significant, as we will soon witness. Our goal is to upload the PHP web shell via the Vendor Logo browse button. Attempting to do this initially will fail since rConfig is checking for the file type. It will only allow uploading image file types (.png,.jpg,.gif, etc.). However, we can bypass this utilizing Burp Suite.

Start Burp Suite, navigate to the browser's network settings menu and fill out the proxy settings. 127.0.0.1 will go in the IP address field, and 8080 will go in the port field to ensure all requests pass through Burp (recall that Burp acts as the web proxy).
Proxy Settings

Proxy settings dialog with options for no proxy, auto-detect, system settings, and manual configuration with HTTP proxy set to 127.0.0.1:8080.

Our goal is to change the content-type to bypass the file type restriction in uploading files to be "presented" as the vendor logo so we can navigate to that file and have our web shell.

Note: Firefox removed FTP support starting with version 90.
Bypassing the File Type Restriction

With Burp open and our web browser proxy settings properly configured, we can now upload the PHP web shell. Click the browse button, navigate to wherever our .php file is stored on our attack box, and select open and Save (we may need to accept the PortSwigger Certificate). It will seem as if the web page is hanging, but that's just because we need to tell Burp to forward the HTTP requests. Forward requests until you see the POST request containing our file upload. It will look like this:
Post Request

Burp Suite showing intercepted HTTP request with headers and PHP code snippet.

As mentioned in an earlier section, you will notice that some payloads have comments from the author that explain usage, provide kudos and links to personal blogs. This can give us away, so it's not always best to leave the comments in place. We will change Content-type from application/x-php to image/gif. This will essentially "trick" the server and allow us to upload the .php file, bypassing the file type restriction. Once we do this, we can select Forward twice, and the file will be submitted. We can turn the Burp interceptor off now and go back to the browser to see the results.
Vendor Added

Vendor management page showing added vendor 'NetVen' with options to add, edit, or remove vendors, and a table listing 'Cisco' and 'NetVen'.

The message: Added new vendor NetVen to Database lets us know our file upload was successful. We can also see the NetVen vendor entry with the logo showcasing a ripped piece of paper. This means rConfig did not recognize the file type as an image, so it defaulted to that image. We can now attempt to use our web shell. Using the browser, navigate to this directory on the rConfig server:

/images/vendor/connect.php

This executes the payload and provides us with a non-interactive shell session entirely in the browser, allowing us to execute commands on the underlying OS.
Webshell Success

Web interface for fetching files with fields for host, port, path, command execution, and sudo output showing allowed commands.
Considerations when Dealing with Web Shells

When utilizing web shells, consider the below potential issues that may arise during your penetration testing process:

    Web applications sometimes automatically delete files after a pre-defined period
    Limited interactivity with the operating system in terms of navigating the file system, downloading and uploading files, chaining commands together may not work (ex. whoami && hostname), slowing progress, especially when performing enumeration -Potential instability through a non-interactive web shell
    Greater chance of leaving behind proof that we were successful in our attack

Depending on the engagement type (i.e., a black box evasive assessment), we may need to attempt to go undetected and cover our tracks. We are often helping our clients test their capabilities to detect a live threat, so we should emulate as much as possible the methods a malicious attacker may attempt, including attempting to operate stealthily. This will help our client and save us in the long run from having files discovered after an engagement period is over. In most cases, when attempting to gain a shell session with a target, it would be wise to establish a reverse shell and then delete the executed payload. Also, we must document every method we attempt, what worked & what did not work, and even the names of the payloads & files we tried to use. We could include a sha1sum or MD5 hash of the file name, upload locations in our reports as proof, and provide attribution.

Now let's test our understanding with some challenge questions.


Target:
10.129.17.218

In the example shown, what must the Content-Type be changed to in order to successfully upload the web shell? (Format: .../... )
        image/gif



Use what you learned from the module to gain a web shell. What is the file name of the gif in the /images/vendor directory on the target? (Format: xxxx.gif)
        ajax-loader.gif

---

**Step 1: Download the PHP web shell**
```bash
cd ~
wget https://raw.githubusercontent.com/WhiteWinterWolf/wwwolf-php-webshell/master/webshell.php
mv webshell.php connect.php
```

---

**Step 2: Open Burp Suite**
```bash
burpsuite &
```
Wait for it to load, click through the prompts, go to **Proxy** tab → make sure **Intercept is Off**

---

**Step 3: Set Firefox proxy** (like your Image 4)
- Firefox → Settings → search "proxy" → Connection Settings
- Select **Manual proxy configuration**
- HTTP Proxy: `127.0.0.1` Port: `8080`
- Check **"Also use this proxy for FTP and HTTPS"**
- Click OK

---

**Step 4: Log into rConfig**

Navigate to:
```
http://10.129.201.101/
```
Login with `admin` / `admin`

---

**Step 5: Go to Devices > Vendors > Add Vendor**
- Enter any vendor name (ex: `TestVen`)
- Click Browse and select `~/connect.php`
- Right before clicking Save/Upload, turn Intercept ON
- Click Save — Burp catches it
- Modify Content-Type: application/x-php → Content-Type: image/gif
- Click Forward
- Turn Intercept OFF again
Sucessful Upload

---

**Step 7: Navigate to your shell**
```
https://10.129.201.101/images/vendor/connect.php

ls : execute 
ls
ajax-loader.gif
cisco.jpg
connect.php
juniper.jpg

```



