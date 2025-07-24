## Arch-Linux-installation-guide

# Connect to interenet**
Nothing will be done in this guide without an internet! If you have it already then skip this step.

**iwd**
iwctl

[iwd]# device list

[iwd]# station name scan

[iwd]# station name get-networks

[iwd]# station name connect SSID

[iwd]# exit

check if you connect to network by _ping -c 3 archlinux.org_

# Partition 

fdisk /dev/_disk_nameyouwanttoparti_

Do it yourself!

**Btrfs**

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

# Packages to install

**From pacstrap**

linux linux-firmware base base-devel nvidia apparmor ufw vim zram-generator networkmanager efibootmgr sbctl htop fuse2 git make amd-ucode btrfs-progs cronie exfat-utils efitools

**Package that could be install later!**
power-profiles-daemon man less grub fwupd yazi reflector lutris steam code sshfs qemu virt-manager virt-viewer dnsmasq vde2 bridge-utils ebtables iptables libvirt swtpmnoto-fonts-cjk bluez-utils

**Android packages**
android-tools android-udev scrcpy

**Gnome packages**
gnome-firmware papers showtime resources

**Kde plasma**
kate ark kalk okular gwenview haruna konsole kclock paritionmanager kdeconnect

**Miscs**
Commit Mono or Geist Mono

# Service to startup
systemctl enable NetworkManager systemd-boot-update systemd-resolved apparmor ufw cronie bluetooth
