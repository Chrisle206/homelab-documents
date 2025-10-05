# Building a Systemd-Centric Arch Linux Workstation

**Author:** Christopher Le  
**Date:** September 2025

---

## Goal

Build a **stateless**, **fully encrypted**, and **systemd-centric** Arch Linux workstation — designed for homelab experimentation and Site Reliability Engineering practice.

---

## Vision & Philosophy

> “Any problem can be solved just by editing the right text files.”  
> — Mischa Van Den Berg

This project represents my goal of understanding Linux from the ground up by constructing a **minimal, reliable, and automatable** system.  
It follows the same philosophy that underpins modern **DevOps** and **Site Reliability Engineering**.

---

## Objectives

- Systemd-centric base (no NetworkManager)
- Fully encrypted disk (LUKS + LVM)
- Managed networking via `systemd-networkd` and `systemd-resolved`
- Hyprland window manager
- Secure, stateless configuration
- Tmux for session multiplexing
- Auto-locking and idle daemon
- Reproducible setup suitable for homelab and development use

---

## Installation Roadmap

1. Create installation medium
2. Boot into the live ISO
3. Partition the disks
4. Encryption and LVM setup
5. Filesystem mounting and `/etc/fstab` generation
6. Networking setup
7. `mkinitcpio` configuration
8. Bootloader setup (systemd-boot)
9. First boot verification
10. Desktop environment (Hyprland)
11. Post-install housekeeping
12. Security and locking
13. Developer tools (Docker, DevPod)

---

## Creating the Install Medium (Windows Host)

### Download the ISO

Visit the official [Arch Linux download page](https://archlinux.org/download/#download-mirrors) and download the latest `.iso` and `.sig` files. This document is a mini fast arch linux setup, for more details visit the Archlinux wiki site.

```
cd ~\Downloads
Get-FileHash .\archlinux-2025.09.01-x86_64.iso -Algorithm SHA256
```

Compare the hash against the value listed on Arch’s website.

Verify GPG Signature
Install Gpg4win if needed:

```
gpg --version
gpg --auto-key-locate "clear,wkd" --locate-external-key pierre@archlinux.org
gpg --verify .\archlinux-2025.09.01-x86_64.iso.sig .\archlinux-2025.09.01-x86_64.iso
```

Expected output (example):

```
gpg: Good signature from "Pierre Schmitz <pierre@archlinux.org>"
```

Write ISO to USB
Use Rufus to create a bootable USB drive.
Then boot into your BIOS/UEFI and select the USB as your primary boot device.

Booting and Initial Setup
Follow the official Arch Installation Guide for detailed references.

Verify Internet Connection

```
ping archlinux.org
```

If using Wi-Fi:

```
ip link
iwctl
station wlan0 scan
station wlan0 get-networks
station wlan0 connect "your_network_name"
curl google.com
```

Sync the system clock:

```
timedatectl set-ntp true
```

Disk Partitioning
List available disks:

```
lsblk
```

Assume the target disk is /dev/nvme0n1.

⚠️ Note: These steps assume a clean disk. If reusing an old one, format it first to avoid conflicts.

Using fdisk:

```
pgsql

g     # Create new GPT partition table
n     # Partition 1 (+1G) → EFI
t     # Set type to EFI System
n     # Partition 2 (remaining space) → LVM
p     # Print partition table
w     # Write and exit
```

Encryption and LVM Setup
Encrypt the LVM partition:

```
cryptsetup luksFormat /dev/nvme0n1p2
cryptsetup open /dev/nvme0n1p2 cryptlvm
```

Create physical and logical volumes:

```
pvcreate /dev/mapper/cryptlvm
vgcreate archvg /dev/mapper/cryptlvm
lvcreate -L 4G archvg -n swap
lvcreate -L 32G archvg -n root
lvcreate -l 100%FREE archvg -n home
```

Format the filesystems:

```
mkfs.ext4 /dev/archvg/root
mkfs.ext4 /dev/archvg/home
mkswap /dev/archvg/swap
mkfs.fat -F32 /dev/nvme0n1p1
```

Mount everything:

```
mount /dev/archvg/root /mnt
mount --mkdir /dev/nvme0n1p1 /mnt/boot
mount --mkdir /dev/archvg/home /mnt/home
swapon /dev/archvg/swap
```

Verify:

```
lsblk -f
```

Installing the Base System

```
pacstrap -K /mnt base linux linux-firmware vim nano
```

Generate fstab:

```
mkdir -p /mnt/etc
genfstab -U /mnt >> /mnt/etc/fstab
cat /mnt/etc/fstab
```

Note: /mnt/etc must exist before generating fstab, otherwise it will fail.

Base Configuration (chroot)
Enter the environment:

```
arch-chroot /mnt
```

Set Root Password

```
passwd
```

Timezone & Clock

```
ln -sf /usr/share/zoneinfo/America/Los_Angeles /etc/localtime
hwclock --systohc
date
```

Localization

```
vim /etc/locale.gen
# Uncomment: en_US.UTF-8 UTF-8

locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf
```

Hostname and Hosts

```
echo "arch-lab" > /etc/hostname
cat <<EOF >> /etc/hosts
127.0.0.1 localhost
::1 localhost
127.0.1.1 arch-lab.localdomain arch-lab
EOF
```

Essential Packages

```
pacman -Syu vim which sudo man-db man-pages texinfo amd-ucode
```

Initramfs (mkinitcpio)
Edit configuration:

```
vim /etc/mkinitcpio.conf
```

Ensure hooks include encryption and LVM:

```
HOOKS=(base udev autodetect modconf block encrypt lvm2 filesystems keyboard fsck)
```

Then rebuild:

```
mkinitcpio -P
```

User Setup

```
useradd -m -G wheel -s /bin/bash chris
passwd chris
EDITOR=vim visudo
# Uncomment: %wheel ALL=(ALL:ALL) ALL
```

Networking (systemd only)

```
systemctl enable --now systemd-networkd
systemctl enable --now systemd-resolved
ip link
```

Bootloader (systemd-boot)
Install and configure:

```
bootctl install
bootctl status
```

Edit /boot/loader/loader.conf:

```
default arch
timeout 3
console-mode max
editor no
```

Create /boot/loader/entries/arch.conf:

```
title Arch Linux
linux /vmlinuz-linux
initrd /amd-ucode.img
initrd /initramfs-linux.img
options cryptdevice=UUID=<UUID_of_nvme0n1p2>:cryptlvm root=/dev/archvg/root rw
```

Find UUID:

```
blkid /dev/nvme0n1p2
```

First Boot

```
exit
umount -R /mnt
swapoff -a
reboot
```

Remove the installation USB when prompted.

Next Steps
Install Hyprland and configure user environment

Set up idle daemons and lock screen

Sync dotfiles from GitHub

Configure systemd-networkd profiles

Harden SSH, firewall, and journald

Lessons Learned
/mnt/etc/fstab must exist before generating entries.

mkinitcpio must be rebuilt before bootloader installation.

Using systemd-boot simplifies EFI management compared to GRUB.

Wi-Fi setup via iwctl can fail if wlan0 isn’t powered on; check with rfkill list.

References
ArchWiki: Installation Guide

ArchWiki: Dm-crypt

ArchWiki: Systemd-boot

ArchWiki: Hyprland
