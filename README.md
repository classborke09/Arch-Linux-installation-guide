# Arch-Linux-installation-guide

##Btrfs

#Mount

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

• Remember to mount boot dir before install packages in /root 

## Packages to install 

linux linux-firmware base base-devel apparmor ufw vim zram-generator networkmanager efibootmgr nvidia-lts sbctl htop fuse2 git make amd-ucode btrfs-progs cronie exfat-utils efitools

# Package that could be install later!
power-profiles-daemon man less android-tools grub fwupd yazi reflector lutris steam code sshfs qemu virt-manager virt-viewer dnsmasq vde2 bridge-utils ebtables iptables libvirt swtpmnoto-fonts-cjk bluez-utils

# Gnome packages
gnome-firmware papers showtime resources

# Kde plasma
kate ark kalk okular gwenview haruna konsole kclock paritionmanager kdeconnect

# Miscs
Commit Mono or Geist Mono

## Service to startup
systemctl enable NetworkManager systemd-boot-update systemd-resolved apparmor ufw cronie bluetooth
