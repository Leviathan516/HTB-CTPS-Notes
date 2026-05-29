
```ad-important
Make sure to check all subdomains and subs of subdomains (**e.g.** dev.web.example.com).

---

You can use `cat results.json | jq` to get a prettier view of the results

---

When you try to curl a directory `curl http://inlanefreight.htb/admin` and hit a 301, add the `/` in the end to get an overview of the content or what's the new file `curl http://inlanefreight.htb/admin/`
```

Target: 
154.57.164.81:32687

What is the IANA ID of the registrar of the inlanefreight.com domain?
        468

┌─[leviathan@parrot]─[~]
└──╼ $whois inlanefreight.com
Domain Name: INLANEFREIGHT.COM
   Registry Domain ID: 2420436757_DOMAIN_COM-VRSN
   Registrar WHOIS Server: whois.registrar.amazon
   Registrar URL: http://registrar.amazon.com
   Updated Date: 2026-05-13T08:57:09Z
   Creation Date: 2019-08-05T22:43:09Z
   Registry Expiry Date: 2026-08-05T22:43:09Z
   Registrar: Amazon Registrar, Inc.
   Registrar IANA ID: 468
   Registrar Abuse Contact Email: trustandsafety@support.aws.com
   Registrar Abuse Contact Phone: +1.2024422253
   Domain Status: clientTransferProhibited https://icann.org/epp#clientTransferProhibited
   Name Server: NS-1303.AWSDNS-34.ORG
   Name Server: NS-1580.AWSDNS-05.CO.UK
   Name Server: NS-161.AWSDNS-20.COM
   Name Server: NS-671.AWSDNS-19.NET
   DNSSEC: unsigned
...
Registrar IANA ID: 468


What http server software is powering the inlanefreight.htb site on the target system? Respond with the name of the software, not the version, e.g., Apache.
        nginx
        
┌─[✗]─[leviathan@parrot]─[~]
└──╼ $curl -I 154.57.164.74:31011
HTTP/1.1 200 OK
Server: nginx/1.26.1
Date: Fri, 29 May 2026 16:20:30 GMT
Content-Type: text/html
Content-Length: 120
Last-Modified: Thu, 01 Aug 2024 09:35:23 GMT
Connection: keep-alive
ETag: "66ab56db-78"
Accept-Ranges: bytes

What is the API key in the hidden admin directory that you have discovered on the target system?
        e963d863ee0e82ba7080fbf558ca0d3f
        
┌─[✗]─[leviathan@parrot]─[~]
└──╼ $gobuster vhost -u http://inlanefreight.htb:32687 -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt --append-domain
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:             http://inlanefreight.htb:32687
[+] Method:          GET
[+] Threads:         10
[+] Wordlist:        /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt
[+] User Agent:      gobuster/3.6
[+] Timeout:         10s
[+] Append Domain:   true
===============================================================
Starting gobuster in VHOST enumeration mode
===============================================================
Found: web1337.inlanefreight.htb:32687 Status: 200 [Size: 104]
Progress: 114442 / 114443 (100.00%)
===============================================================
Finished
===============================================================

Found: web1337.inlanefreight.htb:32687
        add the vhost to /etc/hosts
─[leviathan@parrot]─[~]
└──╼ $echo "154.57.164.81 inlanefreight.htb web1337.inlanefreight.htb" | sudo tee -a /etc/hosts
┌─[leviathan@parrot]─[~]
└──╼ $curl http://web1337.inlanefreight.htb:32687/robots.txt
User-agent: *
Allow: /index.html
Allow: /index-2.html
Allow: /index-3.html
Disallow: /admin_h1dd3n
        admin_h1dd3n found, curl info 
┌─[leviathan@parrot]─[~]
└──╼ $curl http://web1337.inlanefreight.htb:32687/admin_h1dd3n/
<!DOCTYPE html><html><head><title>web1337 admin</title></head><body><h1>Welcome to web1337 admin site</h1><h2>The admin panel is currently under maintenance, but the API is still accessible with the key e963d863ee0e82ba7080fbf558ca0d3f</h2></body></html>


After crawling the inlanefreight.htb domain on the target system, what is the email address you have found? Respond with the full email, e.g., mail@inlanefreight.htb.
        1337testing@inlanefreight.htb
        
    Create Enviorment for web crawling env

┌─[✗]─[leviathan@parrot]─[~]
└──╼ $python3 -m venv myenv
┌─[leviathan@parrot]─[~]
└──╼ $source myenv/bin/activate
(myenv) ┌─[leviathan@parrot]─[~]
└──╼ $pip3 install scrapy
        Add host to etc/hosts
     echo "154.57.164.81 dev.web1337.inlanefreight.htb" | sudo tee -a /etc/hosts

(myenv) ┌─[leviathan@parrot]─[~]
└──╼ $python3 ReconSpider.py http://dev.web1337.inlanefreight.htb:32687
... 
2026-05-29 11:40:46 [scrapy.core.engine] INFO: Spider closed (finished)
         cat results 
(myenv) ┌─[leviathan@parrot]─[~]
└──╼ $cat results.json 
{
    "emails": [
        "1337testing@inlanefreight.htb"
    ],
    "links": [
        "http://dev.web1337.inlanefreight.htb:32687/index-925.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-888.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-24.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-815.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-734.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-895.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-795.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-80.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-687.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-292.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-134.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-329.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-463.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-244.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-918.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-504.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-938.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-334.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-643.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-577.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-203.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-944.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-988.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-226.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-105.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-728.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-949.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-459.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-631.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-909.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-408.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-933.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-77.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-114.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-799.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-977.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-247.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-513.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-964.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-254.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-326.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-626.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-553.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-947.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-561.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-748.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-555.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-204.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-769.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-635.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-165.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-817.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-166.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-789.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-384.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-379.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-302.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-291.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-220.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-641.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-248.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-714.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-531.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-727.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-660.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-202.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-989.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-465.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-948.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-332.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-574.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-472.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-342.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-431.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-939.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-437.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-581.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-403.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-755.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-350.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-189.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-1000.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-785.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-385.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-458.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-585.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-567.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-615.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-224.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-733.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-300.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-737.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-798.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-364.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-760.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-335.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-862.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-807.html",
        "http://dev.web1337.inlanefreight.htb:32687/index-525.html"
    ],
    "external_files": [],
    "js_files": [],
    "form_fields": [],
    "images": [],
    "videos": [],
    "audio": [],
    "comments": [
        "<!-- Remember to change the API key to ba988b835be4aa97d068941dc852ff33 -->"
    ]


What is the API key the inlanefreight.htb developers will be changing too?
        ba988b835be4aa97d068941dc852ff33



