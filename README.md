## Arch Linux installation guide with Luks encrypt, Grub, Btrfs, Snapshots.

For this guide i'll use _nvme_ as a hard drive and this partition pattern:

```
NAME        FSTYPE      FSVER LABEL UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
zram0       swap        1     zram0 43dc6639-b81b-45f5-b47f-d4688de674fb                [SWAP]
nvme0n1
├─nvme0n1p1 vfat        FAT32       4778-D4C0                               1.4G    29% /boot
└─nvme0n1p2 crypto_LUKS 2           480af564-d84c-46b4-81c9-6e87207d7da9
  └─luksdev btrfs                   0034b945-8863-496f-b1c6-10d57635f5be  385.4G    19% /home
                                                                                        /.snapshots
                                                                                        /
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
btrfs subvolume create @snapshots
```
```
cd
```
```
umount /mnt
```
```
mount -o noatime,compress=zstd,space_cache=v2,discard=async,subvol=@ /dev/mapper/luksdev /mnt
```
```
mount --mkdir -o noatime,compress=zstd,space_cache=v2,discard=async,subvol=@home /dev/mapper/luksdev /mnt/home
```
```
mount --mkdir -o noatime,compress=zstd,space_cache=v2,discard=async,subvol=@snapshots /dev/mapper/luksdev /mnt/.snapshots
```

**Boot partition**
```
mount --mkdir /dev/nvme0n1p1 /mnt/boot
```

# Pacstrap
```
pacstrap -K /mnt linux-lts linux-firmware base base-devel apparmor ufw vim networkmanager efibootmgr btrfs-progs cronie tree exfat-utils efitools dosfstools smartmontools snapper grub grub-btrfs inotify-tools
```
> If you have:
**Fingerprint Reader**
```
fprint
```
**Nvidia GPU (depend on your linux kernel version)**
```
nvidia-lts
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
pacman -S sbctl htop fuse2 git make power-profiles-daemon man less fwupd yazi reflector code sshfs qemu virt-manager dnsmasq vde2 bridge-utils iptables libvirt swtpm noto-fonts-cjk bluez-utils thunderbird
```
**Android packages**
```
pacman -S android-tools android-udev scrcpy
```
**Gnome packages**
```
pacman -S gnome gnome-firmware papers showtime mission-center ghostty && pacman -Rns evince gnome-connections gnome-maps gnome-music gnome-user-docs totem sushi yelp
```
**Kde packages**
```
pacman -S plasma kate ark kalk okular gwenview dragon merkuro konsole kclock partitionmanager kdeconnect
```
**Miscs that you might don't need!**
```
pacman -S Commit Mono or Geist Mono
```

# Service to startup
```
systemctl enable NetworkManager systemd-resolved apparmor ufw cronie bluetooth libvirtd
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
echo "yourhostname" >> /etc/hostname
```

# Create username
```
useradd -m yourusername
```
```
passwd yourusername
```
```
usermod -aG wheel yourusername
```
```
visudo
```
> or nano, vi or whatever of your editor.

# Create zram
```
echo "zram" >> /etc/modules-load.d/zram.conf
```
```
echo 'ACTION=="add", KERNEL=="zram0", ATTR{initstate}=="0", ATTR{comp_algorithm}="zstd", ATTR{disksize}="8G", TAG+="systemd"' >> /etc/udev/rules.d/99-zram.rules
```
> Change the _disksize_ if nessesary.
```
echo "/dev/zram0 none swap defaults,discard,pri=100,x-systemd.makefs 0 0" >> /etc/fstab
```

# Setup bootloader with luks
```
grub-install --target=x86_64-efi --efi-directory=esp --bootloader-id=GRUB --modules="tpm cryptodisk" --disable-shim-lock
```
**Edit /etc/default/grub**
```
GRUB_CMDLINE_LINUX_DEFAULT="rd.luks.name=480af564-d84c-46b4-81c9-6e87207d7da9=luksdev root=UUID=0034b945-8863-496f-b1c6-10d57635f5be rw rootfstype=btrfs rootflags=subvol=@ lsm=landlock,lockdown,yama,integrity,apparmor,bpf"
```
> Please change your uuid(xxxx-xxxx-xxxx-xxxx).
```
grub-mkconfig -o /boot/grub/grub.cfg
```

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
```
reboot
```

# Log in your system
```
sudo umount /.snapshots/
```
```
sudo rm -rf /.snapshots/
```
```
sudo snapper -c root create-config /
```
Edit /etc/snapper/configs/root
```
sudo chmod a+rx /.snapshots/
```
```
sudo systemctl enable snapper-timeline.timer
```
```
sudo systemctl enable snapper-cleanup.timer
```
```
sudo systemctl enable grub-btrfsd
```
