# Arch Install

## Pre-chroot

Update system clock:
```sh
timedatectl set-ntp true
```

Setup partitioning:
```sh
fdisk -l
```
Look for the model of disk you want to install the OS on, remember that disk. Common names used are /dev/sdX or /dev/nvme0nX. Where X should be replaced with the disk letter

Example used: /dev/nvme0n1. Replace with corresponding disk

```sh
gdisk /dev/nvme0n1

# the following commands are ran inside fdisk
# delete existing partitions with, run the d command until you have no partitions left
d
=> 1,2,3,...

# create new partition for EFI
n
=> default = 1
=> default
=> +1G
=> EF00

# create a swap partition
n
=> default = 2
=> default
=> +16G # take the size of your RAM or half of it
=> default

# partition for our Linux system
n
=> default = 3
=> default
=> default
=> default

# Write changes  --IMPORTANT--
w
```

Assuming your disk name is /dev/nvme0n1  
Format Partitions:
```bash
mkfs.ext4 /dev/nvme0n1p3

mkfs.fat -F 32 /dev/nvme0n1p1

mkswap /dev/nvme0n1p2
```

Mount partitions
```bash
mount /dev/nvme0n1p3 /mnt

mkdir /mnt/boot
mount /dev/nvme0n1p1 /mnt/boot

swapon /dev/nvme0n1p2
```

Install nano
```sh
pacman -S nano
```

Before we can go onto installing our system we'll enable some things that'll make our downloads faster.
```bash
nano /etc/pacman.conf

# uncomment and change to a higher number
ParallelDownloads = 20
```

Install base system and kernel
```bash
pacstrap /mnt base linux linux-firmware base-devel
```

Generate fstab for our mounted filesystems
```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

Enter our very basic install:
```bash
arch-chroot /mnt
```

## Inside chroot

Set our timezone
```bash
ln -sf /usr/share/zoneinfo/Europe/Brussels /etc/localtime
```

Sync hardware clock
```bash
hwclock --systohc
```

### Setting Locale

Localization:
```bash
nano /etc/locale.gen

# uncomment accordingly
en_US.UTF-8
```

```bash
nano /etc/locale.conf

LANG=en_US.UTF-8
```

### Hostname

Set hostname:
```bash
nano /etc/hostname

sapphire
```

Set our root password:
```
passwd
```

### Setup Users
Create a normal user DO NOT FORGET `-m`:
```bash
useradd -m username
```

Set the password for our new user
```bash
passwd username
```

## Installing boot loader
### bootctl
```bash
bootctl install
```
### add Arch entry  
`nano /boot/loadeer/entries/arch.conf`
```sh
title   Arch Linux
linux   /vmlinuz-linux
initrd  /initramfs-linux.img
options root=/dev/nvme0n1p3 #this can be different depending on your specific install
```

### networkmanager
```sh
pacman -S networkManager
systemctl enable NetworkManager
```
### sudo
```sh
pacman -S sudo
groupadd sudo
usermod -aG sudo sapphire
nano /etc/sudoers
# => uncomment users in sudo group can use sudo
```

### Exit Chroot and Reboot

```bash
exit

reboot
```

## Installing A Desktop Environment
### Installing KDE plasma
`important` use plasma-desktop if you want a basic set of utilities like text editor, calculator, file explorer,....
```bash
pacman -S xorg-server plasma-meta
```

Enabling SDDM and NetworkManager (not sure if required but it doesn't hurt to do it anyways)
```bash
systemctl enable sddm NetworkManager
```

### Install a Terminal and file explorer
you can use others, we use these and they work great
flatpak is for software installation using discover
```bash
pacman -S konsole dolphin flatpak
```

