---
title: "Secure boot on Arch Linux"
subtitle: "Again an history of something that stopped working at 10pm on a Friday."
summary: "10pm. Your PC doesn't boot anymore after an upgrade and the source of the problem seems to be the secure boot. Let's clean this up and get the PC up and running."
date: 2023-08-13T22:00:00+02:00
authors:
  - nanoy
tags:
  - Computer
image:
  caption: "The blue screen of death of Windows, for illustration purposes only. I don't use Windows."
---

Until recently I had set up my secure boot on my computer with an Arch linux system following this [blog post](https://bentley.link/secureboot/) from Matthew Bentley. However, one Friday night after 10pm, I did an update, including the linux kernel, the script automatically signed the new image, and after a reboot it stopped working. I am not sure what changed and I didn't investigated very long but it seems that the creation of the unified image was not working, as even the unsigned unified version was not booting with secure boot disabled.

It was time for a new solution and it is described below.

I strongly recommend reading [the original blog post](https://bentley.link/secureboot/) which was really helpful to me and the [Arch documentation page on secure boot](https://wiki.archlinux.org/title/Unified_Extensible_Firmware_Interface/Secure_Boot).

## What is secure boot ?

Secure boot is an option that prevents booting images that have not been signed with the required keys. In practice the BIOS is provided with some public certificates and the boot images are signed with the secret key which can prevent booting unauthorized software.

In this tutorial we are interested in enabling (or repairing) the secure boot on a linux computer.

Note that I will do oversimplification (mainly because I am lazy) and this tutorial will be specific to my case, and I cannot ensure that it will work for you. In particular, I am running Arch linux on a Thinkpad T470s, and my EFI System Partition (ESP) is mounted in `/boot`. My bootloader is `systemd-boot`.

## Creating keys

This is mainly taken from [Matthew Bentley's original blog post](https://bentley.link/secureboot/) as this is how I created the required files a while ago and this is included for reference. Some of the original blog post also seems to come from [here](http://www.rodsbooks.com/efi-bootloaders/controlling-sb.html#creatingkeys).

The following script can be used to generate the keys and the certificates:

```sh
#!/bin/bash

echo -n "Enter a Common Name to embed in the keys: "
read NAME

openssl req -new -x509 -newkey rsa:2048 -subj "/CN=$NAME PK/" -keyout PK.key \
        -out PK.crt -days 3650 -nodes -sha256
openssl req -new -x509 -newkey rsa:2048 -subj "/CN=$NAME KEK/" -keyout KEK.key \
        -out KEK.crt -days 3650 -nodes -sha256
openssl req -new -x509 -newkey rsa:2048 -subj "/CN=$NAME DB/" -keyout DB.key \
        -out DB.crt -days 3650 -nodes -sha256
openssl x509 -in PK.crt -out PK.cer -outform DER
openssl x509 -in KEK.crt -out KEK.cer -outform DER
openssl x509 -in DB.crt -out DB.cer -outform DER

GUID=`python2 -c 'import uuid; print str(uuid.uuid1())'`
echo $GUID > myGUID.txt

cert-to-efi-sig-list -g $GUID PK.crt PK.esl
cert-to-efi-sig-list -g $GUID KEK.crt KEK.esl
cert-to-efi-sig-list -g $GUID DB.crt DB.esl
rm -f noPK.esl
touch noPK.esl

sign-efi-sig-list -t "$(date --date='1 second' +'%Y-%m-%d %H:%M:%S')" \
                  -k PK.key -c PK.crt PK PK.esl PK.auth
sign-efi-sig-list -t "$(date --date='1 second' +'%Y-%m-%d %H:%M:%S')" \
                  -k PK.key -c PK.crt PK noPK.esl noPK.auth

chmod 0600 *.key

echo ""
echo ""
echo "For use with KeyTool, copy the *.auth and *.esl files to a FAT USB"
echo "flash drive or to your EFI System Partition (ESP)."
echo "For use with most UEFIs' built-in key managers, copy the *.cer files."
echo ""
```

It is recommend to do this on a safe location on your PC (`/root/keys`) for instance. `sbsigntools` and `efitools` are required.

After running the script, there are several keys and certificates:

```
$ ls /root/keys
DB.cer  DB.crt  DB.esl  DB.key  KEK.cer  KEK.crt  KEK.esl  KEK.key  mkkeys.sh  myGUID.txt  noPK.auth  noPK.esl  PK.auth  PK.cer  PK.crt  PK.esl  PK.key
```

Then the keys need to be installed. To do this, copy all the `.cer`, `esl` and `.auth` to `/boot` (or another FAT32 medium that will be accessible at boot time). Then reboot your PC, go to the BIOS, and reset the secure boot in setup mode.

Once on your system again, copy `/usr/share/efitools/efi/KeyTool.efi` to `/boot` and add a temporary boot entry in `/boot/loader/entries/keytool.conf`:

```
title  KeyTool
efi    /KeyTool.efi
```

Now boot in the KeyTool and replace the keys. Select "Edit keys" and delete the db and KEK keys. After this, add the db key: `db > Add new key > DB.esl`, the KEK key: `KEK > Add new key > KEK.esl` and the PK: `The Platform Key (PK) > Replace Key(s) > PK.auth`.

Again I recommend reading Matthew's blog post, this section is better explain (also I took some shortcuts).

## Creating a Unified Kernel Image (UKI)

One of the main difference when using the secure boot is that we need to create a unified kernel image and sign this image. This image contains the kernel, initrd and eventually other information such as the start command line or the name of the os release.

After a bit of search, including on the [Arch documentation page on secure boot](https://wiki.archlinux.org/title/Unified_Extensible_Firmware_Interface/Secure_Boot), I found the ukify utility script that was doing exactly what I wanted as a replacement of the objcopy utility to create the image. On Arch linux, the ukify utility can be installed by installing the `systemd-ukify` package.

```
sudo pacman -Syu systemd-ukify
```


After reading the documentation of ukify, the equivalent of the objcopy line propose in [Matthew Bentley's tutorial](https://bentley.link/secureboot/)

```
objcopy \
    --add-section .osrel=/etc/os-release --change-section-vma .osrel=0x20000 \
    --add-section .cmdline="cmdline.txt" --change-section-vma .cmdline=0x30000 \
    --add-section .linux="/boot/vmlinuz-linux" --change-section-vma .linux=0x40000 \
    --add-section .initrd="/boot/initramfs-linux.img" --change-section-vma .initrd=0x3000000 \
    /usr/lib/systemd/boot/efi/linuxx64.efi.stub kernel.efi
```

can now be simply written as 

```
/usr/lib/systemd/ukify build \
    --linux=/boot/vmlinuz-linux \
    --initrd=/boot/initramfs-linux.img \
    --cmdline=@/etc/cmdline \
    --os-release=@/etc/os-release \
    --output=kernel.efi
```

Note that for the `cmdline` and the `os-release` arguments, the `@` character is very important since it discriminates between given directly the value or the file from where the value should be read.

One slight difference, if you are using another initrd (such as intel microcode), you should specify it by using several time the `--initrd` option:

```
/usr/lib/systemd/ukify build \
    --linux=/boot/vmlinuz-linux \
    --initrd=/boot/intel-ucode.img \
    --initrd=/boot/initramfs-linux.img \
    --cmdline=@/etc/cmdline \
    --os-release=@/etc/os-release \
    --output=kernel.efi
```

This image could now be signed with the same command line as in [Matthew Bentley's blog post](https://bentley.link/secureboot/) using the `sbsign` utility.

```
sbsign --key /root/keys/DB.key --cert /root/keys/DB.crt --output kernel.efi kernel.efi
```

but it's also possible to directly tell ukify to sign the image.

## Creating a signed UKI

Again reading the documentation of ukify, it's possible to create the command line that will in the same time crate the unified image and sign it:

```
/usr/lib/systemd/ukify build \
    --linux=/boot/vmlinuz-linux \
    --initrd=/boot/intel-ucode.img \
    --initrd=/boot/initramfs-linux.img \
    --cmdline=@/etc/cmdline \
    --os-release=@/etc/os-release \
    --secureboot-private-key=/root/keys/DB.key \
    --secureboot-certificate=/root/keys/DB.crt \
    --output=kernel.efi
```

Note that by default, `sbsign` is used as the signer no we don't need to set the --signtool option.

## Automation

Now let's automate it ! The goal here is to configure a pacman hook so that every time the linux kernel is updated, a new signed image is automatically created. It would have been possible to configure ukify using the configuration file but for simplicity I will just put the whole command line in a script, in `/root/secure-boot/make-signed-image.sh`

```bash
#!/bin/bash

BOOTDIR=/boot
CERTDIR=/root/keys
KERNEL=/boot/vmlinuz-linux
INITRDLINUX=/boot/initramfs-linux.img
INITRDUCODE=/boot/intel-ucode.img
OUTIMG=/boot/kernel.efi
CMDLINE=/etc/cmdline
OSRELEASE=/etc/os-release

/usr/lib/systemd/ukify build \
    --linux=${KERNEL} \
    --initrd=${INITRDUCODE} \
    --initrd=${INITRDLINUX} \
    --cmdline=@${CMDLINE} \
    --os-release=@${OSRELEASE} \
    --secureboot-private-key=${CERTDIR}/DB.key \
    --secureboot-certificate=${CERTDIR}/DB.crt \
    --output=${OUTIMG}
```

Finally, set the pacman hook in `/etc/pacman.d/hooks/secure-boot.hook`:

```ini
[Trigger]
Operation = Install
Operation = Upgrade
Type = Package
Target = linux

[Action]
When = PostTransaction
Exec = /bin/sh -c 'while read -r f; do /root/secure-boot/make-signed-image.sh; done'
NeedsTargets
```

Now, each time the linux kernel is updated, the script `make-signed-image.sh` will be run after the update to create the new signed unified image.

For reference, here is the configuration for the entry in systemd-boot (`/boot/loader/entries/arch.conf`)

```
/boot/loader/entries/arch.conf
title	Arch Linux
efi	/kernel.efi
```