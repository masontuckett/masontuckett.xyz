+++
date = '2025-05-01T16:36:37-06:00'
title = 'GNU Privacy Guard'
author = "Mason Tuckett"
description = "A Brief Guide on Managing GPG keys"
tags = [
    "gpg",
    "crypto",
    "privacy",
]
hideReply = true
+++
A Brief Overview of GNU Privacy Guard (GPG)

## What is GPG?

![Public Key Encryption Diagram](/images/posts/gpg/pgp.svg)

GNU Privacy Guard (GPG) is largely a modern replacement—or rather, an extension—of Pretty Good Privacy (PGP), which has been effectively abandoned by Symantec.

GPG is a modern implementation of the OpenPGP standard, often utilized on both *nix and Windows systems. 

Many modern platforms and services are designed to be compatible with both GPG and OpenPGP tools, facilitating secure communication and non-repudiation via cryptographic signatures.

One of its most ubiquitous use cases is in [*nix package management](https://wiki.debian.org/SecureApt)—leveraging Public Key Infrastructure (PKI) to ensure integrity and authenticity within the package management chain of trust. 

## Generating Keys

```bash
gpg --full-generate-key
```

After initializing key generation, you'll be prompted with a menu like this.

```bash
Please select what kind of key you want:
   (1) RSA and RSA
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
   (9) ECC (sign and encrypt) *default*
  (10) ECC (sign only)
  (14) Existing key from card
Your selection? 
```
Choose **(9)** — this creates an Elliptic Curve Cryptography (ECC) key pair, which is far more modern and efficient compared to RSA.

Generally speaking, ECC keys are considered more secure per bit [compared to RSA keys](https://sslinsights.com/ecc-vs-rsa/)—offering equivalent security while being far faster/portable.

```bash
Please select which elliptic curve you want:
   (1) Curve 25519 *default*
   (4) NIST P-384
   (6) Brainpool P-256
```
Choose **(1)** — ED25519 is a sane default and is widely adopted.

```bash
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
```

Choose at your discretion—though it would be wise to periodically rotate keys.

For all intents and purposes, **(0)** is fine. 

```bash
GnuPG needs to construct a user ID to identify your key.

Real name: Mason Tuckett
Email address: mason@tuckett.xyz
Comment: test key
You selected this USER-ID:
    "Mason Tuckett (test key) <mason@tuckett.xyz>"
```
Enter any information that you may find relevant.

Make sure to select a strong, alphanumeric password that includes symbols—preferably over twelve characters long.

```bash
public and secret key created and signed.

pub   ed25519 2025-05-02 [SC]
      07B41B22F08862AC61F1DB5E462E05498C4C7F06
uid                      Mason Tuckett (test key) <mason@tuckett.xyz>
sub   cv25519 2025-05-02 [E]
```

*You should see an output similar to this.* 

**Now we can move on to managing our key pair!**

## Managing Keys

Once you've generated your GPG key, you'll need to share your public key so others may verify your digital signature and send you encrypted messages.

```bash
gpg --fingerprint

pub   ed25519 2025-05-02 [SC]
      07B4 1B22 F088 62AC 61F1  DB5E 462E 0549 8C4C 7F06 # Fingerprint
uid           [ultimate] Mason Tuckett (test key) <mason@tuckett.xyz>
sub   cv25519 2025-05-02 [E]
```

Identifying your fingerprint is crucial for verifying the authenticity of your key; it is essentially a summary of your key pair.

Providing a fingerprint **(07B41B22F08862AC61F1DB5E462E05498C4C7F06)** prevents impersonation—which is commonly utilized for man-in-the-middle attacks. 

_Make sure to provide this when sharing your key._

### Exporting Keys

To share your key with others, it is necessary to properly export your public key.

It's best to use the _\--armor (ASCII)_ option for portability, as it makes everything readable and compatible across all platforms.

```bash
# Export the public key
gpg --armor --export 07B41B22F08862AC61F1DB5E462E05498C4C7F06 > pubkey.gpg

# Verify the public key
cat pubkey.gpg
-----BEGIN PGP PUBLIC KEY BLOCK-----

mDMEaBQztBYJKwYBBAHaRw8BAQdAoXjOBHJ2kHnL6zFhCEdsPfncq0CMSxdDLY6x
IYEss0a0LE1hc29uIFR1Y2tldHQgKHRlc3Qga2V5KSA8bWFzb25AdHVja2V0dC54
eXo+iJMEExYKADsWIQQHtBsi8IhirGHx215GLgVJjEx/BgUCaBQztAIbAwULCQgH
AgIiAgYVCgkICwIEFgIDAQIeBwIXgAAKCRBGLgVJjEx/BpgOAP0YS5iS0FXf9hL+
yysPOAdcw2itZQKemWnMwxl/4FnapwD+Pzrz9ElopvfmsE2hvqwD+STh77yDvzaw
dIQOXiYDTwa4OARoFDO0EgorBgEEAZdVAQUBAQdAlceTsCOfM+wEp/5xI0JG+ge5
nUhgSQhKgkLipF8AJjEDAQgHiHgEGBYKACAWIQQHtBsi8IhirGHx215GLgVJjEx/
BgUCaBQztAIbDAAKCRBGLgVJjEx/BszKAPoDV+OUtoPQITfgjCOdsjbyL4NXONJx
0i5RdBLmpFD3hwD+MtRJ8tpnG+w2OoGll1Vdl+xzP9o18w92rsSptL3x3ww=
=1K3m
-----END PGP PUBLIC KEY BLOCK-----
```
Once you have exported your public key, it is advisable to make backups of your private key.

```bash
# Export the private key (BACKUP PURPOSES ONLY — NEVER SHARE THIS!!!)
gpg --armor --export-secret-keys 07B41B22F08862AC61F1DB5E462E05498C4C7F06 > privatekey.gpg

# Verify the private key
cat privatekey.gpg
-----BEGIN PGP PRIVATE KEY BLOCK-----

     !!! DO NOT SHARE THIS !!!

-----END PGP PRIVATE KEY BLOCK-----
```
It's best to utilize the fingerprint when exporting keys, especially if you have multiple keys tied to the same email address; *this ensures you are exporting the correct key.*

Naturally, you will be sharing your public key quite frequently—but **NEVER SHARE YOUR PRIVATE KEY!**

Public keys are safe to distribute, but private keys should remain **PRIVATE**.

Private keys are really only functionally used for decryption, and making backups ensures that you have redundancy; only do this when transferring or restoring keys.

### Importing Keys

Typically, you will only import a recipient's public key—but you may also need to import redundant copies of private keys.

```bash
# Public key
gpg --import pubkey.gpg

#Private key
gpg --import privatekey.gpg

# Check signatures
gpg --check-sigs 07B41B22F08862AC61F1DB5E462E05498C4C7F06
ub   ed25519 2025-05-02 [SC] 
      07B41B22F08862AC61F1DB5E462E05498C4C7F06
uid           [ultimate] Mason Tuckett (test key) <mason@tuckett.xyz>
sig!3        462E05498C4C7F06 2025-05-02  [self-signature]
sub   cv25519 2025-05-02 [E]
sig!         462E05498C4C7F06 2025-05-02  [self-signature]

gpg: 2 good signatures

# Trust the key
gpg --edit-key 07B41B22F08862AC61F1DB5E462E05498C4C7F06
Secret key is available.

sec  ed25519/462E05498C4C7F06
     created: 2025-05-02  expires: never       usage: SC  
     trust: unknown       validity: unknown
ssb  cv25519/BCA683EF83020019
     created: 2025-05-02  expires: never       usage: E   
[ unknown] (1). Mason Tuckett (test key) <mason@tuckett.xyz>

gpg> trust
```
_**Make sure to be mindful when discerning your level of trust.**_

> 1 = I don't know or won't say\
> 2 = I do NOT trust\
> 3 = I trust marginally\
> 4 = I trust fully\
> 5 = I trust ultimately


### Deleting Keys

When deleting keys, it is incredibly important to make sure you are deleting the correct keys. 

If you are deleting a key pair, the private key will need to be deleted first.

```bash
#Private key
gpg --delete-secret-key 07B41B22F08862AC61F1DB5E462E05498C4C7F06
sec  ed25519/462E05498C4C7F06 2025-05-02 Mason Tuckett (test key) <mason@tuckett.xyz>

Delete this key from the keyring? (y/N) y
This is a secret key! - really delete? (y/N) y

# Public key
gpg --delete-key 07B41B22F08862AC61F1DB5E462E05498C4C7F06
pub  ed25519/462E05498C4C7F06 2025-05-02 Mason Tuckett (test key) <mason@tuckett.xyz>

Delete this key from the keyring? (y/N) y
```

## Signing with GPG

Signing files or messages is incredibly straightforward—just make sure to use the correct private key.

Detached signatures are preferable as both files are compared against one another, unlike a single _attached_ signature.

```bash
# Detached signature
echo "msg" > msg
gpg --armor --default-key 07B41B22F08862AC61F1DB5E462E05498C4C7F06 --detach-sign msg

# Verify the signature
gpg --verify msg.asc msg
gpg: Signature made Fri 02 May 2025 12:00:19 AM MDT
gpg:                using EDDSA key 07B41B22F08862AC61F1DB5E462E05498C4C7F06
gpg: Good signature from "Mason Tuckett (test key) <mason@tuckett.xyz>" [ultimate]

cat msg.asc
-----BEGIN PGP SIGNATURE-----

iHQEABYKAB0WIQQHtBsi8IhirGHx215GLgVJjEx/BgUCaBRfcwAKCRBGLgVJjEx/
Bg4aAPwJirjRBeacdjKTv0oyha0fGl69ZTR0LqgrLQKLMCy5XQD4jiW5gy8XhpzO
VpcOptcsNNtqfovoNNoE6tFahdZ0Dg==
=Ge5O
-----END PGP SIGNATURE-----
```

## Encrypting with GPG

Files and messages can be encrypted using the recipient's public key. 

By design, no intermediary can decrypt the file or message without the recipient's private key.

Therefore, that is why you must exchange keys beforehand; otherwise, the data would effectively be unrecoverable.

```bash
# Encrypt a file
echo "file" > file
gpg --armor --encrypt --recipient recipient@mail.com file

cat file.asc
-----BEGIN PGP MESSAGE-----

hF4DmGWaJPMsGZ4SAQdAt6v31fJBwhecUZHqOrXfoPjEXWY8yjXeMV0GIuPAHSYw
SHf4TJGAcuGF7t35EsRD7TeIPRH7v/AniwonsjXJJwyHAaLEMtq/KU/u3D63j3F2
1EwBCQIQjuprIiTotwqwJYjpKBrQhC8DyC1DTZLZwlM2zmB59TAQeyuPviR8KMcy
4JoxPuR3tsOUOQAnvDZjojtFjtkiWC4a3fqMWU85
=Th91
-----END PGP MESSAGE-----

# Decrypt the file (recipient's private key)
gpg --decrypt file.asc
gpg: encrypted with cv25519 key, ID 98659A24F32C199E, created 2025-05-02
      "recipient (test) <recipient@mail.com>"
file
```

## Closing 

GPG is an essential tool for anyone looking to ensure both integrity and authenticity by adding a secure layer to digital communication.

GPG or OpenPGP tools are useful for establishing chains of trust and for preventing man-in-the-middle attacks—limiting access to sensitive information.

It is a fantastic idea to communicate via GPG-encrypted messages or files, which many privacy-conscious providers like [Proton Mail](https://proton.me/support/how-to-use-pgp) natively support.

Remember to always use encryption when necessary, even if it may not appear convenient at first; _you are in control of your data_.

_**Encrypt away!**_




