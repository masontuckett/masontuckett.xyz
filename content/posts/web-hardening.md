+++
date = '2025-11-07T00:38:44-07:00'
title = 'Web Hardening'
hideReply = true
author = "Mason Tuckett"
description = "A Brief Overview of Web Hardening"
tags = [
    "web",
    "self-hosting",
    "cybersecurity",
]
+++

A Brief Overview of Web Hardening

## What is Web Hardening?

Web hardening is the intentional process of enhancing an application's security posture—by reducing its effective attack surface.

The majority of these enhancements are made at the application layer—defining and permitting how web resources may be interacted with. 

__*In the following sections I will discuss hardening/mitigation strategies*__: locking down headers, HSTS, SNI traps, key exchange, and a live demo.

### Reverse Proxies

Reverse proxies are essential applications for establishing the secure flow of web traffic. 

Oftentimes, many reverse proxies are __NOT__ hardened by default—*enabling a high ceiling for potential administrative misconfiguration.*

[It is estimated that only a small fraction of major sites have adopted valid content security policies (CSP)](https://bluetriangle.com/blog/why-are-so-many-major-websites-operating-without-a-content-security-policy-csp).

{{< small >}}(Displayed below is a capture demonstrating a major educational institution; there are no hardening artifacts present).{{</ small >}}

![Weber State University Security Score (C-) - No CSP is implemented](/images/posts/web-hardening/csp-observatory.webp)

![Weber State University Security Score (C-) - Almost No Hardening Artifacts](/images/posts/web-hardening/csp-observatory-2.webp)

{{< small >}}If not mitigated, this essentially opens the door to widespread XSS attacks; this is still not enough, even with a WAF.{{</ small >}}



### The Web

Many corporations and entities are __STILL__ quite slow regarding their adoption of comprehensive web hardening. 

__Though some configurations may not be enacted due to compatibility concerns:__ 

- __TLS 1.1:__ *Mainly retained due to interoperability concerns in emerging markets (legacy Java runtimes, Windows/IE, and Android), [although this is declining](https://www.rfc-editor.org/rfc/rfc8996)*. 

- __Legacy Cipher Suites:__ *For many modern browsers these are not required (think key exchange akin to ~ TLS_RSA_WITH_AES_128_CBC-SHA)*.

{{< small >}}Insecure fallback ciphers (CBC) are still frequently utilized—set to their respective defaults; opt for secure AEAD ciphers for TLS 1.2+.{{</ small >}}

- __RSA Dependence:__ *A major oversight—as only antiquated browsers/runtimes lack forward secrecy ([ECDHE](https://datatracker.ietf.org/doc/html/rfc8422))*.

- __HTTP/2:__ *Largely a non-issue, only affecting the speed at which non-capable (HTTP/1.1) clients load content; even QUIC (HTTP/3) support is relatively sparse*.

## Terminology

### Key Concepts

- __TLS 1.3:__ *The latest iteration of [Transport Layer Security](https://datatracker.ietf.org/doc/html/rfc8446), a massive leap forward due to perfect forward secrecy; the TLS 1.2 era RSA exchange is deprecated—ephemeral key exchange is utilized (ECDHE).*

- __DNS Security Extensions (DNSSEC):__ *introduces digital signatures to [DNS records](https://www.rfc-editor.org/rfc/rfc9364)—preventing MITM and spoofing attacks (critical for modern sites).*

__DNSSEC Heavily Relies on PKI:__ 

>DNSKEY (root public key) signs DS for __.xyz__.\
>DNSKEY (tld public key) signs DS for __tuckett-test__.xyz.\
>KSK (public key) verified by __.xyz's__ DS.\
>ZSK (public key) signed by KSK (private key).\
>A record (__tuckett-test.xyz__) signed by ZSK (private key).

{{< small >}}Resulting in a robust chain of trust{{</ small >}}

- __DNS Certificate Authority Authorization (CAA):__ *[Permits which certificate authorities](https://www.rfc-editor.org/rfc/rfc8659.html) may permit TLS/SSL certificates; incredibly useful for preventing rogue issuance and impersonation.*

- __AEAD Ciphers:__ *Modern cryptographic primitive that is essential for TLS 1.3 deployment; encryption follows a *combined* step compared to CBC's separate encrypt and MAC functions.*

- __ALPN:__ *Enables the client to propose which protocol (HTTP/1.1, HTTP/2, or HTTP/3) it wants to use when interacting with the server (during the TLS handshake); [ALPN](https://www.rfc-editor.org/rfc/rfc7301).*

- __QUIC (HTTP/3):__ *Replaces traditional TCP web traffic (TCP + TLS + HTTP/2) to a singular layer over a UDP stream (443); headers are encrypted, clients send data on the first packet (0-RTT Resume vs HTTP/2 1-RTT)—[offering exceptional benefits in speed](https://datatracker.ietf.org/doc/html/rfc9114).*

- __Server Name Indication (SNI):__ *SNI is sent in the TLS ClientHello—allowing servers (vHosts) to select the correct certificate during the handshake.*

- __HTTP Strict Transport Security (HSTS):__ *A modern browser security feature and security policy—forcing clients to ONLY communicate using HTTPS.*

{{< small >}}The policy (max-age) is timed, and browsers will not default to HTTP until the policy expires.{{</ small >}}

- __Content Security Policy (CSP):__ *Web headers that define server-to-client behavior—specific policies that determine __how__ resources are accessed/interacted with, CSS elements, scripts, forms, embeds, objects, and requests.*

- __Subresource Integrity (SRI):__ *An essential security policy—[utilizing hash integrity](https://www.w3.org/TR/sri/) (external URL __->__ cryptographic hash) to prevent supply-chain attacks and MITM attacks—ensuring external scripts and styles are untampered.*

{{< small >}}Take notice of the SHA-384 hash (via OWASP).{{</ small >}}

```html
<script src="https://example.com/example-framework.js"
        integrity="sha384-oqVuAfXRKap7fdgcCY5uykM6+R9GqQ8K/uxy9rx7HNQlGYl1kPzQho1wx4JwY8wC"
        crossorigin="anonymous">
</script>
```

- __Online Certificate Status Protocol (OCSP):__ Allows clients to query a server in real-time for the revocation status of an [X.509 certificate](https://www.rfc-editor.org/rfc/rfc6960)—a stark contrast compared to CRLs, which periodically download massive lists. 

{{< small >}}Unfortunately, my provider, Let's Encrypt, has dropped support for OCSP—hence stapling's absence in my headers.{{</ small >}}

- __Certificate Transparency:__ CT is a public, append-only logging system that records every certificate __(TLS)__ issued by a trusted CA—preventing rogue CAs from issuing false certificates without detection.


## Implementation


### Use Cases

When hosting your own website, it is __*incredibly important*__ to scrutinize your threat model.

If you are using an SSG such as [Hugo](https://gohugo.io/) or [Jekyll](https://jekyllrb.com/), deep consideration needs to be made about whether the inclusion of scripts or external elements is warranted.

If your goal (*as is mine*) is to host a simple blog, it is wholly unnecessary to include JS—nonetheless, __non-free__ JS, CDNs, or fonts. 

{{< small >}}You CANNOT properly audit or trust third-party supply chains, and it is quite frivolous to introduce an avoidable attack surface.{{</ small >}}

If you are using script tags on an SSG, you defeat its intended purpose and design philosophy. 

{{< small >}}PLEASE DO NOT IMPLEMENT 'unsafe-inline' OR RAW HTML IN AN SSG!!!{{</ small >}}

### Where Script Tags Are Warranted

If your goal is to showcase a React portfolio or a web app, or you're utilizing a CMS—*then make sure to lock it down!*

Simple one-click deployments are fantastic, but you lose the entire benefit of *understanding* your infrastructure; 

{{< small >}}In quite the same fashion as the privileged (root) Docker, Portainer, and TrueNAS "enthusiasts."{{</ small >}}

## Prerequisites (DNS)

### CAA Records

```txt
### DNS Records ###
# ! Main CAA Record ! #
CAA tuckett-test.xyz    3600    128 issue letsencrypt.org

# ! Optional (Barring Wild) ! #
CAA tuckett-test.xyz    3600    128 issuewild ;
```

### DNSSEC

Refer to your registrar and hosting provider in order to activate these records.

{{< small >}}For the majority of users, it is incredibly straightforward, as this process has become highly automated (one-click generation).{{</ small >}}

__Example Records:__

```txt
### Enable DNSKEY and DS Record for DNSSEC (USE STRONG ALGORITHMS) ###
tuckett-test.xyz.	3600	IN	DNSKEY	257 3 13 ggyW7Ek6IYJeQ2vcxwyh3UFieSJ11yn3lKkBoiw6L4bmyu058Nmov0gl 46ZnmTICKQRx4O44N8VHhp0lXb6JEg==

tuckett-test.xyz.	3600	IN	DS	50834 13 2 C668859BAA36112BDD75F60D8AA157DFF99F1A3845CA3431914B4DA1 89A285C8
tuckett-test.xyz.	3600	IN	DS	50834 13 4 AC7F44830157D8D09B890DD345816C325EE4C663773D114A5836CBA9 29AD0A7EDF7032A082AD9E2D59D0D3938DAEA4A5
```

## Nginx

For this demonstration I will be using my existing setup, but I am currently in the process of migrating from Debian 12 to 13. 

For now, this is a bare install (standard binary package), so—unfortunately—I won't be able to showcase my current transition to rootless [Podman](https://podman.io/).

There are plenty of other fantastic reverse proxies that could be substituted for Nginx; __both [Caddy](https://caddyserver.com/) and [Traefik](https://traefik.io/) come to mind__.

__*Package Information*__

{{< small >}}A light selection of packages, as our threat model is largely static.{{</ small >}}

```sh
### Standard Nginx Release ###
# ! From My Mentee's System ! #
smith@smithbarlow:~$ apt-cache policy nginx
nginx:
  Installed: 1.26.3-3+deb13u1
  Candidate: 1.26.3-3+deb13u1
  Version table:
 *** 1.26.3-3+deb13u1 500
        500 https://deb.debian.org/debian trixie/main amd64 Packages
        500 https://debian.mirror.constant.com trixie/main amd64 Packages
        100 /var/lib/dpkg/status

### Clear Headers ###
smith@smithbarlow:/etc/nginx$ apt-cache policy libnginx-mod-http-headers-more-filter
libnginx-mod-http-headers-more-filter:
  Installed: 1:0.38-2
  Candidate: 1:0.38-2
  Version table:
 *** 1:0.38-2 500
        500 https://deb.debian.org/debian trixie/main amd64 Packages
        500 https://debian.mirror.constant.com trixie/main amd64 Packages
        100 /var/lib/dpkg/status
```

I will be sticking with __Debian Stable__—*avoiding Nginx's official Debian repository for stability concerns*.

Configuration will still reflect each area of interest—though ideally, I'd entirely avoid standard binary releases and opt for containerization.

If you are running Docker as root (when not necessary), you still open a considerable attack surface—*choose Podman (__daemonless__) as a sane alternative*.

{{< small >}}"Complexity theater" ignores the glaring inconsistencies with using "extensive" Ansible playbooks to deploy escapable ROOT containers!{{</ small >}}

### Ciphers

You may apply these universally, or across each designated vHost:

```txt
### /etc/nginx/nginx.conf ###
http {
# ! Proto Options (TLSv1.2 Optional) ! #
ssl_protocols TLSv1.2 TLSv1.3;

# ! Manually Restrict Insecure Ciphers (Optional - TLSv1.2) ! #
ssl_ciphers ECDHE+AESGCM:ECDHE+CHACHA20:!aNULL:!MD5:!DSS;

# ! Fine-Tuning (Preferring Server Ciphers is Still Valid) ! #
# ! Falls Back to secp384r1 ! #
ssl_ecdh_curve X25519:secp384r1;

# ! Redundant But Forces X25519 FIRST ! #
ssl_conf_command Groups X25519:P-384;

# ! Optional Stapling ! #
ssl_stapling on;
ssl_stapling_verify on;

# ! Optional Resolver ! #
# ! I'd Advise Against Using Corporate DNS ! #
resolver 1.1.1.1 8.8.8.8 valid=300s;

# ! Tweak if Needed ! #
resolver_timeout 10s;

# ! ↓ Nginx Config ↓ ! #
}
```

### HTTP/2 HTTP/3

__HTTP/2:__

```txt
### /etc/nginx/sites-enabled/tutorial ###
server {
    server_name search.tuckettlab.xyz;
    listen 443 ssl;
    listen [::]:443 ssl;
    http2 on;

# ! ↓ Site Config ↓ ! #
}
```

__HTTP/3:__

```txt
### /etc/nginx/sites-enabled/tutorial ###
server {
listen 443 quic reuseport;
listen [::]:443 quic reuseport;

# ! HTTP/3 Advertisement ! #
add_header Alt-Svc 'h3=":443"; ma=86400' always;

# ! ↓ Site Config ↓ ! #
}
```

{{< small >}}I am personally not going to use HTTP/3; there is no considerable benefit for my use case.{{</ small >}}
\
{{< small >}}I do not want another open port at this moment.{{</ small >}}

### Security Headers

I prefer using Nginx's snippets directory to keep my vHost configuration files tidy. 

{{< small >}}It's as simple as including a file from snippets to your designated vHost.{{</ small >}}

```txt
### /etc/nginx/sites-enabled/tutorial ###
server {
    server_name vhost;
    root /var/www/tutorial;
    index index.html;

    include /etc/nginx/snippets/security-headers.conf;

# ! ↓ Site Config ↓ ! #
}
```

Security headers can get quite complex, but I assure you that they are incredibly easy to manage. 

I will only be able to cover a select amount of implementations—as I don't want this article to be incessantly exhaustive; [Mozilla has a far more granular breakdown available](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Content-Security-Policy). 

{{< small >}}Figured below is a basic security configuration—geared towards simple sites.{{</ small >}}

```txt
### /etc/nginx/snippets/security-headers.conf ###
# ! Basic Static Site Security Configuration ! #
# ! Includes Some Redundancy for Demo Purposes ! #

### HSTS ###
# ! Non-Preloaded Site ! #
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

# ! Submit to Preload List (hstspreload.org)! #
# ! Use After Submission ! #
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;

### CSP ###
# ! Large List of Variables (Note the Possible External Objects) ! #
# ! Examples of Linkage: ! #
# API: script-src 'self' https://subdomain.sld.tld # 
# Reports: report-uri 'self' https://subdomain.sld.tld/page #
# Fonts: font-src https://subdomain.sld.tld #

### Content Security Policy ###
# ! Explicitly Disables Worrisome Vectors ! #
add_header Content-Security-Policy "frame-ancestors 'none'; default-src 'none'; script-src 'none'; style-src 'self'; style-src-elem 'self'; style-src-attr 'none'; img-src 'self'; connect-src 'none'; object-src 'none'; base-uri 'none'; font-src 'none'; frame-src 'none'; media-src 'none'; form-action 'none'; manifest-src 'none'; upgrade-insecure-requests" always;


### Additional Security Precautions ###
# ! MIME-Sniffing Prevention (Browser Respects Content-Type Header) ! #
add_header X-Content-Type-Options "nosniff" always;

### Origin Integrity ###
# ! Clickjacking Prevention (Legacy) - Blocks iframe Embedding ! #
add_header X-Frame-Options "DENY" always;

# ! Prevents Non-Origin Side-Channel Attacks ! #
# ! Completely Breaks Tabnabbing (window.opener or Similar) # !
add_header Cross-Origin-Opener-Policy "same-origin" always;

# ! Prevents Non-Origin Embeds ! #
# ! Isolates Content (Requiring CORP or CORS from cross-origin) ! #
add_header Cross-Origin-Embedder-Policy "require-corp" always;

# ! Blocks Non-Origin From Loading Resources ! #
# ! Resource Theft/Exfiltration (fetch() or Similar) ! #
add_header Cross-Origin-Resource-Policy "same-origin" always;

# ! Cross-Origin Isolation ! #
# ! Isolates Processes (Renders) ! #
# ! "?0" (Disables) Effectively Groups All Into One Process ! #
add_header Origin-Agent-Cluster "?1" always;

### Referrer Policy ###
# ! Does Not Disclose Source (Your Website -> Linked Site) ! #
# ! Prevents Analytics Bleed (Respects Privacy) ! #
add_header Referrer-Policy "no-referrer" always;

### Permissions Policy ###
# ! Important for Dynamic Sites or Services (Some May Be Deprecated, Best to Disable Everything Unnecessary) ! #
# ! Static Configuration - Note Format: service=() (Blocked) ! #
# ! Dynamic Configuration - Note Format: service=[](subdomain.sld.tld) (Integrates API into Permissions Policy) ! #
# ! Included for Demonstration Purposes (Small List): ! #

add_header Permissions-Policy "accelerometer=(), autoplay=(), battery=()" always;
```

### SRI

You will first need to generate a valid hash for your desired assets. 

```sh
### Generating the Hash ###
# ! Opting for SHA-384 ! #
user@pc:~$ printf 'sha384-%s\n' "$(openssl dgst -sha384 -binary eg.js | openssl base64 -A)"
sha384-bhFRYtN+I8sKH7+Y95efRQxwXuwViLZyl+42or4SE0+Q1uXOKHMK1Lym2jk2mq7k
```

__*Then add your generated hash to any applicable HTML page:*__

```html
<!-- crossorigin is Optional - Add if CORS Is Required (Best To Leave Anonymous for Privacy) -->
<script src="/eg.js"
        integrity="sha384-bhFRYtN+I8sKH7+Y95efRQxwXuwViLZyl+42or4SE0+Q1uXOKHMK1Lym2jk2mq7k"
        crossorigin="anonymous">
</script>
```

__*Implementing an SRI check in your reverse proxy goes as follows:*__

```txt
### /etc/nginx/sites-enabled/sri-example ###
server {
    server_name vhost;
    
    # ! Blocks (Enforces Valid Resources) Scripts/Styles if SRI Check Fails ! #
    add_header Integrity-Policy 'blocked-destinations=(script style), endpoints=(integrity)' always;

    # ! Report Only Mode ! #
    add_header Integrity-Policy-Report-Only 'blocked-destinations=(script style), endpoints=(integrity)' always;

    # ! Specifies Reporting Endpoint (if Needed) ! #
    add_header Reporting-Endpoints 'integrity="https://tld.sld/report"' always;

    # ! Considering Preloading if Possible (Add "crossorigin=" if Required) ! #
    # ! Unrelated to HSTS ! #
    add_header Link '</eg.js>; rel=preload; as=script; integrity="sha384-bhFRYtN..." always;   

# ! ↓ Site Config ↓ ! #
}
```

### Caching

Caching is incredibly effective in improving response times for HTTP clients. 

Serving immutable assets can *drastically* improve site performance.

```txt
### /etc/nginx/sites-enabled/tutorial ###
server {
    # ! Immutable Caching (Matches Common Files) ! #
    location ~* \.(css|js|woff2|ico|svg|png|jpg|jpeg|webp)$ {
        try_files $uri =404;
        add_header Cache-Control "public, max-age=31536000, immutable" always;
        expires 1y;
    }

# ! ↓ Site Config ↓ ! #
}
```

### Method Restriction

It is quite easy to limit the possible HTTP methods that threat actors may utilize for exploitation. 

Limiting your request methods ensures that certain types of common attack-related methods (e.g., __PUT__, __DELETE__, __TRACE__) cannot be performed.

{{< small >}}It is best to limit your vHost's possible request methods to your intended use case.{{</ small >}}

```txt
### /etc/nginx/sites-enabled/tutorial ###
server {
    location / {
        limit_except GET HEAD { deny all; }
        try_files $uri $uri/ =404;
    }

# ! ↓ Site Config ↓ ! #
}
```

### UA-Gating

UA-gating is largely trivial, as scanners/attackers frequently spoof user agents; __this is not a *bulletproof* security measure__. 

```txt
### /etc/nginx/nginx.conf ###
http {
    map $http_user_agent $allowed_ua {
    # ! Denied by Default ! #
        default 0;

    # ! Example Chromium Config (Simple Regex >120) ! #
        ~*Chrome/[1-9][2-9][0-9]|Chrome/1[2-9][0-9]|Chrome/[2-9][0-9][0-9] 1;
    }

# ! ↓ Nginx Config ↓ ! #
}

### /etc/nginx/sites-enabled/tutorial ###
server {
    if ($allowed_ua = 0) { return 403; }

# ! ↓ Site Config ↓ ! #
}
```

### Rate-Limiting

Rate-limiting is of exceptional importance when serving content over the internet. 

Astoundingly, it is somewhat rare to see operators implement this effectively. 

```txt
### /etc/nginx/nginx.conf ###
http {
# ! 10 MB of Entries - 10 Requests a Second ! #
limit_req_zone  $binary_remote_addr  zone=request_rate_limit:10m  rate=10r/s;
limit_conn_zone $binary_remote_addr  zone=connections_rate_limit:10m;

# ! ↓ Nginx Config ↓ ! #
}
```

```txt
### /etc/nginx/sites-enabled/tutorial ###
server {
    # ! Burst of 20 (More Forgiving) ! #
    limit_req  zone=request_rate_limit  burst=20  nodelay;
    limit_conn connections_rate_limit 20;

    # ! Returns 429 (Too Many Requests) ! #
    limit_req_status 429;
    limit_conn_status 429;

    # ! Optional Logging (Bare Response) ! #
    error_page 429 = @ratelimit;
    location @ratelimit {
    internal;
    include /etc/nginx/snippets/security-headers.conf;
    access_log /var/log/nginx/rate_limit.log;
    default_type text/plain;
    return 429 "429 - I See You\n";
    }

# ! ↓ Site Config ↓ ! #
}
```

{{< small >}}Related, yet VERY tight for my static site:{{</ small >}}

__Buffer/Body Protection:__

```txt
### /etc/nginx/sites-enabled/tutorial ###
server {
    # ! Overflow Protection (Miniscule Ceilings) ! #
    # ! Limiting Methods Will Mitigate Uploads ! #
    client_max_body_size 1k;

    # ! You Might Need to Bump These Up (2k, 2 4k) ! #
    client_header_buffer_size 1k;
    large_client_header_buffers 2 1k;

    # ! Proxy Specific (if Needed) ! #
    proxy_buffer_size 1k;
    proxy_buffers 2 1k;
    proxy_busy_buffers_size 1k;

    # ! Specifies How Long Timeouts Will Be ! #
    client_body_timeout 20s;
    client_header_timeout 20s;
    send_timeout 20s;

    # ! Specifies How Many Concurrent HTTP/2 Streams ! #
    # ! I am More Forgiving Here ! #
    http2_max_concurrent_streams 48;

# ! ↓ Site Config ↓ ! #
}
```

### SNI Trap

As modern browsers support SNI by default—*it is best to set an SNI __"trap"__ to reduce low-effort scanners and potential fingerprinting*.

{{< small >}}Optional, but highly recommended; institutes a "feature-lockout" mitigatory strategy{{</ small >}}

```txt
### /etc/nginx/sites-enabled/default ###
# ! HTTP #
server { listen 80 default_server;  listen [::]:80 default_server;  server_name _; return 444; }

# ! HTTPS ! #
server { listen 443 ssl default_server; listen [::]:443 ssl default_server; server_name _; ssl_reject_handshake on; }
```

*Another crucial step to reduce trivial spoofing is to implement a simple host __sanity__ check:*

```txt
### /etc/nginx/sites-enabled/tutorial ###
server {
    listen 443 ssl;
    # ! Must Be Hardcoded ! #
    server_name sld.tld;

    if ($host != "sld.tld") { return 421; }
    return 301 https://sld.tld$request_uri;
# ! ↓ Site Config ↓ ! #
}
```

## Privacy Precautions

Just as you may configure your infrastructure to respect its users' privacy, the infrastructure itself should be as invisible as possible.

Obviously, introducing unnecessary JS and other insipid tracking vectors opens a slew of potential routes for malicious actors—*hence the intrinsic case for web minimalism*.

Introspect, and reflect honestly on the potential ramifications of perpetuating incessant tracking; how will your own infrastructure backstab you?

{{< small >}}Analytics, even for small, static sites—presents nothing but NEGATIVE connotations (for me).{{</ small >}} 

### Fingerprinting Mitigation

It is impossible to completely hide your stack and its latent infrastructure. 

Although most reverse proxies have optional modules and built-in options for minimizing exposure.

{{< small >}}Trixie repositories contain a substantial, mitigatory module: libnginx-mod-http-headers-more-filter.{{</ small >}}

```txt
### /etc/nginx/nginx.conf ###
http {
    # ! DISABLE THIS IMMEDIATELY ! #
    # ! DISPLAYS YOUR FULL STACK ! #
	server_tokens off;

	# ! Strips Additional Server Info ! #
	more_clear_headers Server;
	more_clear_headers X-Powered-By;

    # ! HTTP Status Codes (Include More if Needed) ! #
	more_clear_headers -s 200 404 Server;

    # ! Proxy Specific # !
	proxy_hide_header Server;
	fastcgi_hide_header X-Powered-By;
	uwsgi_hide_header Server;
	scgi_hide_header Server;

    # ! SSL Specific ! #
	ssl_early_data off;
	ssl_session_tickets off;

# ! ↓ Nginx Config ↓ ! #
}
```

```txt
### /etc/nginx/snippets/error-pages.conf ###
error_page 404 =404 @e404;

# ! Optional - Link CSP if You're Crazy Like Me ! #
location @e404 { internal; default_type text/plain; include /etc/nginx/snippets/security-headers.conf; return 404 "404\n"; }
```

```txt
### /etc/nginx/sites-enabled/tutorial ###
server {
    # ! MAKE SURE TO ADD THIS ! #
    include /etc/nginx/snippets/error-pages.conf;

# ! ↓ Site Config ↓ ! #
}
```

## Outcomes

You can verify your security standpoint by utilizing a third-party tool such as [Mozilla's HTTP Observatory](https://developer.mozilla.org/en-US/observatory/analyze?host=masontuckett.xyz).

{{< small >}}My relevant curl results:{{</ small >}}

```sh
### SNI Check ###
myuser@masontuckett:~$ curl -vik https://gnu.org --resolve gnu.org:443:144.202.101.190
* Added gnu.org:443:144.202.101.190 to DNS cache
* Hostname gnu.org was found in DNS cache
*   Trying 144.202.101.190:443...
* ALPN: curl offers h2,http/1.1
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
* TLSv1.3 (IN), TLS alert, unrecognized name (624):
* TLS connect error: error:0A000458:SSL routines::tlsv1 unrecognized name
* closing connection #0
curl: (35) TLS connect error: error:0A000458:SSL routines::tlsv1 unrecognized name

### Rate-Limiting Check ###
me@pc:~$ bash -c 'for i in {1..21}; do curl -s -o /dev/null -w "%{http_code}\n" https://masontuckett.xyz & done; wait' | grep -E "200|429" | sort | uniq -c
     11 200
     10 429
```

## Closing

I have finally finished this article after a few weeks of complete dormancy—*as I am not (typically) the kind of person to lazily publish drafts*.

My leisure has become increasingly limited, but I do intend on trimming my own CSP/headers, as there is a lot of residual redundancy. 

{{< small >}}I DO wish this article was a bit more comprehensive.{{</ small >}}
\
{{< small >}}I'll likely revisit web hardening—adequately securing more typical, dynamic sites.{{</ small >}}

It doesn't impact my security whatsoever—*though I vehemently despise the bloat.*

I am nearly complete with my migration to rootless Podman—so expect a detailed follow-up with deployable configs soon.

__Next Up:__ [Rootless Podman](/posts/podman/).

{{< small >}}You'll see my hatred of Docker/Portainer/Tailscale and JS-laden MidwidtNAS on full display, haha!{{</ small >}}

