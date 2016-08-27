# EFISTUB
## Description

A collection of scripts to manage UEFI boot configurations for Linux EFISTUB kernels.
By using "efistub" the management of plain EFISTUB boot configurations is greatly
simplified. The config file syntax is similar to systemd-boot.

The management of all aspects of UEFI secure boot configurations is directly supported.
Generated signed boot images contain the Linux kernel as well as the thereafter used
intial ramdisks in one file to ensure the verfication of the entire intial boot process.

Are you still using a boot loader like grub or systemd-boot simply because you consider
using plain EFISTUB too cumbersome? Would you like to use secure boot, but hesitated so
far because the setup is complicated? Then efistub should solve your problem.

Key features
  - configuration based UEFI boot menu entry creation
  - automated creation of signed secure boot EFI files
  - easy management of personal secure boot keys

## Usage

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
      Remove UEFI boot menu entry with the name <title>
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

...
  uefi status
      show current secure boot status

  uefi boot2setup
      start UEFI setup after next boot
...

## Tool installation

more to come...

## Setting up EFISTUB configurations

All example configurations assume your linux kernel files are located in /boot and your
EFI system partition is mounted in /boot/efi if not mentioned otherwise.

### Basic setup

For a standard boot configuration all you need to add is the following:

...
#/etc/efistub/config.d/10_arch.conf
#
# Don't forget to insert your specific UUIDs!
#
TITLE="ArchLinux"
ESPDIR="/boot/efi/EFI/arch"
KERNEL="/boot/vmlinuz-linux"
INITRD="/boot/intel-ucode.img /boot/initramfs-linux.img"
OPTIONS="resume=UUID=<your-swap-uuid> root=UUID=<your-rootfs-uuid> ro quiet splash"
...

Now you can install this configuration by executing

...
efistub bootcfg-install
...

You can verify the successful installation with the following commands

...
# show boot menu entry
efibootmgr -v
# the files vmlinuz-linux, intel-ucode.img and initramfs-linux.img should reside on the ESP
ls -l /boot/efi/EFI/arch
...

### Secure Boot setup

bla bla bla

### Automatic update of boot images when a new initramfs is generated

bla bla bla

## Used directories and configuration files

...
/etc/efistub/keys/				location of personal secure boot keys
/etc/efistub/config.d/				location of boot config files
# only used for automatic image updates
/etc/system.d/system/efistub-update.path	trigger automatic boot file generation
/etc/system.d/system/efistub-update.service	run 'efistub update-bootimages' for new kernels
...

## Package dependencies on Arch

The following packages have to be installed in advance

## References

https://wiki.ubuntu.com/SecurityTeam/SecureBoot
https://wiki.archlinux.org/index.php/Unified_Extensible_Firmware_Interface
https://wiki.gentoo.org/wiki/Sakaki%27s_EFI_Install_Guide/Configuring_Secure_Boot
