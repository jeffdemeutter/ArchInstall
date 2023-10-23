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
`nano /boot/loader/entries/arch.conf`
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
<details><summary><b>KDE plasma</b></summary>
  
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
---
</details>

<details><summary><b>DWM</b></summary>

### Installing DWM

Mandatory packages  
`sudo paru -S xorg-server xorg-xinit xorg-xrandr xorg-xsetroot picom dmenu`


Copy the xinit file to your home home directory  
`cp /etc/X11/xinit/xinitrc ~/.xinitrc`

Edit xinit file for startup  
`nano ~/.xinitrc`

  Remove following lines
```
twm &
xclock -geometry 50x50-1+1 &
xterm -geometry 80x50+494+51 &
xterm -geometry 80x20+494-0 &
exec xterm -geometry 80x66+0+0 -name login
```
  Add following lines
```
# keyboard layout
setxkbmap en &

# Display resolution
xrandr --output Virtual-1 --mode 1920x1080 &

# Compositor
picom -f &

# execute DWM
exec dwm
```
`startx` to enter dwm  

### Automaticly call startx
`nano ~/.bash_profile`  
Add following code at the end of the file 
```
if [ -z "${DISPLAY}" ] && [ "${XDG_VTNR}" -eq 1 ]; then
  exec startx
fi
```

### Enable Wifi support
`paru -s networkmanager-dmenu-bluetoothfix-git`

open `networkmanager_dmenu` using dmenu

### Enable Sound
`paru pulseaudio`
`paru pulseaudio-alsa`
`paru pamixer`

Add pulseaudio to xinit  
`nano ~/.xinitrc`  

Add `pulseaudio --start &` before `exec dwm`

You can change/unmute volume through pamixer command
Or you can patch DWM to use a hotkey patch

### Enable numlock on start
`paru numlockx`  
`nano ~/.xinitrc`

Add `numlockx &` before `exec dwm`
</details>
