# Instalare arch

## Verify the boot mode

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
reflector -c "RO" -f 12 -l 10 -n 12 --save /etc/pacman.d/mirrorlist
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

## Install a network manager

```
pacman -S networkmanager
systemctl enable NetworkManager.service
```

## Generate fstab

Exit from chroot:

```
exit
```

Create any remaining mount points, mount the rest of devices and generate fstab:

```bash
mount /dev/sda3 /mnt/home
genfstab -U /mnt >> /mnt/etc/fstab
```

Modify new generated fstab file according your preferences like change noatime on sdaXX:

```bash
mcedit /mnt/etc/fstab
```

## Restart

```
reboot
```

##  Activate a connection

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

## Install Cinnamon

```
pacman -S xorg
pacman -S cinnamon
pacman -S firefox
pacman -S gnome-terminal xed xreader xdg-user-dirs
pacman -S lightdm lightdm-gtk-greeter
systemctl enable lightdm.service
```

then reboot

## Install pamac

Install devel libs:

```
sudo pacman -Syyu base-devel git
```

optimize makepkg here

Clone the Pamac PKGBUILD and build dependencies into a newly-created temporary folder under your home directory:

```
mkdir test
cd test
git clone https://aur.archlinux.org/pamac-aur.git
cd pamac-aur/
makepkg -sic
```

Pamac display empty results when browsing categories. You can fix by downgrade archlinux-appstream-data or use this command :

```
zcat /usr/share/app-info/xmls/community.xml.gz | sed 's|<em>||g;s|<\/em>||g;' | gzip > "new.xml.gz"
sudo cp new.xml.gz /usr/share/app-info/xmls/community.xml.gz
sudo appstreamcli refresh-cache --force
```
for appastreamcli install package appstream

## Access windows shares

```bash
pacman -S smbclient gvfs-smb
```

Because the samba package does not provide this file, one needs to create it . A documented example as in `smb.conf.default` from the [Samba git repository](https://git.samba.org/samba.git/?p=samba.git;a=blob_plain;f=examples/smb.conf.default;hb=HEAD) may be used to setup `/etc/samba/smb.conf`.  After installing cifs-utils or smbclient, load the `cifs` kernel module or reboot to prevent mount fails.

Latest versions of Samba no longer offer older authentication methods and protocols which are still used by some older clients (IP cameras, etc). These devices usually require Samba server to allow NTMLv1 authentication and NT1 version of the protocol, known as CIFS. For these devices to work with latest Samba, you need to add these two configuration parameters into [global] section:

```
server min protocol = NT1
client min protocol = NT1
ntlm auth = yes
```

Anonymous/guest access to a share requires just the first parameter. If the old device will access with username and password, you also need the add the second line too.

smbtree -b -N



## Install microcode for intel procs

```
sudo pacman -S intel-ucode
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

## Do some optimizations

### Disable wifi poweroff on laptop lid close

First find out if Arch applies power management to your wireless chipset:

```
pacman -S wireless_tools
iwconfig
```

You can then not only see the name for your wireless chipset (for example: wlp2s0), but also whether Power Management is **on** for it. When it's **off**, or when no mention is made of Power Management at all, you don't need to do anything.

Modify /etc/NetworkManager/conf.d/default-wifi-powersave-on.conf:

```
[connection]
wifi.powersave = 2
```

### Speed up your Intel wireless chipset

If you have a (reasonably) modern wireless chipset from Intel, it'll run on the iwlwifi driver. If so, you might be able to increase its speed noticeably, by turning on Tx AMPDU for it.

 The purpose of AMPDU is to improve data transmission by aggregating or  grouping together several sets of data. Thus it sharply reduces the  amount of transmission overhead.

 It used to be "on" by default in the iwlwifi driver. But several years  ago, it was turned off because of stability issues on a few wifi  chipsets. This problem affects, however, only a minority of chipsets...

 For turning it on, proceed like this:

First check whether your chipset runs on the iwlwifi driver:

```
lsmod | grep iwlwifi
```

If the terminal output contain the word **iwlwifi** proceed with the next step:

```
echo "options iwlwifi 11n_disable=8" | sudo tee /etc/modprobe.d/iwlwifi-speed.conf
```

### Speed up makepkg

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

### Decrease the swap use

This is especially noticeable on computers with relatively low RAM memory (2 GB or less): they tend to be far too slow. Check your current swappiness setting:

```
cat /proc/sys/vm/swappiness
```

The result will probably be 60. To set the swappiness value permanently, create a sysctl.d configuration file. For example:

```
mcedit /etc/sysctl.d/99-swappiness.conf
vm.swappiness=10
```

## Install a scanner and printer

```
pacman -S  xsane simple-scan hplip cups cups-pdf system-config-printer
systemctl enable org.cups.cupsd.service
```

## Install a lts kernel

```
pacman -S linux-lts linux-lts-headers
grub-mkconfig -o /boot/grub/grub.cfg
```

## Install virtualbox

Use a lts kernel, then install:

```
virtualbox, virtualbox-host-dkms, virtualbox-guest-iso, virtualbox-ext-oracle(AUR)
```

To use the USB ports of your host machine in your virtual machines, add  users that will be authorized to use this feature to the `vboxusers` user group.



## Install some applications

### Utilities:

#### Terminal

```
gnome-terminal, xterm
```

#### File managers

```
mc, doublecmd-gtk2
```

#### Archive managers

```
file-roller (nemo-fileroller), p7zip, unrar
```

#### Diff comparison

```
meld
```

#### File searching

```
catfish
```

#### Integrated development environments

```
geany (geany-plugins), atom
```

#### Character selectors

```
gucharmap
```

#### Partitioning and formatting tools

```
gparted(with all dependecies), gnome-disk-utility
```

#### Disk usage display

```
baobab, gdmap
```

#### Disk image writing

```
balena-etcher, usbimager
```

#### System monitors

```
gnome-system-monitor, gnome-usage, htop, stacer, pacmanlogviewer
```

#### System information viewers

```
hardinfo, inxi, screenfetch, gnome-firmware(fwupd gui)
```

#### Font viewers

```
font-manager
```

#### PDF viewer

```
 xviewer(AUR)
```

#### Image editing/viewer

```
viewnior, gimp
```

#### Sound, video

```
audacious, vlc, kodi
```

#### Internet

```
transmission-gtk, opera (opera-ffmpeg-codecs), filezilla
```

#### Networking

```
openssh
```

#### Cinnamon stuff

```
mintlocale, mint-themes, mint-y-icons
```

Cinnamon applets: weather, system monitor (install libgtop + restart), session manager

#### Accesories: 

```
lightdm-gtk-greeter-settings, gnome-calculator, gnome-screenshot, 
```

#### Office suite 

```
libreoffice-fresh, libreoffice-fresh-ro, hunspell, hunspell-ro, ttf-ms-fonts, typora
```


