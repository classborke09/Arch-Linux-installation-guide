## Arch-Linux-installation-guide

For this guide i'll use _nvme_ as a hard drive and this partition pattern:

```
NAME                    MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINTS
/dev/nvme0n1            259:0    0 476.9G  0 disk  
├─/dev/nvme0n1p1        259:1    0     1G  0 part  /boot
└─/dev/nvme0n1p2        259:2    0 475.9G  0 part  
  └─/dev/mapper/luksdev 253:0    0 475.9G  0 crypt /
                                                   /.....
```

# Connect to interenet
> Nothing will be done in this guide without an internet! If you have already then skip this step.

**iwd**
```
iwctl
```
```
iwctl device list
```
```
iwctl station name scan
```
```
iwctl station name get-networks
```
```
iwctl station name connect SSID
```
```
iwctl exit
```
check if you have an internet by _ping -c 3 archlinux.org_

# Partition and install base packages 
```
fdisk /dev/nvme0n1
```
> Do it yourself!

# Encrypt your drive partition
```
cryptsetup -v luksFormat /dev/nvme0n1p2
```
> follow the prompt and your password!
```
cryptsetup luksOpen /dev/nvme0n1p2 luksdev
```

# Create filesystem for partitions
For nvme0n1p1
```
mkfs.fat -F32 /dev/nvme0n1p1
```
```
mkfs.btrfs /dev/mapper/luksdev
```

# Mount your partitions
**Boot partition**
```
mount --mkdir /dev/nvme0n1p1 /mnt/boot
```

**Root partition**
```
mount /dev/mapper/luksdev /mnt
```
```
cd /mnt
```
```
btrfs subvolume create @
```
```
btrfs subvolume create @home
```
```
btrfs subvolume create @opt
```
```
btrfs subvolume create @var_cache
```
```
btrfs subvolume create @var_lib_gdm
```
```
btrfs subvolume create @var_lib_libvirt
```
```
btrfs subvolume create @var_log
```
```
btrfs subvolume create @var_spool
```
```
btrfs subvolume create @var_tmp
```
```
cd
```
```
umount /mnt
```
```
mount -o noatime,compress=zstd,space_cache=v2,discard=async,subvol=@ /dev/luksdev /mnt
```
```
mount --mkdir -o noatime,compress=zstd,space_cache=v2,discard=async,subvol=@home /dev/luksdev /mnt/home
```
```
mount --mkdir -o noatime,compress=zstd,space_cache=v2,discard=async,subvol=@opt /dev/luksdev /mnt/opt
```
```
mount --mkdir -o noatime,compress=zstd,space_cache=v2,discard=async,subvol=@var_cache /dev/luksdev /mnt/var/cache
```
```
mount --mkdir -o noatime,compress=zstd,space_cache=v2,discard=async,subvol=@var_lib_gdm /dev/luksdev /mnt/var/lib/gdm
```
```
mount --mkdir -o noatime,compress=zstd,space_cache=v2,discard=async,subvol=@var_lib_libvirt /dev/luksdev /mnt/var/lib/libvirt
```
```
mount --mkdir -o noatime,compress=zstd,space_cache=v2,discard=async,subvol=@var_log /dev/luksdev /mnt/var/log
```
```
mount --mkdir -o noatime,compress=zstd,space_cache=v2,discard=async,subvol=@var_spool /dev/luksdev /mnt/var/spool
```
```
mount --mkdir -o noatime,compress=zstd,space_cache=v2,discard=async,subvol=@var_tmp /dev/luksdev /mnt/var/tmp
```

# Pacstrap
```
pacstrap -K /mnt linux linux-firmware base base-devel apparmor ufw vim zram-generator networkmanager efibootmgr sbctl htop fuse2 git make btrfs-progs cronie exfat-utils efitools dosfstools smartmontools
```
> If you have:
**Fingerprint Reader**
```
fprint
```
**Nvidia GPU (depend on your linux kernel version)**
```
nvidia 
```

# After pacstrap
**fstab**
```
genfstab -U /mnt >> /mnt/etc/fstab
```
**chroot**
```
arch-chroot /mnt
```

# Edit pacman file
Add _color_, _ILoveCandy_, _multilib_

# Package that could be install later
```
pacman -S power-profiles-daemon man less grub fwupd yazi reflector code sshfs qemu virt-manager dnsmasq vde2 bridge-utils iptables libvirt swtpm noto-fonts-cjk bluez-utils
```
**Android packages**
```
pacman -S android-tools android-udev scrcpy
```
**Gnome packages**
```
pacman -S gnome gnome-firmware papers showtime resources ptyxis
```
**Kde packages**
```
pacman -S plasma kate ark kalk okular gwenview haruna konsole kclock paritionmanager kdeconnect
```
**Miscs that you might don't need!**
```
pacman -S Commit Mono or Geist Mono
```

# Service to startup
```
systemctl enable NetworkManager systemd-boot-update systemd-resolved apparmor ufw cronie bluetooth
```

# Setup system time
```
ln -sf /usr/share/zoneinfo/Region/locatime /etc/localtime
```
```
hwclock --systohc
```
**localization**

_Edit_ /etc/locale.gen, uncomment _en_US.UTF-8_
```
locale-gen
```
```
echo "LANG=en_US.UTF-8" >> /etc/locale.conf
```
**Change your hostname**
```
echo "_yourhostname_" >> /etc/hostname
```

# Create username
```
useradd -m _yourname_
```
```
passwd _yourname_
```
```
usermod -aG wheel _yourname_
```
edit **visudo**

# Create zram
_Edit_ **/etc/systemd/zram-generator.conf**

```
[zram0]
zram-size = min(ram / 2, 4096)
compression-algorithm = zstd
```

# Setup bootloader with luks
Choose your bootloader, this guide I'll use systemd-boot and uki
```
bootctl install
```
**.preset file**

**_Edit_ /etc/mkinitcpio.d/linux.preset**
```
# mkinitcpio preset file for the 'linux' package

#ALL_config="/etc/mkinitcpio.conf"
ALL_kver="/boot/vmlinuz-linux"

PRESETS=('default' 'fallback')

#default_config="/etc/mkinitcpio.conf"
#default_image="/boot/initramfs-linux.img"
default_uki="esp/EFI/Linux/arch-linux.efi"
default_options="--splash=/usr/share/systemd/bootctl/splash-arch.bmp"

#fallback_config="/etc/mkinitcpio.conf"
#fallback_image="/boot/initramfs-linux-fallback.img"
fallback_uki="esp/EFI/Linux/arch-linux-fallback.efi"
fallback_options="-S autodetect"
```
```
blkid -o value -s UUID /dev/_encrypt_parition_ (eg, /dev/nvme0n1p2) >> /etc/cmdline.d/security.conf
```
```
blkid -o value -s UUID /dev/_dencrypt_parition_ (eg, /dev/mapper/luksdev) >> /etc/cmdline.d/security.conf
```
**_Edit_ your /etc/cmdline.d/root.conf**
```
rd.luks.name=_device-UUID_=luksdev root=UUID=luksdev_ rw rootfstype=btrfs rootflags=subvol=@
```
> Note: _device-UUID_ is a encrypt partition and _luksdev_ is a decrypt one

**_Edit_ mkinitcpio.conf**
Modify **HOOKS**
```
HOOKS=(base **systemd** autodetect microcode modconf kms keyboard **sd-vconsole** block **sd-encrypt** filesystems fsck)
```
```
mkinitcpio -P
```

# Ready to reboot step!
> So at this time your are done and prepare to reboot to your hard drive

**Exit your chroot**
```
exit
```
**Unmount your drive**
```
umount -R /mnt
```
**And**
```
reboot
```
