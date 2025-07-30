## Arch-Linux-installation-guide

For this guide i'll use _nvme_ as a hard drive 
And this partition pattern:
```
nvme0n1
..nvme0n1p1                        /boot
..nvme0n1p2
...._nameofyourluksdrive_          /
                                   /...
                                   /...
```

# Connect to interenet
> Nothing will be done in this guide without an internet! If you have it already then skip this step.

**iwd**
```
iwctl

[iwd]# device list

[iwd]# station name scan

[iwd]# station name get-networks

[iwd]# station name connect SSID

[iwd]# exit
```
check if you connect to network by _ping -c 3 archlinux.org_

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
cryptsetup luksOpen /dev/nvme0n1p2 nameofyourluksdrive
```

**Btrfs**
if you using btrfs 
```
mount /dev/root_parti /mnt
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
mount -o noatime,compress=zstd,space_cache=v2,discard=async,subvol=@ /dev/root_partition /mnt
```
```
mount --mkdir -o noatime,compress=zstd,space_cache=v2,discard=async,subvol=@home /dev/home_partition /mnt/home
```
```
mount --mkdir -o noatime,compress=zstd,space_cache=v2,discard=async,subvol=@opt /dev/home_partition /mnt/opt
```
```
mount --mkdir -o noatime,compress=zstd,space_cache=v2,discard=async,subvol=@var_cache /dev/home_partition /mnt/var/cache
```
```
mount --mkdir -o noatime,compress=zstd,space_cache=v2,discard=async,subvol=@var_lib_gdm /dev/home_partition /mnt/var/lib/gdm
```
```
mount --mkdir -o noatime,compress=zstd,space_cache=v2,discard=async,subvol=@var_lib_libvirt /dev/home_partition /mnt/var/lib/libvirt
```
```
mount --mkdir -o noatime,compress=zstd,space_cache=v2,discard=async,subvol=@var_log /dev/home_partition /mnt/var/log
```
```
mount --mkdir -o noatime,compress=zstd,space_cache=v2,discard=async,subvol=@var_spool /dev/home_partition /mnt/var/spool
```
```
mount --mkdir -o noatime,compress=zstd,space_cache=v2,discard=async,subvol=@var_tmp /dev/home_partition /mnt/var/tmp
```

> **Remember to mount boot dir before install packages in /mnt** 

# Pacstrap
```
pacstrap -K /mnt linux linux-firmware base base-devel apparmor ufw vim zram-generator networkmanager efibootmgr sbctl htop fuse2 git make amd-ucode btrfs-progs cronie exfat-utils efitools dosfstools smartmontools
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
blkid -o value -s UUID /dev/_dencrypt_parition_ (eg, /dev/mapper/_cryptroot_) >> /etc/cmdline.d/security.conf
```
**_Edit_ your /etc/cmdline.d/root.conf**
```
rd.luks.name=_device-UUID_=root root=UUID=_cryptroot_ rw rootfstype=btrfs rootflags=subvol=@
```
> Note: _device-UUID_ is a encrypt partition and _cryptroot_ is a decrypt one
**_Edit_ mkinitcpio.conf**
modify **HOOKS**
```
HOOKS=(base **systemd** autodetect microcode modconf kms keyboard **sd-vconsole** block **sd-encrypt** filesystems fsck)
```
```
mkinitcpio -P
```

# Ready to reboot step!
> So at this time your are done and prepare to reboot to your harddrive
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
