+++
date = '2025-09-04T00:38:44-07:00'
title = 'Web Hosting'
hideReply = true
author = "Mason Tuckett"
description = "A Brief Overview of Nginx Hosting"
tags = [
    "web",
    "dns",
    "self-hosting",
]
+++

A Brief Overview of Hosting Web Pages 

## What is a Web Server?

![Web Server Diagram](/images/posts/web-hosting/web-server.webp)

A web server is specialized software responsible for delivering web content to users over HTTP/HTTPS. 

This can include static resources such as HTML/CSS/JS files—as well as images (JPG/PNG/WEBP) or archives (ZIP). 

### Static Sites

![Web Page Example](/images/posts/web-hosting/web-examples.webp)

At its simplest level, a web server serves static resources to users—rendering a fully functional web page or a browsable file directory; static sites are _static only_, serving **ONLY** static resources. 

However, administrative needs are often far more complex—requiring dynamic sites that are commonly powered by application frameworks or achieved with content management systems (CMS). 

### Dynamic Sites

Dynamic sites differ from static sites in that content is generated _dynamically_ instead of serving precompiled resources. 

When a user requests a page, the client sends a request in the form of [HTTP request methods](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods) to the web server—given to an application framework or CMS (often querying a database/app).

Thereafter, the web server responds with a finalized HTML response in real time—hence the _dynamic_ nature of the site.

{{< small >}} Some common CMS platforms include: WordPress and Drupal; common application frameworks include: Node.js, Django, Ruby on Rails.{{</ small >}}


## What is a Reverse Proxy?

![Reverse Proxy Diagram](/images/posts/web-hosting/reverse-proxy.webp)

Often bundled with web server software, a reverse proxy is an intermediary server that sits between clients and one or more back-end servers (e.g., dynamic services), forwarding requests directly to the web or application servers.

Reverse proxies are especially useful when hosting _dynamic_ sites that rely on application frameworks or CMS platforms.

These setups are often a combination of multiple components—such as databases, application servers, and caching layers (interlaced with [CDNs](https://www.cloudflare.com/learning/cdn/what-is-a-cdn/)). 

As workloads are quite resource-intensive and more complex than traditional static sites, administrators also get the benefits of:

- **Load Balancing**: _Distributing traffic across multiple servers_.
- **Caching**: _Frequently accessed content may be cached, reducing strain on back-end resources_.
- **Security**: _Obscures internal servers—hiding the identity of origin servers (if properly configured)_.
- **TLS/SSL**: _Simplifies the management of TLS/SSL certificates, as they can be handled at the proxy level_.

{{< small >}}Multiple reverse proxies can be chained together—for redundancy or specialized services/roles.{{</ small >}}

## Configuring a Reverse Proxy

For this demonstration I will be deploying Nginx on a Debian (Trixie) VPS; this won't differ significantly when compared to Ubuntu or RHEL derivatives.

There are various reverse proxies that _may_ be utilized, but [Nginx](https://www.cvedetails.com/vendor/10048/Nginx.html) is far more reliable (not to mention, lighter under high concurrency) than [Apache](https://www.cvedetails.com/vendor/45/Apache.html); [Traefik](https://traefik.io/) and [Caddy](https://caddyserver.com/) are also phenomenal options.

### Important Prerequisites

I'd recommend finding a hosting provider that gives you full access to your machine/instance; for full digital sovereignty, avoid AWS/Cloudflare. 

Ideally, it's best to point an A/AAAA record to your instance (if you have purchased a domain).

### Creating a New User

It is not advisable to use a root user for managing your host; create a new user instead.

```sh
### Create a New User (Root) ###
useradd -m -s /bin/bash user
usermod -aG sudo user

# ! Set a Password ! #
passwd user
```

### Generating an SSH Key

It is best to avoid RSA, as it is an **outdated** cryptographic standard compared to the much newer and robust ED25519 algorithm _(yes, even with the -b arg)_.

{{< small >}}RSA 4096 (integer factorization) is still mathematically stronger than ED25519, but RSA generally has lower security per bit than ED25519's ECC algorithm.{{</ small >}} 

```sh
### Generate a Secure ED25519 Key (Client) ###
# ! USE 100 KDF ROUNDS ! #
ssh-keygen -t ed25519 -a 100 -o -f ~/.ssh/sshkey

# ! ENTER A STRONG PASSPHRASE ! #
Generating public/private ed25519 key pair.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 

# ! Output Should Be Similar ! #
Your identification has been saved in /home/user/.ssh/sshkey
Your public key has been saved in /home/user/.ssh/sshkey.pub
The key fingerprint is:
SHA256:tCzBHeMoatZ5jR3pvKaGdRs83AHARTYEk3j5Ln8VccE user@9020MT
The key's randomart image is:
+--[ED25519 256]--+
|     o+OO    ... |
|    ..=*.=  . E  |
|    ..+.*.   o   |
|   o o X.o. .    |
|  + o ++S. . .   |
| o   .o.B.. .    |
|     o +o+ .     |
|    . .oo .      |
|     ..  .       |
+----[SHA256]-----+

### Deploy the Key (From Client) ###
scp ~/.ssh/sshkey.pub root@host:/home/user/

# ! Install the Key (On Host) ! #
mkdir -p /home/user/.ssh
install -m 600 /dev/null /home/user/.ssh/authorized_keys
cat /home/user/sshkey.pub >> /home/user/.ssh/authorized_keys

# ! Manage Permissions (On Host) ! #
chmod 700 /home/user/.ssh
chmod 600 /home/user/.ssh/authorized_keys
chown -R user /home/user

# ! Remove Key (Host) ! #
rm /home/user/sshkey.pub
```

### Firewall Configuration

Before we configure SSH (on the host), we need to manage our firewall first.

UFW is commonly bundled with stock Debian/Ubuntu server images, and while it is still more than enough—iptables is _far more_ granular.

```sh
### UFW (Root) ###
ufw allow http
ufw allow https
ufw allow 5555/tcp
ufw default deny incoming
ufw default allow outgoing
ufw status verbose

ufw enable
systemctl enable --now ufw

# ! OR ! #

### Iptables ###
# ! Flush Rules ! #
iptables -F

# ! UFW Equivalent ! #
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT

# ! Allow Loopback ! #
iptables -A INPUT -i lo -j ACCEPT

# ! Allow Established ! #
iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# ! Allow SSH (Non-standard Port) ! #
iptables -A INPUT -p tcp --dport 5555 -j ACCEPT

# ! Allow HTTP ! #
iptables -A INPUT -p tcp --dport 80 -j ACCEPT

# ! Allow HTTPS ! #
iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# ! Allow ICMP (Caution) ! #
iptables -A INPUT -p icmp -j ACCEPT

# ! Persistence ! #
apt update; apt install -y netfilter-persistent iptables-persistent
netfilter-persistent save
```

### Basic SSH Security 

Now that we've put everything else into place, it is time to secure SSH.

As you are hosting this publicly—it is best to have at _least_ a secure baseline for SSH access. 

This doesn't have to be too complex, but a non-standard port and public key authentication should suit this just fine.

{{< small >}}Take important notice of these configurations.{{</ small >}}

```sh
### /etc/ssh/sshd_config (Root) ###
# ! Set a Non-standard Port ! #
Port 5555

# ! Disable Root Login ! #
PermitRootLogin no

# ! Enable Public Key Authentication ! #
PubkeyAuthentication yes

# ! Disable Password Authentication ! #
PasswordAuthentication no
PermitEmptyPasswords no

# ! Enable Strict Mode ! #
StrictModes yes

# ! Permit User ! #
AllowUsers user

# ! Fallback Prevention ! #
ChallengeResponseAuthentication no

# ! Disable Forwarding (We Are Not a Router) ! #
AllowTcpForwarding no
GatewayPorts no

# ! Prevent Zombie Sessions ! #
ClientAliveInterval 300
ClientAliveCountMax 2

# ! Prevent Session Abuse ! #
MaxSessions 1

# ! Brute Force Prevention ! #
MaxAuthTries 3
LoginGraceTime 30
MaxStartups 3:10:30

# ! Prevent Agent Abuse ! #
AllowAgentForwarding no

# ! Restrict Keys ! #
HostKeyAlgorithms ssh-ed25519
PubkeyAcceptedKeyTypes ssh-ed25519

# ! Useful Additions ! #
Compression no
PermitUserEnvironment no

# ! Test Changes (No Output is Good) ! #
sshd -t

# ! Restart the Service ! #
systemctl restart --now ssh

### SSH In ###
ssh -i ~/.ssh/sshkey user@host -p 5555
```

## Deploying Nginx

After everything else has been completed, we can finally deploy Nginx!

For this demonstration, I will be installing both Nginx and Certbot to make the process as painless as possible (Let's Encrypt); Certbot will automatically renew your TLS/SSL certificate (via Cron).

```sh
### Install Nginx and Certbot (Sudo Now) ###
sudo apt update && sudo apt install -y nginx python3-certbot-nginx
sudo systemctl enable --now nginx

### Create a New Web Directory ###
sudo mkdir -p /var/www/webpage

### Obtain a Certificate ###
certbot certonly -d tuckett-test.xyz

# ! Output Should Be Similar ! #
How would you like to authenticate with the ACME CA?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1: Nginx Web Server plugin (nginx)
2: Runs an HTTP server locally which serves the necessary validation files under
the /.well-known/acme-challenge/ request path. Suitable if there is no HTTP
server already running. HTTP challenge only (wildcards not supported).
(standalone)
3: Saves the necessary validation files to a .well-known/acme-challenge/
directory within the nominated webroot path. A separate HTTP server must be
running and serving files from the webroot path. HTTP challenge only (wildcards
not supported). (webroot)
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
# ! We Are Using Nginx - Enter 1 ! #
Select the appropriate number [1-3] then [enter] (press 'c' to cancel): 1
Enter email address or hit Enter to skip.
 (Enter 'c' to cancel): admin@tuckett.xyz

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please read the Terms of Service at:
https://letsencrypt.org/documents/LE-SA-v1.5-February-24-2025.pdf
You must agree in order to register with the ACME server. Do you agree?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
# ! Agree to Terms ! #
(Y)es/(N)o: y

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Would you be willing, once your first certificate is successfully issued, to
share your email address with the Electronic Frontier Foundation, a founding
partner of the Let's Encrypt project and the non-profit organization that
develops Certbot? We'd like to send you email about our work encrypting the web,
EFF news, campaigns, and ways to support digital freedom.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
# ! Optional ! #
(Y)es/(N)o: n
Account registered.
Requesting a certificate for tuckett-test.xyz

Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/tuckett-test.xyz/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/tuckett-test.xyz/privkey.pem
```
### Configuring Nginx

Now we'll need to configure Nginx so we may publish our web page.

```txt
### /etc/nginx/nginx.conf ###
#! Turn Off Server Tokens (Bare Minimum) ! #
server_tokens off;

### /etc/nginx/sites-available/webpage ###
# ! Minimal Config ! #

# ! HTTP - Redirects to HTTPS ! #
server {
    listen 80;
    listen [::]:80;
    server_name tuckett-test.xyz;

    return 301 https://tuckett-test.xyz$request_uri;
}

# ! HTTPS ! #
server {
    listen 443 ssl;
    listen [::]:443 ssl;
    server_name tuckett-test.xyz;

    # ! Let's Encrypt Certificate ! #
    ssl_certificate     /etc/letsencrypt/live/tuckett-test.xyz/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/tuckett-test.xyz/privkey.pem;

    # Recommended TLS params (installed by certbot)
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    # ! Web Root ! #
    root /var/www/webpage;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

```sh
### Enable Web Page ###
# ! Test Configs ! #
sudo nginx -t 

# ! Avoids Conflicts ! #
sudo rm /etc/nginx/sites-enabled/default

# ! Symbolic Link ! #
sudo ln -s /etc/nginx/sites-available/webpage /etc/nginx/sites-enabled/webpage
```

### HTML Test

```html
### /var/www/webpage/index.html ###
# ! HTML Test ! #
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Web Hosting Guide - Success!</title>
</head>
<body>
  <h1>Web Hosting Guide - Success!</h1>
</body>
</html>
```

### Applying Permissions

It is important that your web root and web content has the proper permissions.

- **755 (Folders)**: Owner (www-data) has rwx, group (www-data) has rx, others have rx.
- **644 (Files)**: Owner (www-data) has rw, group (www-data) only have r, others only have r. 

```sh
### Apply Permissions (Script) ###
#!/bin/bash

set -euo pipefail

WEBROOT="/var/www/webpage"
USER="www-data"
GROUP="www-data"

sudo chown -R "$USER:$GROUP" "$WEBROOT"
sudo find "$WEBROOT" -type d -exec chmod 755 {} \;
sudo find "$WEBROOT" -type f -exec chmod 644 {} \;

sudo systemctl restart --now nginx
```

## Congratulations!

![Finalized Web Page](/images/posts/web-hosting/finalized.webp)

We finally have a working web page published—though we are _far_ from finished. 

This site is still highly vulnerable, and we'll need to implement CSP policies and some configuration changes to Nginx itself; this needs to be **production grade**. 

{{< small >}}In no way should you actually leave it this vulnerable—even if your site is purely static, you should harden it as much as possible.{{</ small >}}

[Check out my follow-up article regarding web hardening](/posts/web-hardening/).



