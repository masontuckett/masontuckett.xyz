+++
date = '2025-12-06T21:27:32-07:00'
title = 'Podman'
author = "Mason Tuckett"
description = "A Brief Overview of Podman"
tags = [
    "self-hosting",
    "home-lab",
    "virtualization",
    "cybersecurity",
]
hideReply = true
+++

A Brief Overview of Podman

## Containerization



## Linux Namespace

![Linux Kernel Diagram](/images/posts/podman/linux-kernel.webp)

## Docker

Docker is an incredibly convenient and largely foolproof solution for implementing operating-system-level virtualization in the manner of streamlined packages (containers). 

By its nature, production environments rely upon fully packaged images with base sub-systems (e.g., Alpine/Bottlerocket/Ubuntu)—to bypass environmental complexity.

Developers and system maintainers may simply deploy an immutable image, ensuring interoperability concerns are adequately addressed. 

{{< small >}}No dependency hell, no version mismatch, and no cross-platform interference.{{</ small >}}

__Typical Deployment Pattern:__

- A Unix-like host provides OS-level virtualization.
- Ansible playbook/Docker compose YMLs are configured.
- Containers are mediated/deployed via a web console.
- Often run as root, only application/container-specific variables are addressed. 

```yaml
### Example Ansible Playbook ###
# ! Via docs.ansible.com ! #
- name: Update web servers
  hosts: webservers
  remote_user: root

  tasks:
  - name: Ensure apache is at the latest version
    ansible.builtin.yum:
      name: httpd
      state: latest

  - name: Write the apache config file
    ansible.builtin.template:
      src: /srv/httpd.j2
      dest: /etc/httpd.conf

- name: Update db servers
  hosts: databases
  remote_user: root

  tasks:
  - name: Ensure postgresql is at the latest version
    ansible.builtin.yum:
      name: postgresql
      state: latest

  - name: Ensure that postgresql is started
    ansible.builtin.service:
      name: postgresql
      state: started
```

### Caveats

Ironically, in common production environments, Docker configuration is often left unchanged—ignorantly left running as root.

When left misconfigured, a containerized root environment *can* correspond to the host's underlying root user.

In a rootful state, it presents a massive, glaringly obvious increase in post-exploit severity.

{{< small >}}Rootful containers will map to UID 0.{{</ small >}}

## Podman

### Caveats


## Configuring Docker

### User Configuration



## Configuring Podman
