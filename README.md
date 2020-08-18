# Instalare arch

## Verify the boot

The easiest way to find out if you are running UEFI or BIOS is to look  for a folder /sys/firmware/efi. The folder will be missing if your  system is using BIOS. 

```bash
ls /sys/firmware/efi/efivars
```

If the command shows the directory without error, then the system is  booted in UEFI mode. If the directory does not exist, the system may be  booted in BIOS mode

## Connect to the internet

Ensure your network interface is listed and enabled:

```bash
ip link
```

For wireless, enter iwctl prompt:

```bash
[iwd]# station wlan0 scan
[iwd]# station wlan0 get-networks
[iwd]# station wlan0 connect SSID
[iwd]# exit
ping google.com
```

## Update the system clock

```bash
timedatectl set-ntp true
```

## Partition the disks

Use this command to list all the disk and partitions on your system:

```bash
fdisk -l
```

Your hard disk should be labelled /dev/sda or /dev/nvme0n1. Please use the appropriate disk labeling for your system. First, select the disk you are going to format and partition:

```
fdisk /dev/sda
```

I suggest that you delete any existing partitions on the disk using command **d**. Once you have the entire disk space free, it’s time to create new partitions with command **n**.

### Create an ESP partition (For UEFI systems only)

**If you have a UEFI system**, you **must** create an EFI partition at the beginning of your disk. Otherwise, skip this step. 

When you enter **n**, it will ask you to choose a disk number, enter 1. Stay with the default block size, when it asks for the partition size, enter +512M.

One important steps is to change the type of the EFI partition to EFI System (instead of Linux system). Enter **t** to change type. Enter L to see all the partition types available and then enter its corresponding number to the EFI system.

### Create root partition for both UEFI and legacy systems

The common partitioning practice was/is to create root, swap and home partitions separately. While you are in the fdisk command, press n to create a new partition. It will automatically give it partition number 2. When you are done with the disk partitioning, enter **w** command to write the changes to the disk and exit out of fdisk command.

> ​	de pus poze

## Create filesystems

Now that you have your disk partitions ready, it’s time to create filesystem on it. Follow the steps for your system:

### Creating filesystem for UEFI system

So, you have two disk partitions and the first one is EFI type. Create a FAT32 file system on it using the mkfs command:

```
mkfs.fat -F32 /dev/sda1
```

Now create an Ext4 filesystem on the root partition:

```
mkfs.ext4 /dev/sda2
```

### Creating filesystem for non-UEFI system

For non-UEFI system, you only have one single root partition. So just make it ext4:

```
mkfs.ext4 /dev/sda1
```

And make and activate a swap partition:

```
mkswap /dev/swap_partition
swapon /dev/swap_partition
```

## Mount the file systems

```bash
mount /dev/sda1 /mnt
```

## Installation

Synchronize package databases

```bash
pacman -Syy
```

Now, install reflector too that you can use to list the fresh and fast mirrors located in your country:

```bash
pacman -S reflector
```

Now, get the good mirror list with reflector and save it to mirrorlist.

```bash
reflector -c "RO" -f 12 -l 10 -n 12 --save  etc/pacman.d/mirrorlist
```

Use the pacstrap script to install the base package, Linux kernel and firmware for common hardware:

```bash
pacstrap /mnt base linux linux-firmware nano mc
```

## Configure the system

### chroot

Change root to new system:

```bash
arch-chroot /mnt
```

### timezone

Set the time zone:

```bash
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
```

Run hwclock to generate /etc/adjtime:

```bash
hwclock --systohc
```

### locale

Edit /etc/locale.gen and uncomment en_US.UTF-8 UTF-8 and other needed locales. Generate the locales by running:

```bash
locale-gen
```

```bash
echo LANG=en_US.UTF-8 > /etc/locale.conf
export LANG=en_US.UTF-8
```

### Network configuration

Create a /etc/hostname file and add the hostname entry:

```bash
echo myarch > /etc/hostname
```

Create hosts file:

```bash
touch /etc/hosts
```

And edit this /etc/hosts file with Nano editor to add the following lines to it (replace myarch with hostname you chose earlier):

```
127.0.0.1	localhost
::1		localhost
127.0.1.1	myarch
```

### Set up root passwd

You should also set the password for the root account using the passwd command:

```bash
passwd
```

## Install grub

### Install grub on UEFI systems

Make sure that you are still using arch-chroot. Install required packages:

```
pacman -S grub efibootmgr
```

Create the directory where EFI partition will be mounted:

```
mkdir /boot/efi
```

Now, mount the ESP partition you had created

```
mount /dev/sda1 /boot/efi
```

Install grub like this:

```
grub-install --target=x86_64-efi --bootloader-id=GRUB --efi-directory=/boot/efi
```

And last step:

```
grub-mkconfig -o /boot/grub/grub.cfg
```

### Install grub on Non-UEFI systems

Install grub package first:

```bash
pacman -S grub
```

And then install grub like this (don’t put the disk number sda1, just the disk name sda):

```
grub-install /dev/sda
```


And last step:

```
grub-mkconfig -o /boot/grub/grub.cfg
```

## Generate fstab

Create any remaining mount points, mount the rest of devices and generate fstab:

```bash
mount /dev/sda3 /mnt/home
genfstab -U /mnt >> /mnt/etc/fstab
```

Modify new generated fstab file according your preferences like change noatime on sdaXX:

```bash
mcedit /mnt/etc/fstab
```

## Install a network manager

```
pacman -S networkmanager
systemctl enable NetworkManager.service
```

## Restart

```
exit
reboot
```

## Activate a connection

Activate a connection using network manager ncursed interface:

```
nmtui
```

## Create a sudo-user

```
useradd -m dan
passwd dan
pacman -S sudo
```

edit sudoers file for user privilege specification:

```
mcedit /etc/sudoers
```

```
##
root ALL=(ALL) ALL
dan ALL=(ALL) ALL
##
```

## Install a desktop environment

```
pacman -S xorg
pacman -S cinnamon
pacman -S firefox
pacman -S lightdm lightdm-gtk-greeter
systemctl enable lightdm.service
```

then reboot

## Create user directories



```
pacman -S xdg-user-dirs
xdg-user-dirs-update
```

## Install pamac

Install devel libs:

```
sudo pacman -Syyu base-devel git
```

Clone the Pamac PKGBUILD and build dependencies into a newly-created temporary folder under your home directory:

```
mkdir test
cd test
git clone https://aur.archlinux.org/pamac-aur.git
cd pamac-aur/
makepkg -sic
```

## Install xapps

xed, xviewer, xreader, mintlocale
cinnamon applets: weather, system monitor (install libgtop + restart)
mint-themes, mint-y-icons, gnome-system-monitor, gnome-calculator, gnome-screenshot, lightdm-gtk-greeter-settings, nemo-fileroller,
doublecmd-gtk2, ttf-ms-fonts, viewnior, gnome-usage, gnome-disk, gparted, transmission, opera, opera-ffmpeg-codecs
catfish

## Install microcode for intel procs

```
sudo pacman -S intel-ucode
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

## Disable wifi poweroff on laptop lid close

Modify /etc/NetworkManager/conf.d/default-wifi-powersave-on.conf:

```
[connection]
wifi.powersave = 2
```

## Speed up your Intel wireless chipset

```
echo "options iwlwifi 11n_disable=8" | sudo tee /etc/modprobe.d/iwlwifi-speed.conf
```

## Install a scanner and printer

```
pacman -S  xsane simple-scan hplip cups cups-pdf
systemctl enable org.cups.cupsd.service
```

## Speed up makepkg

To skip compression, you'll want to make the following change to your /etc/makepkg.conf:

```
[...]
#PKGEXT='.pkg.tar.xz'
PKGEXT='.pkg.tar'
[...]
```

And:

```
#-- Make Flags: change this for DistCC/SMP systems
MAKEFLAGS="-j8"
```

## Office suite

libreoffice-fresh, libreoffice-fresh-ro, hunspell, hunspell-ro
