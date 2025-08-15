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

__Homelab__\
*August 2025*
- Deployed and configured OPNsense as the main router and firewall, setting up VLANs, appropriate subnets, DHCP, and firewall rules to segment and secure homelab traffic.
- Utilized Proxmox to host virtual machines for labs and services, resource allocation, and snapshot management—accomplished utilizing LXC containers and KVM virtual machines.
- Created a SOC lab environment using Kali Linux for vulnerability assessment, a Windows target for exploitation, and the Wazuh stack for real-time detection and prevention.
- Configured OPNsense firewalling with strict baselines—ensuring intervlan traffic is strict and within scope.
- Utilized Wireguard to secure sensitive traffic from self-hosted services to VPS proxy—terminated with TLS 1.2/1.3 certificates.

__VPS__\
*August 2025*
- Deployed a strict firewall policy: default-deny incoming, a strict iptables ruleset only allowing essential ports (non-standard IP-bound SSH port), with Tor service bound to localhost and isolated from public interfaces.
- Obtained an [A+ SSL Labs Score](https://www.ssllabs.com/ssltest/analyze.html?d=masontuckett.xyz), configured Let's Encrypt TLS (TLS 1.2/1.3) with strong AEAD ciphers, implemented HSTS preload, instituted DNSSEC to prevent MITM attacks, and restrictive CAA policies to prevent rogue issuance.
- Secured Nginx with tight content security policies, method restrictions (GET/HEAD only), rate-limiting, appropriate header limits, and privacy-respecting security headers.
- Deployed kernel-level security via extensive sysctl settings, unused protocol and module blacklisting (modprobe blocklist), and AppArmor confinement for Nginx and Tor to limit post-exploit impact.
- Incorporated a strong chain of trust: a signed Tor mirror statement, mirrored public GPG keys, and SHA-512 checksum proofs—all implemented across DNS TXT records and a [GitHub mirror](https://github.com/masontuckett/masontuckett.gpg).

## Verification

```sh
# Web Hosting (Headers)
Clearnet: curl -vkI https://masontuckett.xyz
Tor Mirror: curl -vkI http://izsq26kus3oo53hia253f3cvk5m2g3bdqi4o4obevxucln6zm2xasaid.onion

# DNS
! Main Domain !
dig masontuckett.xyz ANY +noall +answer +authority +additional +dnssec

! Self-Hosted Services !
dig tuckettlab.xyz ANY +noall +answer +authority +additional +dnssec

! Email !
dig tuckett.xyz ANY +noall +answer +authority +additional +dnnsec

# Documentation
Write Up: https://github.com/masontuckett/home-lab
```

# Volunteering

__Active Mentorship__\
*July 2025*
- Following my role as an instructor in the Ken Garff Esports Summer Tech Track—I continued to mentor an outstanding student who showed exceptional curiosity in the cybersecurity field.
- To further his learning, I designed and implemented a home lab tailored around a loosely inspired real-world enterprise environment—reproducing and mirroring my personal configurations.
- I offered continuous support—guiding my mentee through the process of studying/obtaining foundational IT certifications, and exploring his area of interest (penetration testing).
- A fully air-gapped SOC lab was created in Proxmox—allowing my mentee to gain exposure to critical industry toolsets (SIEM/IDS/IPS - Wazuh).
- 802.1Q VLANs were implemented ensuring a secure and realistic learning environment with appropriate WireGuard tunneling for public services.
- DNSSEC, CAA hardening, Nginx hardening (CSP/rate-limiting, UA gating), UFW firewalling, and MAC/sysctl tuning was utilized to ensure excellent availability and a reliable security posture.

## Verification

```sh
# Web Hosting (Headers)
Clearnet: curl -vkI https://smithbarlow.xyz
Tor Mirror: curl -vkI http://v55dqkmukq7hl6hty5sn6wlbcfn6ldadxt3h4wuynb2dyonpu5hrcmyd.onion

# DNS
dig smithbarlow.xyz ANY +noall +answer +authority +additional +dnssec

# Documentation
Post: https://masontuckett.xyz/posts/mentee-lab
Write Up: https://github.com/smithbarlow/Home-Lab
```

