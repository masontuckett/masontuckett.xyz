+++
date = '2025-11-09T16:32:04-07:00'
title = 'Hidden Services'
author = "Mason Tuckett"
description = "A Basic Setup Guide for Tor/I2P Hidden Services"
tags = [
    "privacy",
    "self-hosting",
    "cybersecurity",
]
hideReply = true
+++

A Basic Setup Guide for Tor/I2P Hidden Services

## Hosting Hidden Services

Hosting hidden services is easier than it has ever been—as detailed documentation is now vastly widespread. 

Though I'd like to dispel misconceptions about the "darknet," *or more colloquially the "dark web."*

### The Dark Web

The dark net refers to traffic or protocols that are not within typical clear channels; the qualification is not obscurity or "depth" (deep web).

Typically facilitated under decentralized or independently structured protocols. 

{{< small >}}Both Tor (onion) and I2P (garlic) are the most ubiquitous examples of "dark web" hosting.{{</ small >}}

### Onion Routing

![Tor - Onion Routing Diagram](/images/posts/hidden-services/onion-routing.webp)

Tor utilizes onion routing to anonymize sensitive traffic.

Data is encapsulated in multiple layers of encryption—*with user traffic being propagated through a circuit*. 

Within the circuit, user traffic goes through a path of at least three nodes—obscuring the path of origin.

__Simple Breakdown:__

>Packet -> entry node.\
>Entry node (guard) decrypts the outer layer for the next hop.\
>Middle node decrypts its layer -> exit node.\
>Exit node decrypts the final layer -> revealing the original data and intended destination.

__More Information May Be Found Here:__ [ITP (NYU) Breakdown](https://itp.nyu.edu/networks/explanations/demystifying-the-dark-web-an-introduction-to-tor-and-onion-routing/).

### Garlic Routing

![I2P - Garlic Routing Diagram](/images/posts/hidden-services/garlic-cloves.webp)

I2P uses a similar privacy-preserving routing methodology—*but with a packet-switched mixnet design*. 

It is a natural extension of onion routing, bundling multiple cloves (messages) into a singular, encrypted "garlic" packet. 

It is more resilient compared to onion routing's individual handling, though slower because of its bundling. 

__Simple Breakdown:__

>Users build unidirectional tunnels (ingress/egress) via volunteer routers.\
>Data is effectively layered (similar to Tor)—bundled with padding to obscure data.\
>Packets travel through the sender’s egress tunnel -> the recipient’s ingress.\
>Each node decrypts its layer, unbinding cloves at the end.\
>Replies use separate tunnels to avoid correlation attacks.

{{< small >}}By its nature it consequently has excellent support for P2P services.{{</ small >}}

__More Information May Be Found Here:__ [I2P Project](https://geti2p.net/en/docs/how/garlic-routing).


## Tor Mirrors

Generating the required files is incredibly easy—*as the entire process has been largely demystified and automated*. 

I would avoid relays/nodes, as they may present an __INCREDIBLE__ amount of legal issues depending upon your jurisdiction.

Relays help the project stay alive, providing a viable route for users—*but it is important to note that __intelligence agencies co-opt this__*. 

Tor is not foolproof and does not automatically denote that your traffic is wholly secure __*("1337 DaRk WeB!!!")*__. 

The protocol by design is still quite flawed, as it is possible to decipher user origin by traffic correlation (entry/exit nodes).

Although this is really only utilized by state-level actors, *its ingrained design flaw is still unavoidable*.

{{< small >}}À la compromised or rogue exit nodes.{{</ small >}}

__KEEP THIS IN MIND:__ *Tor is NOT ENTIRELY QUANTUM-RESISTANT*. 

### Analytics

Every browser has a unique fingerprint driven by its UA, system fonts, canvas, screen size, and other ingrained behaviors—*presenting a significant tracking concern*.

It is quite foolish to use a third-party browser with Tor support, especially given that __THE MAJORITY__ of traffic is tied to Tor Browser's (Firefox) standardized fingerprint.

```txt
### Example Brave UA ###
Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/137.0.0.0 Safari/537.36
```

__Browser Tests:__ [Cover Your  Tracks (EFF)](https://coveryourtracks.eff.org/), [Am I Unique?](https://amiunique.org/), [Browser Leaks](https://browserleaks.com/).

{{< small >}}Browser behavior IS NOTICEABLY different between Gecko and Chromium-based browsers.{{</ small >}}

If you're even on Windows at all—__you're ngmi__; stop here and go *reinterpret* your threat model.

{{< small >}}I hope you enjoy Copilot+ Recall!!!{{</ small >}}

### Generating a Tor Mirror

For demo purposes, I'm assuming a bare-install scenario and any recent Debian derivative.

{{< small >}}I'm still in the process of migrating my VPS infrastructure to Podman; it's working fantastically on-prem though!{{</ small >}}

Trixie repositories have resolved Bookworm's persistent issue of ephemeral key loss—*resulting in Tor mirrors randomly failing*. 

```sh
### Install Tor ###
sudo apt update
sudo apt install -y tor

### Useful as This is Bare ###
sudo apt install -y apparmor apparmor-utils wget curl

### Enable/Start Tor ###
sudo systemctl enable --now tor

### Enable/Start AppArmor ###
sudo systemctl enable --now apparmor

### Sanity Check ###
sudo journalctl -xeu tor
sudo journalctl -xeu apparmor
```

{{< small >}}A Tor AppArmor profile should be installed; make sure to personally audit the file and make any appropriate alterations.{{</ small >}}

```sh
### Audit the Config ###
sudo vim /etc/apparmor.d/system_tor

### Parse the Config ###
sudo apparmor_parser -r /etc/apparmor.d/system_tor

### Enforce the Config ###
sudo aa-enforce /etc/apparmor.d/system_tor

### Sanity Check ###
sudo aa-status
sudo systemctl restart tor
```

__Configuring Torrc:__

{{< small >}}It's best to attach Tor to a non-standard port on loopback and let the reverse proxy (Nginx) handle it.{{</ small >}}

I cannot stress how crucial it is to select HiddenServiceVersion3 over prior versions (V1/V2). 

__I'll keep it germane:__ V2 utilizes its now antiquated SHA1/RSA-based PKI (1024-bit keys)—*[while V3 sports modern SHA3/Ed25519 tooling](https://gitlab.torproject.org/legacy/trac/-/wikis/doc/HiddenServiceNames)*.

{{< small >}}V3 is far more resistant to the ever-present birthday attack and other adjacent collision-based attacks.{{</ small >}}

__*Worth a read:*__ [SHAttered](https://shattered.io/).


```txt
### /etc/tor/torrc ###
## Bridge relays (or "bridges") are Tor relays that aren't listed in the
## main directory. Since there is no complete public list of them, even an
## ISP that filters connections to all the known Tor relays probably
## won't be able to block all the bridges. Also, websites won't treat you
## differently because they won't know you're running Tor. If you can
## be a real relay, please do; but if not, be a bridge!
#BridgeRelay 1
## By default, Tor will advertise your bridge to users through various
## mechanisms like https://bridges.torproject.org/. If you want to run
## a private bridge, for example because you'll give out your bridge
## address manually to your friends, uncomment this line:
#PublishServerDescriptor 0

HiddenServiceDir /var/lib/tor/example_mirror
# ! V3 NOT V2 ! #
HiddenServiceVersion 3
# ! Select a Non-Standard Port for Loopback ! #
# ! Useful for Reverse Proxy Integration ! #
HiddenServicePort 80 127.0.0.1:8081
```

After the necessary changes have been made, you will need to restart Tor to generate the hostname and keys.

```sh
### Restart Tor ###
sudo systemctl restart tor

### Check Output ###
sudo ls -lh /var/lib/tor/example_onion
sudo cat /var/lib/tor/example_onion/hostname

### Output Should Be Similar ###
# ! /var/lib/tor/example_onion/ ! #
total 16K
drwx--S--- 2 debian-tor debian-tor 4.0K Nov  7 23:38 authorized_clients
-rw-rw-r-- 1 debian-tor debian-tor   63 Nov  7 23:38 hostname
-rw-rw-r-- 1 debian-tor debian-tor   64 Nov  7 23:38 hs_ed25519_public_key
-rw------- 1 debian-tor debian-tor   96 Nov  7 23:38 hs_ed25519_secret_key

# ! /var/lib/tor/example_onion/hostname ! #
bh2k5od2ooelmftsiaxps3uod2fqwri5fmlh3opftl3aslk2jun3sgqd.onion
```

{{< small >}}Reverse proxy integration will be explained after the I2P section.{{</ small >}}

### Onion Vanity Addresses

[Mkp224o](https://github.com/cathugger/mkp224o) is a popular tool for generating custom addresses, though there are some limitations. 

Because we are generating these vanity addresses via brute-force key generation, you may want to keep your character prefix short. 

{{< small >}}I kept my prefix around five characters (mtuck), which only took around five seconds on my VPS.{{</ small >}}

__Prerequisites:__

```sh
### Download Dependencies ###
sudo apt install -y git gcc libc6-dev libsodium-dev make autoconf

### Clone the mkp224o Repo ###
git clone https://github.com/cathugger/mkp224o.git ~/mkp224o && cd ~/mkp224o

### Build From Source ###
./autogen.sh && ./configure && make

### Generate an Address ###
# ! -t (threads), -n (number) ! #
./mkp224o mtuck -v -n 2 -d example_vanity

# ! Output Should Be Similar ! #
set workdir: example_vanity/
sorting filters... done.
filters:
	mtuck
in total, 1 filter
using 1 thread
mtuckh7hurljundnkqgkgcrgrfkavcnhpxa4vmmoestgry752fxvorad.onion
mtuck44uiof3zofhzwl67csxc6qegyjccy4xihh45nytz5egp7ticpid.onion
waiting for threads to finish... done.
```
Then you will have to move the keys/hostname into its proper directory—*assuming keys/an address were already generated*. 

```sh
### Remove Old Files ###
# ! Cleans Keys/Hostname ! #
sudo rm /var/lib/tor/example_onion/h*

### Copy Newly Generated Files ###
sudo cp ~/mkp224o/example_vanity/gend-address/h* /var/lib/tor/example_onion/

### Set Proper Ownership ###
# ! Debian Uses debian-tor ! #
sudo chown -R debian-tor:debian-tor /var/lib/tor/example_onion

### Verify Perms Are Correct ###
sudo ls -lh /var/lib/tor/example_onion/

### Verify Hostname Is Correct ###
sudo cat /var/lib/tor/example_onion/hostname
```

## Generating an I2P Mirror

```sh
### Download I2PD ###
sudo apt update
sudo apt install -y i2pd

### Enable I2PD ###
sudo systemctl enable --now i2pd
```

{{< small >}}Because we are not using Podman or a similar container solution, we'll need to manually download a "baseline" AA config for I2P.{{</ small >}}

```sh
### Pull I2PD's AA Profile ###
sudo wget -O /etc/apparmor.d/usr.bin.i2pd https://raw.githubusercontent.com/PurpleI2P/i2pd/refs/heads/openssl/contrib/apparmor/usr.bin.i2pd 

### Audit the Config ###
sudo vim /etc/apparmor.d/usr.bin.i2pd

### Parse the Config ###
sudo apparmor_parser -r /etc/apparmor.d/usr.bin.i2pd

### Enforce the Config ###
sudo aa-enforce /etc/apparmor.d/usr.bin.i2pd

### Sanity Check ###
sudo aa-status
sudo systemctl restart i2pd
```
### Configuring I2PD

As I intend this article to be geared toward simply hosting hidden services, I unfortunately won't touch on I2PD-specific security (__for now, haha__). 

{{< small >}}Not to mention documentation is quite lackluster compared to the Tor Project.{{</ small >}}

I2PD will be the lightest solution (currently) available, with most mainstream repositories supporting it by default. 

```txt
### /etc/i2pd/i2pd.conf ###
## Bandwidth configuration
## L limit bandwidth to 32 KB/sec, O - to 256 KB/sec, P - to 2048 KB/sec,
## X - unlimited
## Default is L (regular node) and X if floodfill mode enabled.
## If you want to share more bandwidth without floodfill mode, uncomment
## that line and adjust value to your possibilities. Value can be set to
## integer in kilobytes, it will apply that limit and flag will be used
## from next upper limit (example: if you set 4096 flag will be X, but real
## limit will be 4096 KB/s). Same can be done when floodfill mode is used,
## but keep in mind that low values may be negatively evaluated by Java
## router algorithms.
# ! Changed Mine to P (up to You) ! #
bandwidth = P

[http]
## Web Console settings
## Enable the Web Console (default: true)
# ! Best to Disable the Web Console Later ! #
# ! Useful for Diagnostic Info ! #
enabled = false

## Port to listen for connections
## By default i2pd picks random port. You MUST pick a random number too,
## don't just uncomment this
# ! Set to a Custom Port ! #
# ! Allow TCP/UDP Through WAF/Nftables ! #
port = 4400
```

After we've configured I2PD itself, *we need to handle our firewall*!

```sh
### UFW Config ###
sudo ufw allow 4400/tcp
sudo ufw allow 4400/udp

# ! Verify ! #
sudo ufw status numbered

### IPtables Config ###
sudo iptables -A INPUT -p tcp -m tcp --dport 4400 -j ACCEPT
sudo iptables -A INPUT -p udp -m udp --dport 4400 -j ACCEPT

# ! Verify ! #
sudo iptables -L -v

# ! Save if Needed ! #
sudo netfilter-persistent save
```

Now that I2P can actually communicate within its network, *we can finally configure our eepsite*!

```txt
### /etc/i2pd/tunnels.conf ###
# ! Comment Out Anything Unnecessary ! #
# ! Fine To Edit Here ! #
[eepsite]
type = http
host = 127.0.0.1
port = 9090
keys = eepsite.dat
# ! Reverse Proxy Will Handle This ! #
gzip = false
```
__A Quick Restart Is in Order:__

```sh
### Generates our Key ###
sudo systemctl restart i2pd

### Check the Console (if Enabled) ###
curl http://127.0.0.1:7070
```

We're nearly there, but unlike Tor—I2PD doesn't provide readable hostnames.

You may access this through the web console, but I find it to be counterintuitive. 

{{< small >}}Especially considering you'd be required to use software like Lynx.{{</ small >}} 

You can absolutely use [i2pd-tools](https://github.com/PurpleI2P/i2pd-tools), but we can just pull the address directly.

{{< small >}}That repo is pertinent toward generating vanity addresses—but that doesn't concern me.{{</ small >}}


```sh
### Pulling the Base32 Address ###
sudo apt install -y xxd
sudo sh -c 'printf "%s.b32.i2p\n" $(head -c 391 /var/lib/i2pd/eepsite.dat | sha256sum | xxd -r -p | base32 | sed s/=//g | tr A-Z a-z)'

# ! Output Should Be Similar (Base32) ! #
mqfp6dvkrpf452mvwp2ouzh7dkn4swb7dhbaas76rtpurysdckoq.b32.i2p
```
{{< small >}}Repeat for as many sites as you desire to mirror.{{</ small >}}


## Reverse Proxy Integration

Integrating hidden services with your existing reverse proxy is largely hassle-free. 

Headers help clients switch to their desired mirror without having to scrounge for addresses. 

Tor has had existing support via the Onion-Location header, but seamless I2P discovery is not as widely adopted. 

Adding a custom header similar to Tor's onion-location or __Alt-Svc__ are the most sane options.

__*Feel Free to Reference My Web Hardening Article for Further Security Considerations*:__ [Web Hardening](/posts/web-hardening).

### Headers

```txt
### /etc/nginx/sites-available/main ###
server {
    # ! Tor - Onion Header ! #
    add_header Onion-Location "http://mtuckod2ooelmftsiaxps3uod2fqwri5fmlh3opftl3aslk2jun3sgqd.onion$request_uri" always;

    # ! I2P - Eeepsite Header ! #
    # ! Optional (Not Yet Widely Adopted) ! #
    add_header Alt-Svc 'i2p="eepsite.b32.i2p:80"; ma=86400' always;

    # ! Inlcude Security Headers ! #
    include /etc/nginx/snippets/security-headers.conf;
    include /etc/nginx/snippets/error-pages.conf;

# ! ↓ Site Config ↓ ! #
}
```
### Tor Nginx Config

```txt
### /etc/nginx/sites-available/tutorial_tor ###
# ! Clearnet Redirect ! #
server {
    listen 80;
    server_name tor;
    return 301 https://sld.tld$request_uri;
}
server {
    # ! Only Accepting Tor ! #
    listen 127.0.0.1:8081;
    server_name site.onion;

    # ! Include Web Contents ! #
    root /var/www/example_onion;
    index index.html;
    
    # ! Include Security Configs ! #
    include /etc/nginx/snippets/tor-security-headers.conf;
    include /etc/nginx/snippets/tor-error-pages.conf;

    # ! Blocks Local Leaks ! #
    allow 127.0.0.1;
    deny all;

    # ! Inlcude Security Headers ! #
    # ! Remove HTTPS Related Headers ! #
    include /etc/nginx/snippets/hidden-security-headers.conf;
    include /etc/nginx/snippets/hidden-error-pages.conf;

# ! ↓ Site Config ↓ ! #
}
```

```sh
### Link the Site ###
sudo ln -s /etc/nginx/sites-available/tutorial_tor /etc/nginx/sites-enabled/tutorial_tor

### Test Config ###
sudo nginx -t

### Reload Nginx ###
sudo nginx -s reload
```
### I2P Nginx Config

Configuring an I2P mirror is quite similar to Tor; just ensure that you are using a different port to properly segment traffic. 

```txt
### /etc/nginx/sites-available/tutorial_i2p ###
# ! Clearnet Redirect ! #
server {
    listen 80;
    server_name i2p;
    return 301 https://sld.tld$request_uri;
}
server {
    # ! Only Accepting I2P ! #
    listen 127.0.0.1:9091;
    server_name eepsite.b32.i2p;

    # ! Include Web Contents ! #
    root /var/www/example_eepsite;
    index index.html;
    
    # ! Include Security Configs ! #
    include /etc/nginx/snippets/i2p-security-headers.conf;
    include /etc/nginx/snippets/i2p-error-pages.conf;

    # ! Blocks Local Leaks ! #
    # ! I2P Uses a Larger Local Subnet ! #
    # ! You Need To Be Permissive ! #
    allow 127.0.0.0/8;
    deny all;

    # ! Inlcude Security Headers ! #
    # ! Remove HTTPS Related Headers ! #
    # ! Only Covering HTTP for I2P (for Now) ! #
    include /etc/nginx/snippets/hidden-security-headers.conf;
    include /etc/nginx/snippets/hidden-error-pages.conf;

# ! ↓ Site Config ↓ ! #
}
```

```sh
### Link the Site ###
sudo ln -s /etc/nginx/sites-available/tutorial_i2p /etc/nginx/sites-enabled/tutorial_i2p

### Test Config ###
sudo nginx -t

### Reload Again (if Necessary) ###
sudo nginx -s reload
```

### Accessing Your Mirrors

Because we aren't hosting on the conventional clearnet, we'll need to __LOCALLY__ download specialized software/profiles for accessing our sites.

```sh
### Tor ###
# ! Useful for Auto-Updates/Security ! #
# ! Including Flatseal ! #
flatpak install org.torproject.torbrowser-launcher com.github.tchx84.Flatseal

# ! Manage Flatpak Perms (if Desired) ! #
flatpak run com.github.tchx84.Flatseal

# ! Run the Browser ! #
flatpak run org.torproject.torbrowser-launcher

## I2P ###
# ! Install Locally ! #
# ! Flatpak was Finicky for Me ! #
sudo apt update
sudo apt install -y i2pd

# ! Edit the Local I2PD Config as Stated Above ! #
# ! Ignore Eeepsite Generation ! #
sudo systemctl enable --now i2pd

### Build a Specialized I2PD Browser (Firefox) ###
sudo apt install -y curl tar screen
cd ~; git clone https://github.com/daviduhden/i2pd-browser/
cd i2pd-browser/build && ./build

# ! Start the Browser ! #
cd ../; ./start-i2pd-browser.desktop

# ! Permit if Needed ! #
chmod +x ~/i2pd-browser/start-i2pd-browser.desktop

# ! Monitor the Web Console ! #
# ! NAT Issues May Arise ! #
# ! Can't Help Here (Sorry) ! #
http://127.0.0.1:7070
```

### Deployed Mirrors

{{< small >}}Taken from my open library.{{</ small >}}

__[Tor:](http://mtuckwbhnyx7pk3w3vvyqydv6sag5uxcbsx6dn5q5j4olpqmlkzosbad.onion/)__

![My Library's Tor Mirror](/images/posts/hidden-services/tor-site.webp)

__[I2P:](http://mqfp6dvkrpf452mvwp2ouzh7dkn4swb7dhbaas76rtpurysdckoq.b32.i2p/)__

![My Library's I2P Mirror](/images/posts/hidden-services/i2p-site.webp)



## Outcomes 

If everything went swimmingly, you should have both an I2P and Tor mirror deployed for your site.

Deploying additional mirrors follows the same playbook, but make sure (__REQUIRED__) to segment each mirror to its respective port.

{{< small >}}Note: A simple apt purge will resolve any residual misconfiguration.{{</ small >}}

__Another Thing to Keep in Mind:__

It would be wise to provide cryptographic proof of any operated sites; clear-signing is less clunky for individual statements. 

```sh
gpg --clear-sign privacy-policy.txt
gpg --clear-sign miror-statement.txt
```

## Closing

The "*dark web*" obtains an insanely bad rep from "*libertarian*" types, but don't let that dissuade you. 

Every corner of the internet has its bad actors—though admittedly there is some truth about the "*privacy*" community and their collective behavior. 

Be wary of others online, especially in Fediverse __cesspools__ like [Mastodon](https://fsi.stanford.edu/news/addressing-child-exploitation-federated-social-media) and similar E2EE __[CSAM-INFESTED](https://news.ycombinator.com/item?id=44592603)__ messengers like Matrix.

Any private or E2EE service procures some level of "*hacker*" mystique, but deploying and maintaining mirrors is rather straightforward (not to mention boring).

Architecting your own infrastructure provides a far better learning path than *lazily slapping* web files on GitHub/GitLab pages.

{{< small >}}Trust me, that is the work of a rock-bottom dullard; go lease a $5 VPS, you're better than that!{{</ small >}}
\
{{< small >}}Shoot for learning first and ditch performative "projects."{{</ small >}} 

*If you are operating a mirror or really any online service, I ask that you respect your users' privacy.*
