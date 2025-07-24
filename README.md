## Arch-Linux-installation-guide

# Connect to interenet**
Nothing will be done in this guide without an internet! If you have it already then skip this step.

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
fdisk /dev/_disk_nameyouwanttoparti
```
> Do it yourself!

**Btrfs**
if you using btrfs 
```
mount /dev/root_parti /mnt

cd /mnt

btrfs subvolume create @

btrfs subvolume create @home

cd

umount /mnt

mount -o noatime,compress=zstd,space_cache=v2,discard=async,subvol=@
/dev/root_partition /mnt

mkdir /mnt/home

mount -o noatime,compress=zstd,space_cache=v2,discard=async,subvol=@home
/dev/home_partition /mnt/home
```
• Remember to mount boot dir before install packages in /root 

**pacstrap**
```
pacstrap -K /mnt linux linux-firmware base base-devel nvidia apparmor ufw vim zram-generator networkmanager efibootmgr sbctl htop fuse2 git make amd-ucode btrfs-progs cronie exfat-utils efitools
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

# Package that could be install later
```
pacman -S power-profiles-daemon man less grub fwupd yazi reflector lutris steam code sshfs qemu virt-manager virt-viewer dnsmasq vde2 bridge-utils ebtables iptables libvirt swtpmnoto-fonts-cjk bluez-utils
```
**Android packages**
```
pacman -S android-tools android-udev scrcpy
```
**Gnome packages**
```
pacman -S gnome-firmware papers showtime resources
```
**Kde plasma**
```
pacman -S kate ark kalk okular gwenview haruna konsole kclock paritionmanager kdeconnect
```
**Miscs**
```
pacman -S Commit Mono or Geist Mono
```

# Service to startup
```
systemctl enable NetworkManager systemd-boot-update systemd-resolved apparmor ufw cronie bluetooth
```

# Setup system time
```
ln -sf /usr/share/zoneinfo/Region/location /etc/localtime
```
```
hwclock --systohc
```
**localization**

Edit /etc/locale.gen, uncomment _en_US.UTF-8_
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

# Setup bootloader with luks
Choose your bootloader, this guide I'll use systemd-boot and uki
```
bootctl install
```
**.preset file**
```
**Edit /etc/mkinitcpio.d/linux.preset**
==================================
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
**Edit your /etc/cmdline.d/security.conf**
```
rd.luks.name=_device-UUID_=root root=UUID=_cryptroot_
```
> Note: _device-UUID_ is a encrypt partition and _cryptroot_ is a decrypt one
**Edit mkinitcpio.conf**
modify **HOOKS**
```
HOOKS=(base **systemd** autodetect microcode modconf kms keyboard **sd-vconsole** block **sd-encrypt** filesystems fsck)
```
```
mkinitcpio -P
```
