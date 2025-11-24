+++
date = '2025-08-14T00:38:44-07:00'
title = 'Projects'
menu = "main"
weight = 4
hideReply = true
author = "Mason Tuckett"
description = "Mason Tuckett's Project Page"
+++
# Projects

I am listing some things preemptively—as I am functionally working three jobs.

Migrating all my personal infrastructure, preparing for four certifications, pre-writing all my WGU papers, preparing six community education courses, and two independent programming courses... on top of my day jobs. 

## Personal

__CPE Learning Modules__\
*WIP*
- Independently developed a [C++ course](https://github.com/masontuckett/cpe-prep) structured around the C++ institute's Certified Entry-Level Programmer certification. 

__PCAP Learning Modules__\
*January, 2026*
- Independently developed a [Python course](https://github.com/masontuckett/pcap-prep) structured around the Python Institute's Certified Associate Python Programmer (PCAP) certification. 

__Home Lab__\
*August, 2025*
- Deployed and configured OPNsense as the main router and firewall, setting up VLANs, appropriate subnets, DHCP, and firewall rules to segment and secure home lab traffic.
- Utilized Proxmox to host virtual machines for labs and services, resource allocation, and snapshot management—accomplished utilizing LXC containers (Podman) and KVM virtual machines.
- Created a SOC lab environment using Kali Linux for vulnerability assessment, a Windows target for exploitation, and a full Wazuh stack for real-time detection and prevention.
- Configured OPNsense firewalling with strict baselines—ensuring inter-VLAN traffic is strict and within scope.
- Utilized WireGuard to secure sensitive traffic from self-hosted services to VPS proxy—terminated with TLS 1.2/1.3 certificates.



__VPS__\
*August, 2025*
- Deployed a strict firewall policy: default-deny incoming, a strict iptables ruleset only allowing essential ports (non-standard IP-bound SSH port), with Tor service bound to localhost and isolated from public interfaces.
- Obtained an [A+ SSL Labs Score](https://www.ssllabs.com/ssltest/analyze.html?d=masontuckett.xyz), perfect [PageSpeed Insights scores](https://pagespeed.web.dev/analysis/https-masontuckett-xyz-projects/mdabd69caq?form_factor=mobile), configured Let's Encrypt TLS (TLS 1.2/1.3) with strong AEAD ciphers, [implemented HSTS preload](https://hstspreload.org/?domain=masontuckett.xyz#submission-form), instituted DNSSEC to prevent MITM attacks, and restrictive CAA policies to prevent rogue issuance.
- Secured Nginx with tight [content security policies](https://developer.mozilla.org/en-US/observatory/analyze?host=masontuckett.xyz), method restrictions (GET/HEAD only), rate-limiting, appropriate header limits, and privacy-respecting security headers.
- Deployed kernel-level security via extensive sysctl settings, unused protocol and module blacklisting (modprobe blocklist), and AppArmor confinement for Nginx and Tor to limit post-exploit impact.
- Incorporated a strong chain of trust: a signed [Tor mirror statement](https://github.com/masontuckett/masontuckett.gpg/blob/main/tor-mirror-statement.txt), mirrored public GPG keys, and [SHA-512 checksum proofs](https://github.com/masontuckett/masontuckett.gpg/blob/main/sha512-hashes.txt)—all implemented across DNS TXT records and a [GitHub mirror](https://github.com/masontuckett/masontuckett.gpg).

![Mason Tuckett's Home Lab](/images/mason-tuckett-home-lab.webp)

### Verification

```sh
### Web Hosting (Headers) ###
# ! Clearnet ! #
curl -vkI https://masontuckett.xyz

# ! Tor ! #
curl -vI --socks5-hostname 127.0.0.1:9050 http://mtuckod2ooelmftsiaxps3uod2fqwri5fmlh3opftl3aslk2jun3sgqd.onion

### DNS ###
# ! Main Domain ! #
dig @9.9.9.9 masontuckett.xyz any +dnssec

# ! Self-Hosted Services ! #
dig @9.9.9.9 tuckettlab.xyz any +dnssec

# ! Email ! #
dig @9.9.9.9 tuckett.xyz any +dnssec

### Documentation ###
Write Up: https://github.com/masontuckett/home-lab
```

## Volunteering

__Active Mentorship (High School Student)__\
*July, 2025*
- Following my role as an instructor in the Ken Garff Esports Summer Tech Track—I continued to mentor an outstanding student who showed exceptional curiosity in the cybersecurity field.
- To further his learning, I designed and implemented a home lab tailored around a loosely inspired real-world enterprise environment—reproducing and mirroring my personal configurations.
- I offered continuous support—guiding my mentee through the process of studying/obtaining foundational IT certifications, and exploring his area of interest (penetration testing).
- A fully air-gapped SOC lab was created in Proxmox—allowing my mentee to gain exposure to critical industry toolsets (SIEM/IDS/IPS - majority Wazuh).
- 802.1Q VLANs were implemented ensuring a secure and realistic learning environment with appropriate WireGuard tunneling ([TLS 1.2/1.3](https://www.ssllabs.com/ssltest/analyze.html?d=smithbarlow.net&latest)) for public services.
- DNSSEC, CAA hardening, Nginx hardening ([CSP](https://developer.mozilla.org/en-US/observatory/analyze?host=smithbarlow.net)/rate-limiting, UA-gating), UFW firewalling, and MAC/sysctl tuning was utilized to ensure excellent availability and a reliable security posture.

[A Brief Write Up May Be Found Here.](/posts/mentee-lab/)

### Verification

```sh
### Web Hosting (Headers) ###
# ! Clearnet ! #
curl -vkI https://smithbarlow.net

# ! Tor ! #
curl -vI --socks5-hostname 127.0.0.1:9050 http://3xem3fnyur2igj2ldd7vqp7lmecxlwxm4kdcmsbcqrqv2i3gn7e4y4id.onion

### DNS ###
dig @9.9.9.9 smithbarlow.net any +dnssec

### Site Repo ###
https://github.com/smithbarlow/smithbarlow.net
```

__Active Mentorship (College Student)__\
*October, 2025*
- As a mutual friend of mine returned from his mission (LDS), he confided in me that he was unsure of what path he wanted to take with his career.
- Understanding what his goals, aims, and ambitions were—I recommended that he seek Weber State University's [Management Information Systems (MIS)](https://www.weber.edu/Majors/management-information-systems.html) program.
- I closely advised his transition from Economics to MIS, with the context of his propensity for working closely under tech startups; [he is prime Carnegie Mellon material](https://www.weber.edu/wsumagazine/fall-2020/weber-watch/WSU-MIS-graduates.html). 
- I offered continuous support, discussing his prospects on many multiple-hour calls—building an academic road map for achieving his goals. 
- I mirrored my configuration and setup of my high school mentee—constructing an enterprise-grade lab and an online portfolio template. 
- I adamantly emphasized the significance of improving his technical skill set with a certification game plan before MIS enrollment. 

### Verification

{{< small >}}Information will be provided as soon as it goes live.{{</ small >}} 




