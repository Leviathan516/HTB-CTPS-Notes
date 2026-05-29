# Crawling
## Extracting Valuable Information

Crawlers can extract a diverse array of data, each serving a specific purpose in the reconnaissance process:

- **Links (Internal or External)**
- **Comments:** Comments sections on blogs, forums, or other interactive pages can contain useful information. Users often inadvertently reveal sensitive details, internal processes, or hints of vulnerabilities in their comments.
- **Metadata:** Refers to "data about data". In context of web pages, it include information like titles, descriptions, keywords, author names, and dates. This metadata can provide valuable context about a page's content, purpose, and relevance to your reconnaissance goals.
- **Sensitive Files:** Web crawlers can be configured to actively search for files that might be exposed on a website. This include` backup files` (e.g. `.bak`, `.old`), `configuration files` (e.g., `web.config`, `settings.php`), `log files` (e.g., `error_log`, `access_log`), and other files containing `passwords`, `API Keys`. Examining extracted files (especially logs and backups) can reveal important data like `database credentials`, `encryption keys`, or even source code snippets.

```ad-note
Make sure to group the data and look for patterns, not just look at individual links. Doing this can help discovering a hidden chain of critical findings.

**e.g.** Seeing a comment mentioning a file server, then while looking at crawl rsults we see multiple links related to a **/files/** directory
```

# robots.txt

## What is robots.txt?

This file contains instructions in the form of "directives" that tell bots which parts of the website they can and cannot crawl.
### How robots.txt Works

The directives in robots.txt typically target specific user-agents, which are identifiers for different types of bots. For example, a directive might look like this:

```txt
User-agent: *
Disallow: /private/
```

# Well-Known URIs

The `.well-knows` typically is accessed via `/.well-known/` path on the web server, it centralizes a website's critical metadata, including config files and information related to its services, protocols, and security mechanisms. 

By establishing a consistent location for such data, `.well-known` simplifies the discovery and access process for various stakeholders, including web browsers, applications, and security tools. 

For instance, to access a website's security policy, a client would request `https://example.com/.well-known/security.txt`

The `Internet Assigned Numbers Authority` (`IANA`) maintains a [registry](https://www.iana.org/assignments/well-known-uris/well-known-uris.xhtml) of `.well-known` URIs, each serving a specific purpose defined by various specifications and standards. Below is a table highlighting a few notable examples:

|URI Suffix|Description|Status|Reference|
|---|---|---|---|
|`security.txt`|Contains contact information for security researchers to report vulnerabilities.|Permanent|RFC 9116|
|`/.well-known/change-password`|Provides a standard URL for directing users to a password change page.|Provisional|[https://w3c.github.io/webappsec-change-password-url/#the-change-password-well-known-uri](https://w3c.github.io/webappsec-change-password-url/#the-change-password-well-known-uri)|
|`openid-configuration`|Defines configuration details for OpenID Connect, an identity layer on top of the OAuth 2.0 protocol.|Permanent|[http://openid.net/specs/openid-connect-discovery-1_0.html](http://openid.net/specs/openid-connect-discovery-1_0.html)|
|`assetlinks.json`|Used for verifying ownership of digital assets (e.g., apps) associated with a domain.|Permanent|[https://github.com/google/digitalassetlinks/blob/master/well-known/specification.md](https://github.com/google/digitalassetlinks/blob/master/well-known/specification.md)|
|`mta-sts.txt`|Specifies the policy for SMTP MTA Strict Transport Security (MTA-STS) to enhance email security.|Permanent|RFC 8461|

## Web Recon and .well-known

One particularly useful URI is `openid-configuration`.  When a client application wants to use OpenID Connect for authentication, it can retrieve the OpenID Connect Provider's configuration by accessing the `https://example.com/.well-known/openid-configuration` endpoint. This endpoint returns a JSON document containing metadata about the provider's endpoints, supported authentication methods, token issuance, and more:

```json
{ "issuer": "https://example.com", "authorization_endpoint": "https://example.com/oauth2/authorize", "token_endpoint": "https://example.com/oauth2/token", "userinfo_endpoint": "https://example.com/oauth2/userinfo", "jwks_uri": "https://example.com/oauth2/jwks", "response_types_supported": ["code", "token", "id_token"], "subject_types_supported": ["public"], "id_token_signing_alg_values_supported": ["RS256"], "scopes_supported": ["openid", "profile", "email"] }
```

The information obtained from the `openid-configuration` endpoint provides multiple exploration opportunities:

1. **Endpoint Discovery:** 
	- **Authorization Endpoint:** Identify URL for user authentication requests.
	- **Token Endpoint:** Finding the URL where tokens are issues.
	- **Userinfo Endpoint:** Locating the endpoint that provides user information
2. **JWKS URI**
3. **Supported Scopes and Responses Types**
4. **Algorithm Details**

After spidering inlanefreight.com, identify the location where future reports will be stored. Respond with the full domain, e.g., files.inlanefreight.com.
        inlanefreight-comp133.s3.amazonaws.htb
        
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
2026-05-29 11:36:26 [scrapy.utils.log] INFO: Scrapy 2.16.0 started (bot: scrapybot)
2026-05-29 11:36:26 [scrapy.utils.log] INFO: Versions:
{'lxml': '6.1.1',
 'libxml2': '2.14.6',
 'cssselect': '1.4.0',
 'parsel': '1.11.0',
 'w3lib': '2.4.1',
 'Twisted': '26.4.0',
 'Python': '3.13.5 (main, May  5 2026, 21:05:52) [GCC 14.2.0]',
 'pyOpenSSL': '26.2.0 (OpenSSL 4.0.0 14 Apr 2026)',
 'cryptography': '48.0.0',
 'Platform': 'Linux-7.0.7+parrot7-amd64-x86_64-with-glibc2.41'}
2026-05-29 11:36:26 [scrapy.addons] INFO: Enabled addons:
[]
2026-05-29 11:36:26 [scrapy.extensions.telnet] INFO: Telnet Password: 1eb0545d55d478c0
2026-05-29 11:36:26 [scrapy.middleware] INFO: Enabled extensions:
['scrapy.extensions.corestats.CoreStats',
 'scrapy.extensions.logcount.LogCount',
 'scrapy.extensions.telnet.TelnetConsole',
 'scrapy.extensions.memusage.MemoryUsage',
 'scrapy.extensions.logstats.LogStats']
2026-05-29 11:36:26 [scrapy.crawler] INFO: Overridden settings:
{'LOG_LEVEL': 'INFO'}
2026-05-29 11:36:26 [scrapy.middleware] INFO: Enabled downloader middlewares:
['scrapy.downloadermiddlewares.offsite.OffsiteMiddleware',
 'scrapy.downloadermiddlewares.httpauth.HttpAuthMiddleware',
 'scrapy.downloadermiddlewares.downloadtimeout.DownloadTimeoutMiddleware',
 'scrapy.downloadermiddlewares.defaultheaders.DefaultHeadersMiddleware',
 'scrapy.downloadermiddlewares.useragent.UserAgentMiddleware',
 '__main__.CustomOffsiteMiddleware',
 'scrapy.downloadermiddlewares.retry.RetryMiddleware',
 'scrapy.downloadermiddlewares.redirect.MetaRefreshMiddleware',
 'scrapy.downloadermiddlewares.httpcompression.HttpCompressionMiddleware',
 'scrapy.downloadermiddlewares.redirect.RedirectMiddleware',
 'scrapy.downloadermiddlewares.cookies.CookiesMiddleware',
 'scrapy.downloadermiddlewares.httpproxy.HttpProxyMiddleware',
 'scrapy.downloadermiddlewares.stats.DownloaderStats']
2026-05-29 11:36:26 [scrapy.middleware] INFO: Enabled spider middlewares:
['scrapy.spidermiddlewares.start.StartSpiderMiddleware',
 'scrapy.spidermiddlewares.httperror.HttpErrorMiddleware',
 'scrapy.spidermiddlewares.referer.RefererMiddleware',
 'scrapy.spidermiddlewares.urllength.UrlLengthMiddleware',
 'scrapy.spidermiddlewares.depth.DepthMiddleware']
2026-05-29 11:36:26 [scrapy.middleware] INFO: Enabled item pipelines:
[]
2026-05-29 11:36:26 [scrapy.core.engine] INFO: Spider opened
2026-05-29 11:36:26 [scrapy.extensions.logstats] INFO: Crawled 0 pages (at 0 pages/min), scraped 0 items (at 0 items/min)
2026-05-29 11:36:26 [scrapy.extensions.telnet] INFO: Telnet console listening on 127.0.0.1:6023
2026-05-29 11:36:27 [scrapy.downloadermiddlewares.retry] ERROR: Gave up retrying <GET http://dev.web1337.inlanefreight.htb:32687> (failed 3 times): DNS lookup failed: no results for hostname lookup: dev.web1337.inlanefreight.htb.
2026-05-29 11:36:27 [scrapy.core.scraper] ERROR: Error downloading <GET http://dev.web1337.inlanefreight.htb:32687>
Traceback (most recent call last):
  File "/home/leviathan/myenv/lib/python3.13/site-packages/scrapy/utils/_download_handlers.py", line 55, in wrap_twisted_exceptions
    yield
  File "/home/leviathan/myenv/lib/python3.13/site-packages/scrapy/core/downloader/handlers/http11.py", line 131, in download_request
    return await maybe_deferred_to_future(agent.download_request(request))
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/home/leviathan/myenv/lib/python3.13/site-packages/twisted/internet/defer.py", line 1082, in _runCallbacks
    current.result = callback(  # type: ignore[misc]
                     ~~~~~~~~^^^^^^^^^^^^^^^^^^^^^^^
        current.result, *args, **kwargs
        ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    )
    ^
  File "/home/leviathan/myenv/lib/python3.13/site-packages/twisted/internet/endpoints.py", line 1135, in startConnectionAttempts
    raise error.DNSLookupError(
        f"no results for hostname lookup: {self._hostText}"
    )
twisted.internet.error.DNSLookupError: DNS lookup failed: no results for hostname lookup: dev.web1337.inlanefreight.htb.

The above exception was the direct cause of the following exception:

Traceback (most recent call last):
  File "/home/leviathan/myenv/lib/python3.13/site-packages/twisted/internet/defer.py", line 1834, in _inlineCallbacks
    result = context.run(
        cast(Failure, result).throwExceptionIntoGenerator, gen
    )
  File "/home/leviathan/myenv/lib/python3.13/site-packages/twisted/python/failure.py", line 467, in throwExceptionIntoGenerator
    return g.throw(self.value.with_traceback(self.tb))
           ~~~~~~~^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/home/leviathan/myenv/lib/python3.13/site-packages/scrapy/core/engine.py", line 497, in _download
    result = yield self.downloader.fetch(request)
             ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/home/leviathan/myenv/lib/python3.13/site-packages/twisted/internet/defer.py", line 1834, in _inlineCallbacks
    result = context.run(
        cast(Failure, result).throwExceptionIntoGenerator, gen
    )
  File "/home/leviathan/myenv/lib/python3.13/site-packages/twisted/python/failure.py", line 467, in throwExceptionIntoGenerator
    return g.throw(self.value.with_traceback(self.tb))
           ~~~~~~~^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/home/leviathan/myenv/lib/python3.13/site-packages/scrapy/core/downloader/__init__.py", line 131, in fetch
    result: Response | Request = yield (
                                 ^^^^^^^
    ...<3 lines>...
    )
    ^
  File "/home/leviathan/myenv/lib/python3.13/site-packages/twisted/internet/defer.py", line 1247, in adapt
    extracted: _SelfResultT | Failure = result.result()
                                        ~~~~~~~~~~~~~^^
  File "/home/leviathan/myenv/lib/python3.13/site-packages/scrapy/core/downloader/middleware.py", line 87, in download_async
    result = await self._process_exception(ex, request)
             ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/home/leviathan/myenv/lib/python3.13/site-packages/scrapy/core/downloader/middleware.py", line 156, in _process_exception
    raise exception
  File "/home/leviathan/myenv/lib/python3.13/site-packages/scrapy/core/downloader/middleware.py", line 80, in download_async
    result: Response | Request = await self._process_request(
                                 ^^^^^^^^^^^^^^^^^^^^^^^^^^^^
        request, download_func
        ^^^^^^^^^^^^^^^^^^^^^^
    )
    ^
  File "/home/leviathan/myenv/lib/python3.13/site-packages/scrapy/core/downloader/middleware.py", line 114, in _process_request
    return await download_func(request)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/home/leviathan/myenv/lib/python3.13/site-packages/scrapy/core/downloader/__init__.py", line 191, in _enqueue_request
    return await maybe_deferred_to_future(d)  # fired in _wait_for_download()
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/home/leviathan/myenv/lib/python3.13/site-packages/scrapy/core/downloader/__init__.py", line 258, in _wait_for_download
    response = await self._download(slot, request)
               ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/home/leviathan/myenv/lib/python3.13/site-packages/scrapy/core/downloader/__init__.py", line 228, in _download
    response: Response = await self.handlers.download_request_async(request)
                         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/home/leviathan/myenv/lib/python3.13/site-packages/scrapy/core/downloader/handlers/__init__.py", line 156, in download_request_async
    return await handler.download_request(request)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/home/leviathan/myenv/lib/python3.13/site-packages/scrapy/core/downloader/handlers/http11.py", line 130, in download_request
    with wrap_twisted_exceptions():
         ~~~~~~~~~~~~~~~~~~~~~~~^^
  File "/usr/lib/python3.13/contextlib.py", line 162, in __exit__
    self.gen.throw(value)
    ~~~~~~~~~~~~~~^^^^^^^
  File "/home/leviathan/myenv/lib/python3.13/site-packages/scrapy/utils/_download_handlers.py", line 63, in wrap_twisted_exceptions
    raise CannotResolveHostError(str(e)) from e
scrapy.exceptions.CannotResolveHostError: DNS lookup failed: no results for hostname lookup: dev.web1337.inlanefreight.htb.
2026-05-29 11:36:27 [scrapy.core.engine] INFO: Closing spider (finished)
2026-05-29 11:36:27 [scrapy.statscollectors] INFO: Dumping Scrapy stats:
{'downloader/exception_count': 3,
 'downloader/exception_type_count/scrapy.exceptions.CannotResolveHostError': 3,
 'downloader/request_bytes': 690,
 'downloader/request_count': 3,
 'downloader/request_method_count/GET': 3,
 'elapsed_time_seconds': 0.7865658289993007,
 'finish_reason': 'finished',
 'finish_time': datetime.datetime(2026, 5, 29, 18, 36, 27, 682991, tzinfo=datetime.timezone.utc),
 'items_per_minute': None,
 'log_count/ERROR': 2,
 'log_count/INFO': 3,
 'memusage/max': 72175616,
 'memusage/startup': 72175616,
 'responses_per_minute': None,
 'retry/count': 2,
 'retry/max_reached': 1,
 'retry/reason_count/scrapy.exceptions.CannotResolveHostError': 2,
 'scheduler/dequeued': 3,
 'scheduler/dequeued/memory': 3,
 'scheduler/enqueued': 3,
 'scheduler/enqueued/memory': 3,
 'start_time': datetime.datetime(2026, 5, 29, 18, 36, 26, 896424, tzinfo=datetime.timezone.utc)}
2026-05-29 11:36:27 [scrapy.core.engine] INFO: Spider closed (finished)
(myenv) ┌─[leviathan@parrot]─[~]
└──╼ $


─[leviathan@parrot]─[~]
└──╼ $python3 ReconSpider.py http://inlanefreight.com
2026-05-28 15:28:11 [scrapy.utils.log] INFO: Scrapy 2.16.0 started (bot: scrapybot)
2026-05-28 15:28:11 [scrapy.utils.log] INFO: Versions:
{'lxml': '6.1.1',
 'libxml2': '2.14.6',
 'cssselect': '1.4.0',
 'parsel': '1.11.0',
 'w3lib': '2.4.1',
 'Twisted': '26.4.0',
 'Python': '3.13.5 (main, May  5 2026, 21:05:52) [GCC 14.2.0]',
 'pyOpenSSL': '26.2.0 (OpenSSL 4.0.0 14 Apr 2026)',
 'cryptography': '48.0.0',
 'Platform': 'Linux-7.0.7+parrot7-amd64-x86_64-with-glibc2.41'}
2026-05-28 15:28:11 [scrapy.addons] INFO: Enabled addons:
[]
2026-05-28 15:28:11 [scrapy.extensions.telnet] INFO: Telnet Password: 72de605028f07c45
2026-05-28 15:28:11 [scrapy.middleware] INFO: Enabled extensions:
['scrapy.extensions.corestats.CoreStats',
 'scrapy.extensions.logcount.LogCount',
 'scrapy.extensions.telnet.TelnetConsole',
 'scrapy.extensions.memusage.MemoryUsage',
 'scrapy.extensions.logstats.LogStats']
2026-05-28 15:28:11 [scrapy.crawler] INFO: Overridden settings:
{'LOG_LEVEL': 'INFO'}
2026-05-28 15:28:11 [scrapy.middleware] INFO: Enabled downloader middlewares:
['scrapy.downloadermiddlewares.offsite.OffsiteMiddleware',
 'scrapy.downloadermiddlewares.httpauth.HttpAuthMiddleware',
 'scrapy.downloadermiddlewares.downloadtimeout.DownloadTimeoutMiddleware',
 'scrapy.downloadermiddlewares.defaultheaders.DefaultHeadersMiddleware',
 'scrapy.downloadermiddlewares.useragent.UserAgentMiddleware',
 '__main__.CustomOffsiteMiddleware',
 'scrapy.downloadermiddlewares.retry.RetryMiddleware',
 'scrapy.downloadermiddlewares.redirect.MetaRefreshMiddleware',
 'scrapy.downloadermiddlewares.httpcompression.HttpCompressionMiddleware',
 'scrapy.downloadermiddlewares.redirect.RedirectMiddleware',
 'scrapy.downloadermiddlewares.cookies.CookiesMiddleware',
 'scrapy.downloadermiddlewares.httpproxy.HttpProxyMiddleware',
 'scrapy.downloadermiddlewares.stats.DownloaderStats']
2026-05-28 15:28:11 [scrapy.middleware] INFO: Enabled spider middlewares:
['scrapy.spidermiddlewares.start.StartSpiderMiddleware',
 'scrapy.spidermiddlewares.httperror.HttpErrorMiddleware',
 'scrapy.spidermiddlewares.referer.RefererMiddleware',
 'scrapy.spidermiddlewares.urllength.UrlLengthMiddleware',
 'scrapy.spidermiddlewares.depth.DepthMiddleware']
2026-05-28 15:28:11 [scrapy.middleware] INFO: Enabled item pipelines:
[]
2026-05-28 15:28:11 [scrapy.core.engine] INFO: Spider opened
2026-05-28 15:28:11 [scrapy.extensions.logstats] INFO: Crawled 0 pages (at 0 pages/min), scraped 0 items (at 0 items/min)
2026-05-28 15:28:11 [scrapy.extensions.telnet] INFO: Telnet console listening on 127.0.0.1:6023
^H2026-05-28 15:28:15 [scrapy.core.engine] INFO: Closing spider (finished)
2026-05-28 15:28:15 [scrapy.statscollectors] INFO: Dumping Scrapy stats:
{'downloader/request_bytes': 2709,
 'downloader/request_count': 10,
 'downloader/request_method_count/GET': 10,
 'downloader/response_bytes': 93960,
 'downloader/response_count': 10,
 'downloader/response_status_count/200': 8,
 'downloader/response_status_count/301': 2,
 'dupefilter/filtered': 64,
 'elapsed_time_seconds': 3.81463662200008,
 'finish_reason': 'finished',
 'finish_time': datetime.datetime(2026, 5, 28, 22, 28, 15, 17938, tzinfo=datetime.timezone.utc),
 'httpcompression/response_bytes': 162654,
 'httpcompression/response_count': 7,
 'items_per_minute': 0.0,
 'log_count/INFO': 3,
 'memusage/max': 72122368,
 'memusage/startup': 72122368,
 'request_depth_max': 2,
 'response_received_count': 8,
 'responses_per_minute': 160.0,
 'scheduler/dequeued': 10,
 'scheduler/dequeued/memory': 10,
 'scheduler/enqueued': 10,
 'scheduler/enqueued/memory': 10,
 'start_time': datetime.datetime(2026, 5, 28, 22, 28, 11, 203301, tzinfo=datetime.timezone.utc)}
2026-05-28 15:28:15 [scrapy.core.engine] INFO: Spider closed (finished)
(scrapy-env) ┌─[leviathan@parrot]─[~]
└──╼ $ls
 10.129.72.114                   hashed.txt                  oradiag_leviathan   target-NFS
 cx_Oracle-8.3.0                 id_rsa                      Pictures            Templates
 cx_Oracle-8.3.0.tar.gz          important.txt               ReconSpider.py      tom_id_rsa
 Desktop                         inlanefreight.htb_ips.txt   ReconSpider.zip     tom-rsa
 dev.inlanefreight.htb_ips.txt   Music                       results.json        venvs
 Documents                      'NFS HTB'                    rockyou.txt         Videos
 Downloads                       nikto                       subdomains.txt
 flag.txt                        odat                        target
(scrapy-env) ┌─[leviathan@parrot]─[~]
└──╼ $cat results.json 
{
    "emails": [
        "emma.williams@inlanefreight.com",
        "jeremy-ceo@inlanefreight.com",
        "lily.floid@inlanefreight.com",
        "hans.mueller@inlanefreight.com",
        "support@inlanefreight.com",
        "cvs@inlanefreight.com",
        "freya.kartboom@inlanefreight.com",
        "manuel.pernilious@inlanefreight.com",
        "enterprise-support@inlanefreight.com",
        "info@inlanefreight.com",
        "john.smith4@inlanefreight.com",
        "samuel.dot@inlanefreight.com",
        "fiona.dante@inlanefreight.com",
        "info@themeansar.com",
        "david.jones@inlanefreight.com",
        "enterprise@inlanefreight.com"
    ],
    "links": [
        "https://www.inlanefreight.com/index.php/offices/#content",
        "https://www.inlanefreight.com/index.php/contact/#content",
        "https://www.inlanefreight.com/index.php/news/",
        "https://www.inlanefreight.com/index.php/about-us/#content",
        "https://www.inlanefreight.com/index.php/offices/",
        "https://www.inlanefreight.com/wp-content/uploads/2020/09/goals.pdf",
        "https://www.inlanefreight.com/#content",
        "https://www.inlanefreight.com/index.php/career/#content",
        "https://www.inlanefreight.com/index.php/news/#content",
        "https://www.themeansar.com",
        "https://www.inlanefreight.com",
        "https://www.inlanefreight.com/index.php/career/",
        "https://www.inlanefreight.com/index.php/contact/",
        "https://www.inlanefreight.com/index.php/about-us/",
        "https://www.inlanefreight.com/"
    ],
    "external_files": [
        "https://www.inlanefreight.com/index.php/news/pdf",
        "https://www.inlanefreight.com/wp-content/uploads/2020/09/goals.pdf"
    ],
    "js_files": [
        "https://www.inlanefreight.com/wp-includes/js/jquery/jquery-migrate.min.js?ver=3.3.2",
        "https://www.inlanefreight.com/wp-content/themes/ben_theme/js/bootstrap.min.js?ver=5.6.17",
        "https://www.inlanefreight.com/wp-includes/js/jquery/jquery.min.js?ver=3.5.1",
        "https://www.inlanefreight.com/wp-content/themes/ben_theme/js/jquery.smartmenus.bootstrap.js?ver=5.6.17",
        "https://www.inlanefreight.com/wp-includes/js/wp-embed.min.js?ver=5.6.17",
        "https://www.inlanefreight.com/wp-content/themes/ben_theme/js/navigation.js?ver=5.6.17",
        "https://www.inlanefreight.com/wp-content/themes/ben_theme/js/owl.carousel.min.js?ver=5.6.17",
        "https://www.inlanefreight.com/wp-content/themes/ben_theme/js/jquery.smartmenus.js?ver=5.6.17"
    ],
    "form_fields": [],
    "images": [
        "https://www.inlanefreight.com/wp-content/uploads/2021/03/AboutUs_01-1024x810.png",
        "https://www.inlanefreight.com/wp-content/uploads/2021/03/Career_02-300x235.jpg",
        "https://www.inlanefreight.com/wp-content/uploads/2021/03/Career_01-300x235.jpg",
        "https://www.inlanefreight.com/wp-content/uploads/2021/03/AboutUs_03-1024x810.png",
        "https://www.inlanefreight.com/wp-content/uploads/2021/03/AboutUs_04-1024x810.png",
        "https://www.inlanefreight.com/wp-content/uploads/2021/03/Offices_01-1024x359.png",
        "https://www.inlanefreight.com/wp-content/uploads/2021/03/AboutUs_02-1024x810.png"
    ],
    "videos": [],
    "audio": [],
    "comments": [
        "<!-- navbar-toggle -->",
        "<!-- /Navigation -->",
        "<!-- Right nav -->",
        "<!-- /Right nav -->",
        "<!--==================== feature-product ====================-->",
        "<!-- Navigation -->",
        "<!--\nSkip to content<div class=\"wrapper\">\n<header class=\"transportex-trhead\">\n\t<!--==================== Header ====================-->",
        "<!-- /navbar-toggle -->",
        "<!--Sidebar Area-->",
        "<!--/overlay-->",
        "<!--==================== TOP BAR ====================-->",
        "<!-- change Jeremy's email to jeremy-ceo@inlanefreight.com -->",
        "<!-- TO-DO: change the location of future reports to inlanefreight-comp133.s3.amazonaws.htb -->",
        "<!-- Logo -->",
        "<!-- Blog Area -->",
        "<!-- #secondary -->",
        "<!--==================== transportex-FOOTER AREA ====================-->",
        "<!-- #masthead -->"
    ]
}(scrapy-env) ┌─[leviathan@parrot]─[~]
└──╼ $






