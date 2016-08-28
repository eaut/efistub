# EFISTUB
## Description

A script to manage UEFI boot configurations for Linux EFISTUB kernels. By using "efistub" the management of plain EFISTUB boot configurations is greatly simplified. The config file syntax is similar to systemd-boot.

The management of all aspects of UEFI secure boot configurations is directly supported.
Generated signed boot images contain the Linux kernel as well as the thereafter used
intial ramdisks in one file to ensure the verfication of the entire intial boot process.

Are you using a boot loader like grub or systemd-boot simply because you consider
using plain EFISTUB too cumbersome? Would you like to use secure boot, but hesitated so
far because the setup is complicated? Then efistub should solve your problem.

Key features
  - configuration based UEFI boot menu entry creation
  - automated creation of signed secure boot EFI files
  - easy management of personal secure boot keys

## Usage

The script efistub has subcommands. They are "bootctl", "keys", "uefi".
```
Usage: efistub command [ARGS]
```

### BOOT MANAGEMENT

```
bootctl install [<config-file>]
    Install all boot configurations

bootctl update [<config-file>]
    Update all boot images

bootctl rm-entry <title>
    Manually remove UEFI boot menu entry with the name <title>
```

### KEY MANAGEMENT

```
keys create [more]
    Create personal UEFI secure boot keys (PK,KEK,DB)

    The optional argument "more" converts personal keys
    to '.esl' format for use with KeyTool and
    '.cer' format for use with many built-in UEFI key
    managers

keys install
    install secure boot keys into UEFI databases (DB,KEK)

keys switch [usermode|setupmode]
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

## Tool installation

TODO: add installation procedure

## Setting up EFISTUB configurations

All example configurations assume your linux kernel files are located in /boot and your
EFI system partition is mounted in /boot/efi if not mentioned otherwise.

### Basic setup

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
and check that the files vmlinuz-linux, intel-ucode.img and initramfs-linux.img reside on the ESP
with
```
ls -l /boot/efi/EFI/arch
```
### Secure Boot setup

TODO: bla bla bla

### Automatic update of boot images when a new initramfs is generated

You can update all boot images with after a new kernel was installed with
```
efistub bootctl update
```

Full automation can be done using two systemd files.
```
# 1st  - /etc/systemd/system/efistub-update.path
[Unit]
Description=Trigger efistub boot image generation

[Path]
PathChanged=/boot/initramfs-linux-fallback.img

[Install]
WantedBy=multi-user.target
WantedBy=system-update.target
```
```
# 2nd  - /etc/systemd/system/efistub-update.path
[Unit]
Description=Start efistub boot image generation

[Service]
Type=oneshot
ExecStart=/usr/bin/efistub bootctl update
```
## Used directories and configuration files

```
/etc/efistub/config.d/  location of boot configuration files
/etc/efistub/keys/      location of personal secure boot keys
```

Automatic image updates via systemd

```
/etc/system.d/system/efistub-update.path	    trigger automatic boot file generation
/etc/system.d/system/efistub-update.service   run 'efistub update-bootimages' for new kernels
```

## References

A very good summary of all the fragments that you can find regarding EFISTUB only secure boot was recently written
by [Matthew Bentley](https://bentley.link/secureboot).

Other:
https://wiki.archlinux.org/index.php/Secure_Boot
