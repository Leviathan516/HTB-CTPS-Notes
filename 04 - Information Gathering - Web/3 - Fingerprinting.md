Fingerprinting Techniques

There are several techniques used for web server and technology fingerprinting:

    Banner Grabbing: Banner grabbing involves analysing the banners presented by web servers and other services. These banners often reveal the server software, version numbers, and other details.
    Analysing HTTP Headers: HTTP headers transmitted with every web page request and response contain a wealth of information. The Server header typically discloses the web server software, while the X-Powered-By header might reveal additional technologies like scripting languages or frameworks.
    Probing for Specific Responses: Sending specially crafted requests to the target can elicit unique responses that reveal specific technologies or versions. For example, certain error messages or behaviours are characteristic of particular web servers or software components.
    Analysing Page Content: A web page's content, including its structure, scripts, and other elements, can often provide clues about the underlying technologies. There may be a copyright header that indicates specific software being used, for example.

A variety of tools exist that automate the fingerprinting process, combining various techniques to identify web servers, operating systems, content management systems, and other technologies:
Tool	Description	Features
Wappalyzer	Browser extension and online service for website technology profiling.	Identifies a wide range of web technologies, including CMSs, frameworks, analytics tools, and more.
BuiltWith	Web technology profiler that provides detailed reports on a website's technology stack.	Offers both free and paid plans with varying levels of detail.
WhatWeb	Command-line tool for website fingerprinting.	Uses a vast database of signatures to identify various web technologies.
Nmap	Versatile network scanner that can be used for various reconnaissance tasks, including service and OS fingerprinting.	Can be used with scripts (NSE) to perform more specialised fingerprinting.
Netcraft	Offers a range of web security services, including website fingerprinting and security reporting.	Provides detailed reports on a website's technology, hosting provider, and security posture.
wafw00f	Command-line tool specifically designed for identifying Web Application Firewalls (WAFs).	Helps determine if a WAF is present and, if so, its type and configuration.
Fingerprinting inlanefreight.com

Let's apply our fingerprinting knowledge to uncover the digital DNA of our purpose-built host, inlanefreight.com. We'll leverage both manual and automated techniques to gather information about its web server, technologies, and potential vulnerabilities.
Banner Grabbing

Our first step is to gather information directly from the web server itself. We can do this using the curl command with the -I flag (or --head) to fetch only the HTTP headers, not the entire page content.

        shellsession
Leviathan516x@htb[/htb]$ curl -I inlanefreight.com

The output will include the server banner, revealing the web server software and version number:

        shellsession
Leviathan516x@htb[/htb]$ curl -I inlanefreight.com

HTTP/1.1 301 Moved Permanently
Date: Fri, 31 May 2024 12:07:44 GMT
Server: Apache/2.4.41 (Ubuntu)
Location: https://inlanefreight.com/
Content-Type: text/html; charset=iso-8859-1

In this case, we see that inlanefreight.com is running on Apache/2.4.41, specifically the Ubuntu version. This information is our first clue, hinting at the underlying technology stack. It's also trying to redirect to https://inlanefreight.com/ so grab those banners too

        shellsession
Leviathan516x@htb[/htb]$ curl -I https://inlanefreight.com

HTTP/1.1 301 Moved Permanently
Date: Fri, 31 May 2024 12:12:12 GMT
Server: Apache/2.4.41 (Ubuntu)
X-Redirect-By: WordPress
Location: https://www.inlanefreight.com/
Content-Type: text/html; charset=UTF-8

We now get a really interesting header, the server is trying to redirect us again, but this time we see that it's WordPress that is doing the redirection to https://www.inlanefreight.com/

        shellsession
Leviathan516x@htb[/htb]$ curl -I https://www.inlanefreight.com

HTTP/1.1 200 OK
Date: Fri, 31 May 2024 12:12:26 GMT
Server: Apache/2.4.41 (Ubuntu)
Link: <https://www.inlanefreight.com/index.php/wp-json/>; rel="https://api.w.org/"
Link: <https://www.inlanefreight.com/index.php/wp-json/wp/v2/pages/7>; rel="alternate"; type="application/json"
Link: <https://www.inlanefreight.com/>; rel=shortlink
Content-Type: text/html; charset=UTF-8

A few more interesting headers, including an interesting path that contains wp-json. The wp- prefix is common to WordPress.
Wafw00f

Web Application Firewalls (WAFs) are security solutions designed to protect web applications from various attacks. Before proceeding with further fingerprinting, it's crucial to determine if inlanefreight.com employs a WAF, as it could interfere with our probes or potentially block our requests.

To detect the presence of a WAF, we'll use the wafw00f tool. To install wafw00f, you can use pip3:

        shellsession
Leviathan516x@htb[/htb]$ pip3 install git+https://github.com/EnableSecurity/wafw00f

Once it's installed, pass the domain you want to check as an argument to the tool:

        shellsession
Leviathan516x@htb[/htb]$ wafw00f inlanefreight.com

                ______
               /      \
              (  W00f! )
               \  ____/
               ,,    __            404 Hack Not Found
           |`-.__   / /                      __     __
           /"  _/  /_/                       \ \   / /
          *===*    /                          \ \_/ /  405 Not Allowed
         /     )__//                           \   /
    /|  /     /---`                        403 Forbidden
    \\/`   \ |                                 / _ \
    `\    /_\\_              502 Bad Gateway  / / \ \  500 Internal Error
      `_____``-`                             /_/   \_\

                        ~ WAFW00F : v2.2.0 ~
        The Web Application Firewall Fingerprinting Toolkit
    
[*] Checking https://inlanefreight.com
[+] The site https://inlanefreight.com is behind Wordfence (Defiant) WAF.
[~] Number of requests: 2

The wafw00f scan on inlanefreight.com reveals that the website is protected by the Wordfence Web Application Firewall (WAF), developed by Defiant.

This means the site has an additional security layer that could block or filter our reconnaissance attempts. In a real-world scenario, it would be crucial to keep this in mind as you proceed with further investigation, as you might need to adapt techniques to bypass or evade the WAF's detection mechanisms.
Nikto

Nikto is a powerful open-source web server scanner. In addition to its primary function as a vulnerability assessment tool, Nikto's fingerprinting capabilities provide insights into a website's technology stack.

Nikto is pre-installed on pwnbox, but if you need to install it, you can run the following commands:

        shellsession
Leviathan516x@htb[/htb]$ sudo apt update && sudo apt install -y perl
Leviathan516x@htb[/htb]$ git clone https://github.com/sullo/nikto
Leviathan516x@htb[/htb]$ cd nikto/program
Leviathan516x@htb[/htb]$ chmod +x ./nikto.pl

To scan inlanefreight.com using Nikto, only running the fingerprinting modules, execute the following command:

        shellsession
Leviathan516x@htb[/htb]$ nikto -h inlanefreight.com -Tuning b

The -h flag specifies the target host. The -Tuning b flag tells Nikto to only run the Software Identification modules.

Nikto will then initiate a series of tests, attempting to identify outdated software, insecure files or configurations, and other potential security risks.

        shellsession
Leviathan516x@htb[/htb]$ nikto -h inlanefreight.com -Tuning b

- Nikto v2.5.0
---------------------------------------------------------------------------
+ Multiple IPs found: 134.209.24.248, 2a03:b0c0:1:e0::32c:b001
+ Target IP:          134.209.24.248
+ Target Hostname:    www.inlanefreight.com
+ Target Port:        443
---------------------------------------------------------------------------
+ SSL Info:        Subject:  /CN=inlanefreight.com
                   Altnames: inlanefreight.com, www.inlanefreight.com
                   Ciphers:  TLS_AES_256_GCM_SHA384
                   Issuer:   /C=US/O=Let's Encrypt/CN=R3
+ Start Time:         2024-05-31 13:35:54 (GMT0)
---------------------------------------------------------------------------
+ Server: Apache/2.4.41 (Ubuntu)
+ /: Link header found with value: ARRAY(0x558e78790248). See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Link
+ /: The site uses TLS and the Strict-Transport-Security HTTP header is not defined. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security
+ /: The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type. See: https://www.netsparker.com/web-vulnerability-scanner/vulnerabilities/missing-content-type-header/
+ /index.php?: Uncommon header 'x-redirect-by' found, with contents: WordPress.
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ /: The Content-Encoding header is set to "deflate" which may mean that the server is vulnerable to the BREACH attack. See: http://breachattack.com/
+ Apache/2.4.41 appears to be outdated (current is at least 2.4.59). Apache 2.2.34 is the EOL for the 2.x branch.
+ /: Web Server returns a valid response with junk HTTP methods which may cause false positives.
+ /license.txt: License file found may identify site software.
+ /: A Wordpress installation was found.
+ /wp-login.php?action=register: Cookie wordpress_test_cookie created without the httponly flag. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies
+ /wp-login.php:X-Frame-Options header is deprecated and has been replaced with the Content-Security-Policy HTTP header with the frame-ancestors directive instead. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options
+ /wp-login.php: Wordpress login found.
+ 1316 requests: 0 error(s) and 12 item(s) reported on remote host
+ End Time:           2024-05-31 13:47:27 (GMT0) (693 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested

The reconnaissance scan on inlanefreight.com reveals several key findings:

    IPs: The website resolves to both IPv4 (134.209.24.248) and IPv6 (2a03:b0c0:1:e0::32c:b001) addresses.
    Server Technology: The website runs on Apache/2.4.41 (Ubuntu)
    WordPress Presence: The scan identified a WordPress installation, including the login page (/wp-login.php). This suggests the site might be a potential target for common WordPress-related exploits.
    Information Disclosure: The presence of a license.txt file could reveal additional details about the website's software components.
    Headers: Several non-standard or insecure headers were found, including a missing Strict-Transport-Security header and a potentially insecure x-redirect-by header.



Target: 
10.129.42.195

Determine the Apache version running on app.inlanefreight.local on the target system. (Format: 0.0.0)
        2.4.41

─[leviathan@parrot]─[~]
└──╼ $curl -I 10.129.42.195
HTTP/1.1 200 OK
Date: Wed, 27 May 2026 23:12:13 GMT
Server: Apache/2.4.41 (Ubuntu)
Last-Modified: Mon, 16 Aug 2021 18:15:18 GMT
ETag: "b8-5c9b12f02857c"
Accept-Ranges: bytes
Content-Length: 184
Vary: Accept-Encoding
Content-Type: text/html

Which CMS is used on app.inlanefreight.local on the target system? Respond with the name only, e.g., WordPress.
        Joomla!

┌─[leviathan@parrot]─[~]
└──╼ $whatweb http://app.inlanefreight.local/
http://app.inlanefreight.local/ [200 OK] Apache[2.4.41], Bootstrap, Cookies[72af8f2b24261272e581a49f5c56de40], Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.41 (Ubuntu)], HttpOnly[72af8f2b24261272e581a49f5c56de40], IP[10.129.42.195], JQuery, MetaGenerator[Joomla! - Open Source Content Management], OpenSearch[http://app.inlanefreight.local/index.php/component/search/?layout=blog&amp;id=9&amp;Itemid=101&amp;format=opensearch], Script, Title[Home], UncommonHeaders[permissions-policy]
       

On which operating system is the dev.inlanefreight.local webserver running in the target system? Respond with the name only, e.g., Debian.
        Ubuntu

┌─[leviathan@parrot]─[~]
└──╼ $curl -s -I -H "Host: dev.inlanefreight.local" http://10.129.42.195/
HTTP/1.1 200 OK
Date: Wed, 27 May 2026 23:34:03 GMT
Server: Apache/2.4.41 (Ubuntu)
Set-Cookie: 02a93f6429c54209e06c64b77be2180d=sifeunnj4g1dvbeg3l106vr146; path=/; HttpOnly
Expires: Wed, 17 Aug 2005 00:00:00 GMT
Last-Modified: Wed, 27 May 2026 23:34:14 GMT
Cache-Control: no-store, no-cache, must-revalidate, post-check=0, pre-check=0
Pragma: no-cache
Content-Type: text/html; charset=utf-8



