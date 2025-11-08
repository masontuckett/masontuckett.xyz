+++
date = '2025-08-30T00:38:44-07:00'
title = 'DNS Hardening'
hideReply = true
author = "Mason Tuckett"
description = "A Brief Overview of DNS Hardening"
tags = [
    "web",
    "dns",
    "cybersecurity",
]
+++

A Brief Overview of DNS Hardening

## What is DNS?

[Domain Name System (DNS)](https://www.rfc-editor.org/rfc/rfc9499) is a protocol designed to translate IP addresses (IPv4/6) into human-readable addresses—e.g., [search.tuckettlab.xyz](https://search.tuckettlab.xyz).

{{< small >}}In practice the inverse: human-readable addresses > IP addresses{{</ small >}}

Without DNS, navigating the internet would be needlessly exhausting—requiring users to manually enter an IP address every time they were to access web content. 

Additionally, dynamic mapping executed via DNS allows for essential accessibility functions like geo-availability—providing content within relevant geographic locations—_significantly accelerating your web traffic_.

## How Does DNS Work?

![DNS Structure Diagram](/images/posts/dns-hardening/dns-structure.jpg)

DNS is a highly structured and hierarchical protocol—relying on several chains of trust to translate addresses. 

### Root Servers

- The top of the rung, served by 13 root server identifiers (12 organizations) distributed worldwide.
- Does not store zone records for domains, but they return referrals to the authoritative nameservers for the requested TLD (not individual domains): **.xyz(.)**.

### Top-Level Domains

- Second, just below root servers—housing domain extensions like: **.edu, .com, .net, and .xyz**.
- TLD servers return referrals to authoritative nameservers for second-level domains.

### Second-Level Domains

- Third in the chain, delegated _(by authoritative servers)_ and purchasable from domain registrars; second-level domains are registered under a TLD: **_tuckettlab_.xyz**.
- SLDs lie within authoritative servers, _essentially_ where your DNS records actually exist (zone): **A/AAAA**—pointing to valid IP addresses.

### Third-Level Domains

- Fourth in the chain, subdomains (within parent zone) managed by DNS records **(CNAME, A, AAAA)**: **_search_.tuckettlab.xyz**
- Commonly used for load balancing and service separation: **_mail_.tuckettlab.xyz (MX)**.

### Host/FQDN
- Last step: produces a fully qualified domain name (FQDN), which uniquely identifies a host—_accessible to users_. 
- For users: a recursive DNS server queries step by step: root **>** top-level domain server **>** second-level authoritative **>** subdomain (third-level domain) **>** resolver returns the corresponding record to the client.

### Response Codes

**NXDOMAIN:** A response code representing the absence or non-existence of a domain.

- Authoritative servers for the TLD respond NXDOMAIN if the entire SLD doesn’t exist under its respective TLD; otherwise, NXDOMAIN comes from the authoritative SLD server.

**NODATA (NOERROR/Empty):** A response code indicating that the domain _does_ exist, but no records of the requested type are present (A is present, AAAA is not). 

### Example Queries

```txt
### Successful Query ###
dig NS search.tuckettlab.xyz

; <<>> DiG 9.18.30-0ubuntu0.24.04.2-Ubuntu <<>> NS search.tuckettlab.xyz
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: !!!NOERROR!!!, id: 51001
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;search.tuckettlab.xyz.		IN	NS

;; AUTHORITY SECTION:
tuckettlab.xyz.		150	IN	SOA	ns1.vultr.com. dnsadm.choopa.com. 0 10800 3600 604800 3600

;; Query time: 1 msec
;; SERVER: 127.0.0.53#53(127.0.0.53) (UDP)
;; WHEN: Sat Aug 30 11:52:13 MDT 2025
;; MSG SIZE  rcvd: 113

### Unsuccessful Query ###
dig NS ?.xyz

; <<>> DiG 9.18.30-0ubuntu0.24.04.2-Ubuntu <<>> NS ?.xyz
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: !!!NXDOMAIN!!!, id: 61057
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;?.xyz.				IN	NS

;; AUTHORITY SECTION:
xyz.			773	IN	SOA	ns0.centralnic.net. hostmaster.centralnic.net. 3000828297 900 1800 6048000 3600

;; Query time: 80 msec
;; SERVER: 127.0.0.53#53(127.0.0.53) (UDP)
;; WHEN: Sat Aug 30 11:55:18 MDT 2025
;; MSG SIZE  rcvd: 99
```

## DNS Records

Within the managed zone, records are applied to guide users towards specific services.

### Common Records

{{< small >}}Concise for the Sake of Brevity{{< /small >}} 

- **NS** - Name Server _(Delegates authority for domains)_.
- **SOA** - Start of Authority (Provides administrative information about the zone)
- **A/AAAA** - IPv4/6 Addresses _(Points to the designated IP address.)_.
- **CNAME** - Canonical Name _(Points a name to another name; A/AAAA record is after the target)_.
- **MX** - Mail Record _(Normally points to mail subdomain)_.
- **PTR** - Pointer _(Reverse DNS is configured by your IP provider (VPS/ISP), not on forward zone; IP > domain)_.
- **TXT** - Text _(Commonly used for SPF/DKIM/DMARC for mail; can be informative)_.
- **CAA** - Certificate Authority Authorization _(Permits which certificate authorities can permit SSL/TLS Certs)_.
- **DNSKEY** - Domain Name System Key _(Stores public key in zone; used for DNSSEC validation)_.
- **DS** - Delegation Signer _(Stored in parent zone, contains hash of child zone's DNSKEY record)_.
- **CDNSKEY/CDS** - Child DNSKEY/DS _(Utilized for child to publish key details to parent zone)_.

### TXT-Related Records

- **SPF** - Sender Policy Framework _(Specifies which mail servers are authorized To send mail)_.
- **DKIM** - DomainKeys Identified Mail _(Cryptographic signature system for email content/headers)_.
- **DMARC** - Domain-based Message Authentication _(Instructs receiving mail servers to handle mail that fails SPF/DKIM checks; may report to admin's email)_.

### TTLs

When configuring DNS zones, you will have the option to give records a Time to Live (TTL)—representing the number of seconds a DNS record is cached before it is re-queried by an authoritative server.

## Record Representation

[Dig Web Interface](https://www.digwebinterface.com/)

```txt
### ! FORMAT ! ###
Domain (Root:.)   TTL Class Record Data Priority
 
### NS ###
dig NS tuckettlab.xyz

tuckettlab.xyz.		300	IN	NS	ns1.vultr.com.
tuckettlab.xyz.		300	IN	NS	ns2.vultr.com.

## SOA ##
dig SOA tuckettlab.xyz

tuckettlab.xyz.		300	IN	SOA	ns1.vultr.com. dnsadm.choopa.com. 0 10800 3600 604800 3600

### A/AAAA ###
dig A tuckettlab.xyz AAAA tuckettlab.xyz

tuckettlab.xyz.		3600	IN	A	149.28.67.27
tuckettlab.xyz.		3600	IN AAAA 2001:db8::25

### CNAME ###
dig CNAME www.tuckettlab.xyz

www.tuckettlab.xyz.  3600  IN  CNAME  tuckettlab.xyz.

### MX ###
dig MX tuckettlab.xyz

tuckettlab.xyz.		3600	IN	MX	10 mail.server.net.

### PTR ###
dig -x 149.28.67.27

27.67.28.149.in-addr.arpa. 3555	IN	PTR	tuckettlab.xyz.

### CAA ###
dig CAA tuckettlab.xyz

tuckettlab.xyz.		3600	IN	CAA	0 issue "tlsprovider.net"
tuckettlab.xyz.		3600	IN	CAA	0 issuewild "tlsprovider.net"
tuckettlab.xyz.		3600	IN	CAA	0 iodef "mailto:admin@admin.net"


### DNSKEY/DS ###
dig @9.9.9.9 DNSKEY tuckettlab.xyz DS tuckettlab.xyz

tuckettlab.xyz.		2761	IN	DNSKEY	257 3 13 UguAJ1bTfrBPNp+jHQ2oZNaPSUsUHaCnwRW+Wcxaf2BueGEmnMGAINj2 1b7gANfBBqqXoB6dGTeOBHfvjAvOdA==
tuckettlab.xyz.		3600	IN	DS	35175 13 4 865FE510C4CE390FF7285308B9351880A84039F1D75ACA278EF37F9D 71F605D8EC69C87E2F5660CA8BC5BC477A715868
tuckettlab.xyz.		3600	IN	DS	35175 13 2 871FD8CC8AFC7A1A6C60558112837CD7F288A20CC7F639709AD4A703 9C21E6B9

### TXT Mail Records ###
dig TXT tuckettlab.xyz TXT selector._domainkey.tuckettlab.xyz TXT _dmarc.tuckettlab.xyz 

tuckettlab.xyz.		3600	IN	TXT	"v=spf1 include:_spf.mail.net ~all"
_dmarc.tuckettlab.xyz.	3600	IN	TXT	"v=DMARC1; p=quarantine"
selector._domainkey.tuckettlab.xyz. 3600 IN TXT "v=DKIM1; k=key type; p=pubkey"
```

## Hardening DNS

{{< small >}}Now this is where the fun begins...{{</ small >}}

Hardening DNS is quite a bit more straightforward than dealing with content security policies and general systems. 

Most registrars allow you to manually edit your zone; if not, find a more consumer-friendly registrar.

For hardening purposes, some registrars hide DNSSEC behind a paywall; make sure to do your research.

If you are using a VPS for your infrastructure, it may be best to move your zone to your VPS provider—supplying a unified management pane with easy DNSSEC support.


### A/AAAA

It is best to keep these records current and remove anything that may be used by malicious actors. 

Many providers offer built-in proxies, where you can mask your origin traffic behind another IP address.

{{< small >}}I avoided AAAA records because they may increase my attack surface—exposing an additional interface that is susceptible if not hardened.{{</ small >}}

### CNAME

CNAME records do not resolve directly—requiring two lookups instead of one—slowing resolution (negligible).

This may not be an inherent issue, but wildcard records should be avoided because this opens your zone to a litany of abuse (largely impersonation).

Additionally, it can make administration far more difficult, as each chain must be validated under DNSSEC; unstable in general.

### MX 

Depending on your use case, you may want this enabled or disabled; are you using email with this domain?

If your domain does not require email, setting a [NULL MX](https://www.rfc-editor.org/rfc/rfc7505.html) record prevents any attempts at impersonation.

### MX TXT Records

Setting your [SPF TXT](https://www.rfc-editor.org/rfc/rfc7208.html) record to a hard fail further cements your defense against potential forgery **(-all HARD vs ~all SOFT)**.

Sane [DMARC TXT](https://www.rfc-editor.org/rfc/rfc7489.html) values are also incredibly important for securing your email.

With that in mind, DMARC has quite a selection of potential options:

- Policy **(p)**: _should not be set to quarantine_; it is best to outright **REJECT** (following a staging process) to prevent against phishing.

- Subdomain Policy **(sp)**: behaves in a similar fashion—but only covering subdomains; if not set, subdomains may default back to a weaker policy.

- Alignment Mode for DKIM **(adkim)**: is a great complement towards a secure DKIM setup; _it may contain the values strict_ **(s)** _or relaxed_ **(r)**.

- Alignment Mode for SPF **(aspf)**: behaves quite similarly to adkim but is a reinforcement for your SPF TXT record; _again, it may contain values strict_ **(s)** _or relaxed_ **(r)**.

- The Percentage of Messages **(pct)**: is related to how many potential messages these policies apply to; _optimal if set to_ **100 (100%)**.

- Aggregate Reports **(rua)**: may be enabled, which includes reporting for sent mail and DKIM/DMARC/SPF stats.

- Forensic Reports **(ruf)**: may also be enabled—sending immediate notices for DMARC evaluation failures (far noisier than rua).

- The Failure Options tag **(fo)**: complements both reporting features, ranging in values from **0** (both SPF/DKIM fail), **1** (either SPF/DKIM fails), **d** (DKIM), **s** (SPF).

{{< small >}}For my setup, I do not want ANY mail to be sent from this domain—as this is for self-hosted services ONLY.{{</ small >}}\
{{< small >}}DKIM will not exist, nor possibly be resolved.{{</ small >}}

```txt
### MX Record - NO MAIL ###
tuckettlab.xyz.		3600	IN	MX	0 .

### MX TXT Records - Complements NO MAIL Policy (No rua/ruf) ###
tuckettlab.xyz.		1736	IN	TXT	"v=spf1 -all"
_dmarc.tuckettlab.xyz.	1818	IN	TXT	"v=DMARC1; p=reject; sp=reject; adkim=s; aspf=s; pct=100"

### DMARC With Reporting ###
_dmarc.tuckettlab.xyz. 1818    IN  TXT "v=DMARC1; p=reject; sp=reject; rua=mailto:admin@tuckett.xyz; ruf=mailto:admin@tuckett.xyz; fo=1; adkim=s; aspf=s; pct=100"
```

### DNSSEC

I cannot stress how important it is to enable DNSSEC for your domains.

Doing so provides:

- **Authenticity**: _(Supplies strong cryptographic context for whether the record truly came from the authoritative server)_.
- **Integrity**: _(Prevents MITM tampering; guards DNS responses in transit)_.
- **Anti-Cache Poisoning**: _[(Impersonators CANNOT inject falsified DNS data into resolvers)](https://docs.projectsecurity.io/na101/attacks/dnsattack/)_.

It is as simple as configuring a few keys from your registrar and publishing them within your zone.

```txt
### Enable DNSKEY and DS Record for DNSSEC (USE STRONG ALGORITHMS) ###
tuckettlab.xyz.		3600	IN	DNSKEY	257 3 13 UguAJ1bTfrBPNp+jHQ2oZNaPSUsUHaCnwRW+Wcxaf2BueGEmnMGAINj2 1b7gANfBBqqXoB6dGTeOBHfvjAvOdA==

tuckettlab.xyz.		214	IN	DS	35175 13 4 865FE510C4CE390FF7285308B9351880A84039F1D75ACA278EF37F9D 71F605D8EC69C87E2F5660CA8BC5BC477A715868
tuckettlab.xyz.		214	IN	DS	35175 13 2 871FD8CC8AFC7A1A6C60558112837CD7F288A20CC7F639709AD4A703 9C21E6B9
```

### CAA

Also a massive oversight; [CAA records](https://www.rfc-editor.org/rfc/rfc8659.html) prevent rogue issuance by malicious CAs. 

CAA flags are set in 8-bit values (0-255); 0 is default (non-critical)—which allows soft fails (sometimes needed for compatibility). 

128 is functionally the only value worth using, as it is treated as critical and far more strict (must be understood by CAs). 

Both issue and issuewild are utilized for SSL/TLS distinction; wildcard certs may be utilized, though they are not advised.

```txt
# Sane Value - Only Allows Strict Issuance From Let's Encrypt ###
tuckettlab.xyz.		2103	IN	CAA	128 issue "letsencrypt.org"

# Disable Wildcard Certs - Prevents Against Wildcard Abuse (; = NO CAs) ###
tuckettlab.xyz.		2103	IN	CAA	128 issuewild ";"

# Incident Object Description Exchange Format - Instructs CAs Where to Send Reports ###
tuckettlab.xyz.		2103	IN	CAA	128 iodef "mailto:admin@tuckett.xyz"
```
 
### TXT Records

It may be useful to include important information within your TXT records, as most text may be inserted.

{{< small >}}I included important statements, checksums, onion location(s) (if not pulled by HTTP header), and my GPG info.{{</ small >}} 

```txt
### My TXT Records ###
tuckettlab.xyz.		1736	IN	TXT	"gpg-mirror=https://github.com/masontuckett/masontuckett.gpg"
tuckettlab.xyz.		1736	IN	TXT	"gpg=https://masontuckett.xyz/masontuckett.gpg"
tuckettlab.xyz.		1736	IN	TXT	"gpg-pubkey-fpr=7272B8036060E4590218A95D11F58765E7B21193"
tuckettlab.xyz.		1736	IN	TXT	"onion-search=ujozjm7regwuwszlcyx2g3jn6bgegrxk5x72q3om6sxb7kq7x5fgcwqd"
tuckettlab.xyz.		1736	IN	TXT	"privacy-policy=https://masontuckett.xyz/privacy-policy.txt"
tuckettlab.xyz.		1736	IN	TXT	"onion-library=ekybwg2m7fgocpnr5ofjjo2lrfunptquipqihfaxodand4qfm6rc3gqd"
tuckettlab.xyz.		1736	IN	TXT	"auth-statement=https://masontuckett.xyz/tor-mirror-statement.txt"
tuckettlab.xyz.		1736	IN	TXT	"sha512sums=https://masontuckett.xyz/sha512-hashes.txt"
```

### Live Demonstration

```txt
;; ANSWER SECTION:
tuckettlab.xyz.		3600	IN	A	149.28.67.27
tuckettlab.xyz.		3600	IN	RRSIG	A 13 2 3600 20250911000000 20250821000000 35175 tuckettlab.xyz. pLFdcvumIdn66a+KwNtzpAdXrL2/QoBaKK4Ah2+mb0c0uoh81gZMlHnI yHO3q4+D8ZiokKKGg+GOllTRFu0REw==
tuckettlab.xyz.		300	IN	NS	ns2.vultr.com.
tuckettlab.xyz.		300	IN	NS	ns1.vultr.com.
tuckettlab.xyz.		300	IN	RRSIG	NS 13 2 300 20250911000000 20250821000000 35175 tuckettlab.xyz. NKfKYWEfEC6QKsLpL8Skb948Mj2uCxl3YgUkweaFrBix97lZdqOGzxH/ XuUUry62xW/8x9SCZi+xfv7Zh7JLxA==
tuckettlab.xyz.		300	IN	SOA	ns1.vultr.com. dnsadm.choopa.com. 0 10800 3600 604800 3600
tuckettlab.xyz.		300	IN	RRSIG	SOA 13 2 300 20250911000000 20250821000000 35175 tuckettlab.xyz. zsnBppGRceCk6wVxB2ilh61eCEODg8zR4AHr/RhcCSjnvjjlc+rme1wP RAFZYN/0tmO2jTPsyE2GjgXNxnZ0tw==
tuckettlab.xyz.		3600	IN	MX	0 .
tuckettlab.xyz.		3600	IN	RRSIG	MX 13 2 3600 20250911000000 20250821000000 35175 tuckettlab.xyz. gt6nQjDrYIrXUhWV75i3BABNiGFV3/1NVeF1n8j1wrjZs81IC6umI66l NEHYfqq702CXnaq5kAe4qv/CBg3u6g==
tuckettlab.xyz.		3600	IN	TXT	"v=spf1 -all"
tuckettlab.xyz.		3600	IN	TXT	"sha512sums=https://masontuckett.xyz/sha512-hashes.txt"
tuckettlab.xyz.		3600	IN	TXT	"auth-statement=https://masontuckett.xyz/tor-mirror-statement.txt"
tuckettlab.xyz.		3600	IN	TXT	"privacy-policy=https://masontuckett.xyz/privacy-policy.txt"
tuckettlab.xyz.		3600	IN	TXT	"gpg-mirror=https://github.com/masontuckett/masontuckett.gpg"
tuckettlab.xyz.		3600	IN	TXT	"onion-library=ekybwg2m7fgocpnr5ofjjo2lrfunptquipqihfaxodand4qfm6rc3gqd"
tuckettlab.xyz.		3600	IN	TXT	"gpg=https://masontuckett.xyz/masontuckett.gpg"
tuckettlab.xyz.		3600	IN	TXT	"onion-search=ujozjm7regwuwszlcyx2g3jn6bgegrxk5x72q3om6sxb7kq7x5fgcwqd"
tuckettlab.xyz.		3600	IN	TXT	"gpg-pubkey-fpr=7272B8036060E4590218A95D11F58765E7B21193"
tuckettlab.xyz.		3600	IN	RRSIG	TXT 13 2 3600 20250911000000 20250821000000 35175 tuckettlab.xyz. 6TLPYOEnXr6ml3B7aTluAwR71fGfit6BQ93Lwumoq6VZZPu2fqd1r05i 8yEZa6tMVCJMhwDKb1GPhyXyWzmNAw==
tuckettlab.xyz.		300	IN	NSEC	_dmarc.tuckettlab.xyz. A NS SOA MX TXT RRSIG NSEC DNSKEY CAA
tuckettlab.xyz.		300	IN	RRSIG	NSEC 13 2 300 20250911000000 20250821000000 35175 tuckettlab.xyz. qTvUILjKw2EToMEjOiBE82s+frvHy4HL100cwCXNGxWVXbJvY3PmfTuy OVyBqpjw4NBHV26ORaNuuIfV5NB0yg==
tuckettlab.xyz.		3600	IN	DNSKEY	257 3 13 UguAJ1bTfrBPNp+jHQ2oZNaPSUsUHaCnwRW+Wcxaf2BueGEmnMGAINj2 1b7gANfBBqqXoB6dGTeOBHfvjAvOdA==
tuckettlab.xyz.		3600	IN	RRSIG	DNSKEY 13 2 3600 20250911000000 20250821000000 35175 tuckettlab.xyz. wrdN1qTW2ZfWZ1wX8I5q7ryFfVQH71LZAUEqN6xeKY+AQRy4TJiU/IEi gxuCoPXnxS1D7N9XHwMip0rUOpie0A==
tuckettlab.xyz.		3600	IN	CAA	128 iodef "mailto:admin@tuckett.xyz"
tuckettlab.xyz.		3600	IN	CAA	128 issuewild ";"
tuckettlab.xyz.		3600	IN	CAA	128 issue "letsencrypt.org"
tuckettlab.xyz.		3600	IN	RRSIG	CAA 13 2 3600 20250911000000 20250821000000 35175 tuckettlab.xyz. +dIuWxaDGXHeN7U/KlzxZOKZATc5a6Mrj7HzcGn88nAJ52dNPj4Suzud SNwZxeg9em8Vb2fVOqifpK/1FbAZrg==

;; Query time: 32 msec
;; SERVER: 8.8.8.8#53(8.8.8.8) (TCP)
;; WHEN: Sat Aug 30 16:18:37 MDT 2025
;; MSG SIZE  rcvd: 1878

;; ANSWER SECTION:
_dmarc.tuckettlab.xyz.	3600	IN	TXT	"v=DMARC1; p=reject; sp=reject; adkim=s; aspf=s; pct=100"

;; Query time: 259 msec
;; SERVER: 9.9.9.9#53(9.9.9.9) (UDP)
;; WHEN: Sat Aug 30 16:16:10 MDT 2025
;; MSG SIZE  rcvd: 118
```

{{< small >}} My TTLs are unable to go past 3600; my provider enforces TTL uniformity; other values are the remaining time within the TTL.{{</ small >}}

### Post-Hardening Evaluation

I believe if you are serious about your security, it is both intrinsically and extrinsically very important to verify your current security posture against other well-known sites/platforms.

I may plan on migrating my infrastructure to a platform that supports NSEC3 (currently stuck at NSEC)—which can prevent [trivial zone walking](https://dnsinstitute.com/documentation/dnssec-guide/ch06s02.html). 

{{< small >}}For comparison: "Cyber Guru" John Hammond's Training Platform.{{</ small >}}

```txt
;; ANSWER SECTION:
justhacking.com.	3524	IN	TXT	"v=spf1 include:_spf.google.com ~all"
justhacking.com.	3524	IN	TXT	"google-site-verification=WizBgAHBHgn4CAkraQKDWWpGqWqGCVianppbA3tfDGw"
justhacking.com.	14324	IN	MX	5 alt1.aspmx.l.google.com.
justhacking.com.	14324	IN	MX	10 alt4.aspmx.l.google.com.
justhacking.com.	14324	IN	MX	5 alt2.aspmx.l.google.com.
justhacking.com.	14324	IN	MX	10 alt3.aspmx.l.google.com.
justhacking.com.	14324	IN	MX	1 aspmx.l.google.com.
justhacking.com.	21524	IN	SOA	ns-cloud-a1.googledomains.com. cloud-dns-hostmaster.google.com. 12 21600 3600 259200 300
justhacking.com.	21524	IN	NS	ns-cloud-a2.googledomains.com.
justhacking.com.	21524	IN	NS	ns-cloud-a3.googledomains.com.
justhacking.com.	21524	IN	NS	ns-cloud-a4.googledomains.com.
justhacking.com.	21524	IN	NS	ns-cloud-a1.googledomains.com.
justhacking.com.	14324	IN	A	64.23.145.242

;; Query time: 16 msec
;; SERVER: 8.8.8.8#53(8.8.8.8) (TCP)
;; WHEN: Sat Aug 30 16:22:17 MDT 2025
;; MSG SIZE  rcvd: 479
```
