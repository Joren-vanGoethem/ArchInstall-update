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
Look for the model of disk you want to install the OS on
Write the the disk down. Common names used are /dev/sdX or /dev/nvme0nX. Where X should be replaced with the disk letter

Example used: /dev/nvme0n1. Replace with corresponding disk

```sh
gdisk /dev/nvme0n1

# the following commands are ran inside gdisk
# delete existing partitions with, run the d command until you have no partitions left (unless you want to dual boot, don't delete the windows partitions)
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
=> +32G # take the size of your RAM or half of it
=> default

# partition for our Linux system
n
=> default = 3
=> default
=> default
=> default

# Write changes --IMPORTANT--
w
```

assuming your disk name is /dev/nvme0n1
Format Partitions:
```bash
mkfs.fat -F 32 /dev/nvme0n1p1

mkswap /dev/nvme0n1p2

mkfs.ext4 /dev/nvme0n1p3
```

Mount partitions
```bash
mount /dev/nvme0n1p3 /mnt
mkdir /mnt/boot
mount /dev/nvme0n1p1 /mnt/boot

swapon /dev/nvme0n1p2
```

install nano
```bash
pacman -S nano
```

Before we can go onto installing our system we'll enable some things that'll make our downloads faster.
```bash
nano /etc/pacman.conf

ParallelDownloads=20
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

## Inside our chroot

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
nl_BE.UTF-8
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

### Setup Users
Set our root password:
```
passwd
```

Create a normal user DO NOT FORGET -m :
```bash
useradd -m username
```

Set the password for our new user
```bash
passwd username
```

# Installing boot loader
## bootctl
```bash
bootctl install
```

### add Arch entry

add a new file to `/boot/loader/entries` named `arch.conf` and add the following content:
```bash
title   Arch Linux
linux   /vmlinuz-linux
initrd  /initramfs-linux.img
options /root=/dev/nvme0n1p3 #this can be different depending on your specific install
```

### add Windows entry for dual boot
install edk2-shell and copy the file to your boot partition
```bash
sudo pacman -S edk2-shell
sudo cp /usr/share/edk2-shell/x64/Shell.efi /boot/shellx64.efi
```

get the PARTUUID for your windows-esp partition
```bash
sudo blkid | grep vfat
```

Usually, the Windows EFI Partiton is labelled “EFI system partition”, you should get a line that looks like that:
```bash
/dev/nvme1n1p2: UUID="52CC-E135" BLOCK_SIZE="512" TYPE="vfat" PARTLABEL="EFI system partition" PARTUUID="e2cc5bf0-9654-4ba3-bdc7-cdb1c2db2c3b"
```

set console-mode to max in loader config
```bash
sudo nano /boot/loader/loader.conf

> add the following line to the file
console-mode max
```

reboot and choose EFI shell in the loader screen which gets autocreated if you copied the `shellx64.efi` file correctly.
it should display a list of FS aliases followed by that fs alias' partition details. if it does not enter the command map.

take note of the FS alias that contains the `PARTUUID` you got from the windows partition. this can look something like HD0b or HD2c but can also be longer.

enter the exit command and reboot into your linux installation.

create a file in your boot partition called `windows.nsh` and add the following with your correct fs alias:
```bash
HD2c:EFI\Microsoft\Boot\Bootmgfw.efi
```

now create a loader entry for that windows installation by creating a file windows.conf under `/boot/loader/entries` with the following content:

```bash
title   Windows
efi     /shellx64.efi
options -nointerrupt -noconsolein -noconsoleout windows.nsh
```

### installing necessary tools
installing networkmanager and enabling the service for network access after install
```bash
pacman -S networkmanager
systemctl enable NetworkManager
```

Setup Sudo
```bash
pacman -S sudo
groupadd sudo
usermod -aG sudo username
nano /etc/sudoers
# => uncomment users in sudo group can use sudo
```

### Installing KDE plasma
`important` use plasma-desktop if you want a basic set of utilities like text editor, calculator, file explorer,....
```bash
pacman -S xorg-server plasma-meta
```

Enabling SDDM
```bash
systemctl enable sddm
```

### Install a Terminal and file explorer
you can use others, we use these and they work great
flatpak is for software installation using discover
```bash
pacman -S konsole dolphin flatpak
```

### Exit Chroot and Reboot

```bash
exit
reboot
```

# update in case of incorrect GPG keys
```bash
rm -rf /etc/pacman.d/gnupg/

pacman-key --init
pacman-key --populate archlinux
pacman-key --refresh-keys

pacman -S archlinux-keyring
```

# installing a font containing emojis

run the fonts.sh script in this repository with root priveleges
