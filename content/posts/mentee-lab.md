+++
date = '2025-07-12T21:50:48-06:00'
title = 'Mentee Lab'
hideReply = true
author = "Mason Tuckett"
description = "An Overview of a Homelab I designed for My Mentee"
tags = [
    "self-hosting",
    "home-lab",
    "security",
]
+++
## My Thought Process
Following my role as an instructor in the Ken Garff Esports Summer Tech Track—I continued to mentor an **outstanding** student who showed **exceptional** curiosity in the cybersecurity field.

To further his learning, I designed and implemented a home lab tailored around *a loosely inspired real-world enterprise environment.* 

## Network Infrastructure

![Smith's Home Lab Diagram](/images/posts/mentee-lab/Smith-Network-Diagram.png)

### Subnetting

I chose not to implement a flat /24 network as it is inherently insecure for almost any sort of network—especially for exposed services.

I decided to keep subnet assignments tight and centered around a usable pool that heavily prevents lateral movement—ensuring a security-first mindset.

/29 was used for the native VLAN **(VLAN 1)**—primarily due to it being the trusted subnet and making troubleshooting far easier; *I thought that it was far more appropriate for his needs.*

**VLAN 2** is on a /30 subnet, which limits potential attack surface, and is solely for the Proxmox management interface.

**VLAN 3** needs little to no usable hosts because it is intended only for logging services.

**VLAN 5** is far more generous with a /27 subnet, which would be best for tightening and expandability.

Segmenting services would've been best, but again—*this setup needs to be accessible for independent troubleshooting.*

With that in mind, DHCP is enabled on VLAN 1 in case his network configuration interferes with deliverability; **my mentee has three "free" untagged ports on VLAN 1 for troubleshooting purposes.**

My mentee has a keen interest in penetration testing, so I integrated a SOC lab into his home lab for learning purposes.

It is entirely virtual, isolated on to **vmbr4** with no physical network integration; *this VLAN is fully air gapped.*

### Network Hardware

OPNsense was used as a central router/firewall with extensive ACLs to prevent unnecessary inter-VLAN traffic and exposure.

*I found it very beginner-friendly while exposing my mentee to real-world toolsets.*

A simple Netgear gigabit managed switch **(GS308Ev4)** was used to establish trunk ports between OPNsense and his Proxmox "outputs."

Three trunk ports were utilized for carrying traffic out of his Proxmox node: an upstream trunk for VLANs **(ibg0)** 1, 2, 3, and 5, and two independent ports **(enp1sfp0 and enp1sfp1)** for VLANs 3 and 5.

*A dummy __PVID of 4094__ was used for untagged traffic on the three trunk ports—limiting potential traffic errors/security risks.*

Every other port—including the Proxmox management interface—was kept untagged.

### Virtualization

**_I cannot state how asinine it is to run everything on bare metal_**—a mistake I see far too frequently in student-led home labs.

It makes management/expansion an absolute headache and leaves every process on a *"level playing field"*—severely increasing potential attack surface.

With that in mind, I opted for [Proxmox](https://www.proxmox.com/en/products/proxmox-virtual-environment/overview), an open-source competitor to VMware's EsXi platform—offering an enterprise experience without VMware's absurd licensing fees.

*Linux bridges were created to match each trunk port and were tagged appropriately within Proxmox.*

### Linux Bridges

![Bridge 0 Config](/images/posts/mentee-lab/vmbr0.png)

**vmbr0** is native to Proxmox (VLAN 2)—untagged and *purposefully **not** VLAN aware*.

![Bridge 1 Config](/images/posts/mentee-lab/vmbr0.png)

**vmbr1** is for logging purposes (VLAN 3).

![Bridge 2 Config](/images/posts/mentee-lab/vmbr2.png)

**vmbr2** is used for self-hosted services (VLAN 5). 
 
![Bridge 4 Config](/images/posts/mentee-lab/vmbr4.png)

**vmbr4** is for internal use only (VLAN 4).

### Implementing Virtual Machines

For most home labs, I would implement a hardened frontend reverse proxy—though for his setup, it's largely unnecessary *(for now)*. 

The lab mostly consists of LXC containers and KVM virtual machines—as most services would be inoperable/inefficient if using Docker containers.

Each machine is manually assigned an IP address to avoid unnecessary DHCP servers running on each VLAN interface.

![Example Networking Config](/images/posts/mentee-lab/example-config.png)

VLAN tagging also occurs on each virtual network interface—his PaperMC server, as noted above, is rate limited to avoid exhausting his network; granted his hardware is more than capable, I found it to be highly important.

## Future Plans

Eventually, we will tunnel all relevant traffic through a WireGuard server hosted on the cloud so his services are masked. 

##### *Likely two hardened Nginx reverse proxies on each end—with TLS 1.3 termination on the VPS*

For now, the only service that is exposed is his PaperMC instance *(on a nonstandard port of 65555)*—though that will likely expand given his needs will increase.

OpenMediaVault will likely remain for internal use only, although Nextcloud may replace that in the future (tunneled).

## Documentation

*I want to stress that this was co-designed with my mentee, and he was actively involved in this process.*

##### *I am extremely proud of his performance and technical aptitude in implementing/maintaining this home lab.*

Extensive documentation, including interface configs, switch configs, ACLs, and Proxmox scripts/configs, may be found [here](https://github.com/masontuckett/Smith-Home-Lab) or at [smithbarlow.xyz](https://smithbarlow.xyz).

