<p align="center">
  <img src="https://cdn0.iconfinder.com/data/icons/flat-round-system/512/archlinux-512.png" alt="Arch Linux Logo" width="200"/>
</p>

# Linux (arch)

A clear and concise step-by-step guide for installing Arch Linux manually from the live ISO.  
Each step is self-contained and easy to follow. Commands are provided in blocks — modify disk names (`/dev/nvme0n1p2`, etc.) for your setup.

---

## Preparations

1. Create a Bootable USB  
   Use tools like Rufus, BalenaEtcher, or `dd` to flash the Arch ISO.

2. Boot into UEFI Mode  
   Disable Secure Boot in BIOS/UEFI if needed.

3. Have the Arch Wiki Ready  
   Keep another device open on the Arch Wiki for reference.

---

## 1 — Boot & Network

Boot into the Arch live environment from your USB.

### Check Connectivity (Wired)

```bash
ping -c 3 archlinux.org
Wi-Fi (iwctl)
```
```bash
 
iwctl
Inside iwctl:
```
```bash
 
station wlan0 scan
station wlan0 get-networks
station wlan0 connect <Your_SSID>
exit
Verify:
```

```bash
 
ping -c 3 archlinux.org
```
2 — Set the System Clock
```bash
 
timedatectl set-ntp true
```
3 — Partition the Disk (UEFI Example)
Warning: This erases all data. Verify device names carefully.

Launch the partitioning tool:

```bash
 
cfdisk /dev/nvme0n1
```
Recommended layout:

512 MB — EFI System Partition
Remaining — Linux filesystem
Example:
EFI → /dev/nvme0n1p1
ROOT → /dev/nvme0n1p2

4 — Format Partitions
```bash
mkfs.fat -F32 /dev/nvme0n1p1
mkfs.ext4 /dev/nvme0n1p2
```
5 — Mount Filesystems

```bash
 
mount /dev/nvme0n1p2 /mnt
mkdir -p /mnt/boot
mount /dev/nvme0n1p1 /mnt/boot
If you have a separate home partition:
```
```bash
mount /dev/nvme0n1p3 /mnt/home
```
6 — Select Mirrors (Optional)
```bash
 
nano /etc/pacman.d/mirrorlist
Move the fastest mirrors to the top.
```
7 — Install the Base System
```bash
 
pacstrap /mnt base linux linux-firmware vim nano networkmanager
```
Optional:
```bash
 
pacstrap /mnt base-devel
```
8 — Generate fstab
```bash
 
genfstab -U /mnt >> /mnt/etc/fstab
cat /mnt/etc/fstab
```
9 — Enter the New System
```bash
 
arch-chroot /mnt
All further commands now run inside the new system.
```
10 — Timezone & Hardware Clock
```bash
 
ln -sf /usr/share/zoneinfo/Asia/Kolkata /etc/localtime
hwclock --systohc
```
11 — Locale Setup

```bash
 
echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen
locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf
```
12 — Hostname & Hosts
```bash
 
echo "myhostname" > /etc/hostname
```
```bash
 
cat > /etc/hosts <<EOF
127.0.0.1    localhost
::1          localhost
127.0.1.1    myhostname.localdomain myhostname
EOF
```
Replace myhostname in both places.


13 — Set Root Password
```bash
 
passwd
```
14 — Create a User Account
```bash
 
useradd -m -G wheel -s /bin/bash youruser
passwd youruser
Install sudo:
```
```bash
 
pacman -S --noconfirm sudo
EDITOR=nano visudo
Uncomment the following line:
```
```sql
 
%wheel ALL=(ALL) ALL
```
15 — Install & Configure Bootloader (GRUB, UEFI)
```bash
 
pacman -S --noconfirm grub efibootmgr
mkdir -p /boot/efi
mount /dev/nvme0n1p1 /boot/efi
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=Arch
grub-mkconfig -o /boot/grub/grub.cfg
```
16 — Enable Network Service
```bash
 
systemctl enable NetworkManager
```
17 — Install Display Stack (Optional)
Minimal Xorg:

```bash

pacman -S --noconfirm xorg xorg-xinit mesa
Wayland + Hyprland:
```
```bash
pacman -S --noconfirm hyprland wayland-protocols wlroots swaybg waybar kitty rofi
```
18 — Install Useful Tools
```bash
 
pacman -S --noconfirm network-manager-applet git vim htop neofetch
```
19 — Enable Display Manager (Optional)
GDM example:

```bash
pacman -S --noconfirm gdm
systemctl enable gdm
```
20 — AUR Helper & Dotfiles (Optional, After Reboot)
```bash

sudo pacman -S --noconfirm base-devel git
git clone https://aur.archlinux.org/yay.git /tmp/yay
cd /tmp/yay
makepkg -si
```
21 — Finalize Installation
```bash

pacman -Syu
exit
umount -R /mnt
reboot
```

## References

• https://tar.ninja/posts/2017/03/Arch-Linux-With-SSD-Caching  
• https://wiki.archlinux.org/index.php/installation_guide  
• https://www.youtube.com/watch?v=METZCp_JCec  
• https://unix.stackexchange.com/questions/105389/arch-grub-asking-for-run-lvm-lvmetad-socket-on-a-non-lvm-disk  
