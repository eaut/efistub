# EFISTUB
## Description

A script to manage UEFI boot configurations for Linux EFISTUB kernels. By using "efistub"
the management of plain EFISTUB boot configurations is greatly simplified. The config file
syntax is similar to systemd-boot.

The management of all aspects of UEFI secure boot configurations is directly supported.
Generated signed boot images contain the Linux kernel as well as the thereafter used
initial ramdisks in one file to ensure the verfication of the entire intial boot process.

Are you using a boot loader like grub or systemd-boot simply because you consider
using plain EFISTUB too cumbersome? Would you like to use secure boot, but hesitated so
far because the setup is complicated? Then efistub should solve your problem.

Key features
  - configuration based UEFI boot menu entry creation
  - automated creation of signed secure boot EFI files
  - easy management of personal secure boot keys
  - short and simple bash script does it all

## Usage

The script efistub has subcommands. They are "bootctl", "keys", "uefi".
```
Usage: efistub command [ARGS]
```

### BOOT MANAGEMENT COMMANDS

```
bootctl install [<config-file>]
    Install all boot configurations

bootctl update [<config-file>]
    Update all boot images

bootctl rm-entry <title>
    Manually remove UEFI boot menu entry with the name <title>
```

### KEY MANAGEMENT COMMANDS

```
keys create [more]
    Create personal UEFI secure boot keys (PK,KEK,DB)

    The optional argument "more" converts personal keys to '.esl' format
    for use with KeyTool and '.cer' format for use with many built-in
    UEFI key managers

keys install
    install secure boot keys into UEFI databases (DB,KEK)

keys activate [usermode|setupmode]
    Usermode: activate usermode by installing the personal PK key
    Setupmode: activate setupmode by removing the personal PK key
```

### UEFI COMMANDS

```
uefi status
    show current secure boot status

uefi boot2setup
    start UEFI setup after next boot
```

## Installation

Create your own package with `cd arch-pkg ; makepkg`

Install it with `pacman -U efistub-git`

## Setting up efistub configurations

All example configurations assume your linux kernel files are located in `/boot`
and your EFI system partition is mounted in `/boot/efi` if not mentioned otherwise.
They are located in `/usr/share/doc/efistub/config-examples`

### Basic boot example

For a standard boot configuration all you need to add is the following:

```
#/etc/efistub/config.d/10_arch.conf
#
# Don't forget to insert your specific UUIDs!
#
TITLE="ArchLinux"
ESPDIR="/boot/efi/EFI/arch"
KERNEL="/boot/vmlinuz-linux"
INITRD="/boot/intel-ucode.img /boot/initramfs-linux.img"
OPTIONS="resume=UUID=<your-swap-uuid> root=UUID=<your-rootfs-uuid> ro quiet splash"
```
Now you can install this configuration by executing
```
efistub bootctl install
```
To verify the successful installation list all UEFI boot entries with
```
efibootmgr -v
```
and check that the files `vmlinuz-linux, intel-ucode.img and initramfs-linux.img`
reside on the ESP with
```
ls -l /boot/efi/EFI/arch
```
### Secure boot example

To add a secure boot configuration you need the following:
```
#/etc/efistub/config.d/20_arch-signed.conf
#
# Don't forget to insert your specific UUIDs!
#
TITLE="ArchSec"
EFISIGNED="/boot/efi/EFI/arch/linux-boot-signed.efi"
KERNEL="/boot/vmlinuz-linux"
INITRD="/boot/intel-ucode.img /boot/initramfs-linux.img"
OPTIONS="quiet splash resume=UUID=<your-swapfs-uuid> root=UUID=<your-rootfs-uuid> ro"
```
This setup requires your own secure boot keys. You can generate them with
the following command:
```
efistub keys create
```
Your system must be in UEFI setup mode to load the keys to the UEFI key databases.
Usually you switch to UEFI setup mode by clearing all secure boot keys in your
motherboard setup. On many systems you can insert the keys directly with
```
efistub keys install
```
If that fails you can insert them with KeyTool or the built-in UEFI keymanager.
In this case your need more key formats. You create those files with
```
efistub keys create more
```
The keys are stored in /etc/efistub/keys.
Now you can install this configuration by executing
```
efistub bootctl install
```
To verify the successful installation list all UEFI boot entries with
```
efibootmgr -v
```
and check that the file `linux-boot-signed.efi` resides on the ESP with
```
ls -l /boot/efi/EFI/arch
```
Finally activate secure boot with
```
efistub keys activate usermode
```
Reboot and verify that your system booted in secure boot mode
```
efistub uefi status
```

### Automatic update of boot images when a new initramfs is generated

You manualy update all boot images after a new kernel was installed with
```
efistub bootctl update
```

This can be automated by using the provided systemd files. Just enable them with
```
systemctl enable efistub-update.path
```

## Used directories and configuration files

```
/etc/efistub/config.d/  location of boot configuration files
/etc/efistub/keys/      location of personal secure boot keys
```

Automatic boot image updates

```
/usr/lib/systemd/system/efistub-update.path	    trigger automatic boot file generation
/usr/lib/systemd/system/efistub-update.service  run 'efistub bootctl update'
```

## References

A very good summary of all the information fragments you can find on the Internet
regarding EFISTUB based UEFI secure boot was recently written
by [Matthew Bentley](https://bentley.link/secureboot).

Other:
- [Secure Boot](https://wiki.archlinux.org/index.php/Secure_Boot)
- [EFISTUB](https://wiki.archlinux.org/index.php/EFISTUB)
