+++
date = '2025-03-13T00:38:44-07:00'
title = 'Librebooting a Thinkpad T440p'
author = "Mason Tuckett"
description = "A Guide for Installing Libreboot on the Lenovo Thinkpad T440p"
tags = [
    "hardware",
    "libreboot",
    "firmware",
]
hideReply = true
+++
A Guide for Installing Libreboot on the Lenovo Thinkpad T440p

## What is Libreboot?

Libreboot is a free and open-source replacement for proprietary UEFI/BIOS firmware—neutering invasive technologies like [Intel ME](https://itsfoss.com/fact-intel-minix-case/) in the process.

The project seeks to remove proprietary [binary blobs](https://libreboot.org/news/policy.html#what-is-a-binary-blob) included in firmware (required for key components to function)—providing a privacy-respecting alternative for users.

Libreboot is primarily geared towards enthusiasts—though it does have limited hardware support; however, many devices are now compatible.

_Libreboot is a distribution of Coreboot—wherein all updates come from upstream [Coreboot](https://www.coreboot.org/) development._

Unlike Coreboot, Libreboot is designed to be completely free and open-source, including only essential [CPU microcode updates](https://libreboot.org/news/microcode.html).

_**For more details, visit [here](https://libreboot.org/).**_

# Installation


## Tools 
You will need an EEPROM flash programmer.

OR a SOIC/SOP 8 test clip, eight female DuPont wires, and a compatible SBC (such as a Raspberry Pi w GPIO pins).

### SOIC 8 Wiring
![SOIC-8 Wiring](/images/posts/t440p-libreboot/wiring.webp)

### CH341A Wiring
![CH341A Instructions](/images/posts/t440p-libreboot/ch341a.webp)

_**For my installation, I used the CH341A programmer—though, [this is not advised](https://libreboot.org/docs/install/spi.html#do-not-buy-ch341a).**_

## Software

Preferably, you will need a device with Linux installed; if you are using an SBC, install SSH for flashing.


### Obtaining Build Files

```sh
# Prerequisites
sudo apt install -y git wget build-essential

# Obtain Libreboot's build system
git clone https://codeberg.org/libreboot/lbmk ~/lbmk
cd ~/lbmk

# Obtain Libreboot T440p image
wget https://mirror.math.princeton.edu/pub/libreboot/stable/20241206/roms/libreboot-20241206rev10_t440plibremrc_12mb.tar.xz
```
### Making the Libreboot Build System

```sh
# Set available threads for faster compilation
export XBMK_THREADS=12

# Set your distro and build necessary dependencies
sudo ./mk dependencies mint

# Build flashprog
./mk -b flashprog
mv ~/lbmk/elf/flashprog/flashprog ../..
```
### Preparing Images

```sh
# Inject necessary vendor files
./mk inject libreboot-20241206rev10_t440plibremrc_12mb.tar.xz

# Unpack the injected image
tar -xvf libreboot-20241206rev10_t440plibremrc_12mb.tar.xz

# Split the top ROM (4MB)
dd if=~/lbmk/bin/t440plibremrc_12mb/seagrub_t440plibremrc_12mb_libgfxinit_corebootfb_usqwerty.rom of=top.rom bs=1M skip=8

# Split the bottom ROM (8MB)
dd if=~/lbmk/bin/t440plibremrc_12mb/seagrub_t440plibremrc_12mb_libgfxinit_corebootfb_usqwerty.rom of=bot.rom bs=1M count=8
```
### Disassembling the Laptop

Refer to [this guide](https://www.myfixguide.com/manual/lenovo-thinkpad-t440-disassembly/) for disassembling the laptop; make sure both the main battery and CMOS battery are unplugged. 

![T440p top (4MB) and bottom (8MB) ROM locations](/images/posts/t440p-libreboot/t440p-chips.webp)

### Backing Up Stock ROMs

![CH341A reading ROMs](/images/posts/t440p-libreboot/connected.webp)

_**Your setup will look different; my casing was fused together, so I had to improvise.**_

```sh
# Backup top stock ROMs
sudo ./flashprog --programmer ch341a_spi -c W25Q32FV -r top-stock-bk.rom
sudo ./flashprog --programmer ch341a_spi -c W25Q32FV -r top-stock-bk2.rom

# Backup bottom stock ROMs
sudo ./flashprog --programmer ch341a_spi -c W25Q64BV/W25Q64CV/W25Q64FV -r bot-stock-bk.rom
sudo ./flashprog --programmer ch341a_spi -c W25Q64BV/W25Q64CV/W25Q64FV -r bot-stock-bk2.rom

# Verify ROMs were dumped correctly
sha512sum top-stock* bot-stock*
```
### Flashing Libreboot Images

```sh
# Write the top ROM
sudo ./flashprog --programmer ch341a_spi -c W25Q32FV -w ~/lbmk/top.rom

# Write the bottom ROM
sudo ./flashprog --programmer ch341a_spi -c W25Q64BV/W25Q64CV/W25Q64FV -w ~/lbmk/bot.rom
```
_**Reassemble the unit—and voilà! Your laptop now sports Libreboot!**_

![Libreboot successfully Installed - (Lenovo Thinkpad T440p)](/images/posts/t440p-libreboot/libreboot-installed.webp)
