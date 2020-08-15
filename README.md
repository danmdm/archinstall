# Instalare arch

## Verify the boot mode

To verify the boot mode, list the efivars directory:

```bash
ls /sys/firmware/efi/efivars
```

## Connect to the internet

Ensure your network interface is listed and enabled:

```bash
ip link
```

Enter iwctl prompt:

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

```bash
fdisk -l 
```

de completat

## Format the partitions

```bash
mkfs.ext4 /dev/sda1
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

```
pacman -S reflector
```

Now, get the good mirror list with reflector and save it to mirrorlist. 

```bash
reflector -c "RO" -f 12 -l 10 -n 12 --save  etc/pacman.d/mirrorlist
```

Use the pacstrap script to install the base package, Linux kernel and firmware for common hardware:

```
pacstrap /mnt base linux linux-firmware nano mc
```

## Configure the system

### fstab

Mount the rest of devices and generate fstab:

```bash
mount /dev/sda3 /mnt/home
genfstab -U /mnt >> /mnt/etc/fstab
```

Modify new generated fstab file according your preferences like change noatime on sdax:

```
mcedit /mnt/etc/fstab
```

### chroot

Change root to new system:

```
arch-chroot /mnt 
```

### timezone

Set the time zone:

```
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
```

Run hwclock to generate /etc/adjtime:

```
hwclock --systohc
```

### locale

Edit /etc/locale.gen and uncomment en_US.UTF-8 UTF-8 and other needed locales. Generate the locales by running:

```
locale-gen
```

```
echo LANG=en_US.UTF-8 > /etc/locale.conf
export LANG=en_US.UTF-8
```

## Network configuration

Create a /etc/hostname file and add the hostname entry:

```
echo myarch > /etc/hostname
```

Create hosts file:

```
touch /etc/hosts
```

And edit this /etc/hosts file with Nano editor to add the following lines to it (replace myarch with hostname you chose earlier):

```
127.0.0.1	localhost
::1		localhost
127.0.1.1	myarch
```

## Set up root passwd

You should also set the password for the root account using the passwd command:

```
passwd
```

## Install grub

Install grub on Non-UEFI systems

Install grub package first:
```
pacman -S grub
```

And then install grub like this (don’t put the disk number sda1, just the disk name sda):
```
grub-install /dev/sda
```

Last step:
```
grub-mkconfig -o /boot/grub/grub.cfg
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
