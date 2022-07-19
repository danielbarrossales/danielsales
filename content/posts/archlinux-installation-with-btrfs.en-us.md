---
title: "Installing Arch Linux with BTRFS filesystem"

date: 2021-04-08T11:46:29-03:00
url: installing-archlinux-with-btrfs/
image: images/2021-thumbs/archlinux-installation-with-btrfs.png
categories:
  - Linux
tags:
  - Archlinux
  - Btrfs
draft: false
---
## System installation
In this tutorial I assume that you already have an installation media ready and you know how to boot the system from it.
<!--more-->
### 1. Environment configuration
#### Key Mapping
The console's default keymap is US, but it can easily be changed from with the `loadkeys` command. To change the keymap to the Brazilian standard we can use the following command:
{{< highlight bash >}}
loadkeys br-abnt2
{{</highlight>}}

To find out which keymaps are available on your system:
{{< highlight bash >}}
localectl list-keymaps
{{</highlight>}}

### 2. Disk partitions
#### List disks
The `lsblk` command will return disks and partitions with extra data such as size and mount point.
#### Create partitions
At this point we have two options, that I know of, of commands that can be used: `fdisk` and `cfdisk`. Of these two options `cfdisk` is more friendly to newbies, as it is a GUI application.
Once you have chosen the disk you will use to install archlinux just use one of the commands above followed by the name of the disk. Ex.:
{{< highlight bash >}}
cfdisk /dev/sda
{{</highlight>}}
The goal is to have two partitions, first the boot partition with 512MB, the second with the rest of the available space.
In the rest of this guide I will assume that the name of the boot partition is `/dev/sda1` and that of the root partition is `/dev/sda2`, <strong> you should replace with the names of your system partitions.</strong >

#### Format partitions
{{< highlight bash >}}
mkfs.fat -F32 /dev/sda1 # Format boot partition with Fat32
mkfs.btrfs /dev/sda2 # Format root partition with btrfs
{{</highlight>}}

#### Creating subvolumes
One of the main reasons to use btrfs is the ease of backing up and restoring the system, a feature that can only be used if we have subvolumes. In this step, our goal is to create the following subvolumes:
| Subvolume | Description |
|-----------|-----------|
|@root|System root|
|@home|Users folder|
|@var|(Optional) will be mounted to /var|
|@tmp|(Optional) will be mounted to /tmp|
|@snapshots| This is where the system snapshots will be saved|
|@swap| This is where we will create our swapfile|

With the following commands:
{{< highlight bash >}}
mount /dev/sda2 /mnt # Mount the partition so we can create the subvolumes
btrfs subvolume create /mnt/@root # Create subvolume @root
btrfs subvolume create /mnt/@home # Create subvolume @home
btrfs subvolume create /mnt/@var # Create subvolume @var
btrfs subvolume create /mnt/@tmp # Create subvolume @tmp
btrfs subvolume create /mnt/@snapshots # Create subvolume @snapshots
btrfs subvolume create /mnt/@swap # Create subvolume @swap
umount /mnt # Unmount the partition so we can use the subvolumes we just created
{{</highlight>}}

Now we need to mount the subvolumes:

{{< highlight bash >}}
mount -o defaults,noatime,compress=zstd:2,subvol=@root /dev/sda2 /mnt # Mount system root
# If the system is not efi change boot/efi,... to boot...
mkdir -p /mnt/{boot/efi,var,home,tmp,.snapshots,.swap} # Create all specified directories recursively
mount -o defaults,noatime,compress=zstd:2,subvol=@home /dev/sda2 /mnt/home # Mount user folder
mount -o defaults,noatime,compress=zstd:2,subvol=@var /dev/sda2 /mnt/var
mount -o defaults,noatime,compress=zstd:2,subvol=@tmp /dev/sda2 /mnt/tmp
mount -o defaults,noatime,compress=zstd:2,subvol=@snapshots /dev/sda2 /mnt/.snapshots
mount -o defaults,noatime,compress=zstd:2,subvol=@swap /dev/sda2 /mnt/.swap
# If the system is not efi change /mnt/boot/efi to /mnt/boot
mount /dev/sda1 /mnt/boot/efi
{{</highlight>}}

Explanation: `compress=zstd:2` defines the algorithm and compression level for the file system, use `compress=none` if you want to opt out. `noatime` signals the system not to write to the file when it is read this can improve the lifespan of ssds as it has to do less writing but can cause applications that depend on it to not work properly.
In this step, if the system is BIOS and not UEFI, change all above mentions of boot/efi to boot.

### 3. Installation
{{< highlight bash >}}
pacstrap /mnt base base-devel linux linux-firmware nano # or vim depends on your preference
{{</highlight>}}

### 4. Configure the system
#### Configure swapfile
{{< highlight bash >}}
truncate -s 0 /mnt/.swap/swapfile # Create swapfile with size 0
chattr +C /mnt/.swap/swapfile # Disable COW on file
btrfs property set /mnt/.swap/swapfile compression none # Forces the file not to be compressed, the above command should make this not necessary, but it's here just in case
fallocate -l 9G /mnt/.swap/swapfile # allocates 9G of space for the swapfile, this size is only needed, in most cases, if we want to hibernate our machine.
chmod 600 /mnt/.swap/swapfile # Set permissions so that only the root user can access the file
mkswap /mnt/.swap/swapfile # Create swap file
swapon /mnt/.swap/swapfile # Mount swap
{{</highlight>}}

#### Generate fstab
fstab is the file responsible for defining how the partitions will be mounted during system startup. This file can be generated according to the partitions mounted in a given directory as follows:
{{< highlight bash >}}
genfstab -U /mnt >> /mnt/etc/fstab
{{</highlight>}}

#### Chroot, change root to installed system
{{< highlight bash >}}
arch-chroot /mnt # Change the root of the system, that is, changes here will be made on the installed system, until you exit this bash.
{{</highlight>}}
From here we'll be configuring the installed system, so make sure you don't exit bash above until we're done.

#### Timezone
{{< highlight bash >}}
# ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
ln -sf /usr/share/zoneinfo/America/Recife /etc/localtime # Set the time zone to Recife city
hwclock --systohc # Generate /etc/adjtime
{{</highlight>}}

#### Location
Locales or localizations are used by locale-aware programs and libraries to define how to correctly render text, values, time, etc...
To generate locales it is necessary to edit the file /etc/locale.gen and uncomment the lines with the desired locales, in our case pt_BR.UTF-8 UTF-8 (and in some cases en_US.UTF-8 UTF-8 as well since I had problems for not having this locale generated in the system).
{{< highlight bash >}}
locale-gen # generate uncommented line locales in /etc/locale.gen
echo "LANG=pt_BR.UTF-8" > /etc/locale.conf # Write the language that will be used by the system in the /etc/locale.conf file
echo "KEYMAP=br-abnt2" > /etc/vconsole.conf # Defines the keymap that will be used by the system's virtual consoles
{{</highlight>}}

#### Network
{{< highlight bash >}}
echo "my-hostname" > /etc/hostname # Set hostname
{{</highlight>}}

#### Safety
Change the root user password
{{< highlight bash >}}
passwd # will help you to set the password for the current user (root)
{{</highlight>}}

Add your user, after all you should not be using the system as root
{{< highlight bash >}}
useradd -m -g users -G wheel,storage,power,audio -s /bin/bash yourusername
passwd yourusername # set your user password
{{</highlight>}}

### 5. Configure System Boot
{{< highlight bash >}}
# Installing a bootloader and required dependencies
# efibootmgr is only needed if on an EFI system
pacman -S grub grub-btrfs efibootmgr dosfstools mtools ntfs-3g
# If the system is EFI use the command below to install grub (boot manager)
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB
# If not, use:
# grub-install /dev/sda # or whatever the name of the disk where your system's root partition is
grub-mkconfig -o /boot/grub/grub.cfg # generate boot configuration file
{{</highlight>}}

### 6. Install Plasma desktop environment and network
{{< highlight bash >}}
pacman -S sddm plasma networkmanager # Install plasma desktop environment, display manager and network manager
systemctl enable sddm # enable sddm to be initialized along with the system
systemctl enable NetworkManager # enable the network manager to be initialized along with the system
{{</highlight>}}

### 7. Finalization
Ready. You have installed Arch Linux.
Now:
{{< highlight bash >}}
exit # to exit chroot
umount -a # unmount all mounted partitions
reboot # reboots the system.
{{</highlight>}}

### References

[Copy-on-Write](https://pt.wikipedia.org/wiki/C%C3%B3pia_em_grava%C3%A7%C3%A3o)
[Installing Archlinux with BTRFS (medium.com)](https://medium.com/@fredlins/instala%C3%A7%C3%A3o-do-arch-linux-com-btrfs-grub-btrfs-snapper-snapper-gui-dc6eb1371f43#a2d7)
[Installing  Archlinux (archlinux.org)](https://wiki.archlinux.org/index.php/Installation_guide_(Portugu%C3%AAs)#Pr%C3%A9-instala%C3%A7%C3%A3o)
[Installing POP_OS with btrfs](https://mutschler.eu/linux/install-guides/pop-os-btrfs/)