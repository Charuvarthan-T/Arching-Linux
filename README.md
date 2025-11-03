# 🜂 Arching-Linux — Manual Installation Guide

A **clear and concise** step-by-step guide for installing Arch Linux **manually** from the live ISO.  
Each step is self-contained and easy to follow. Commands are provided in blocks — modify disk names (`/dev/nvme0n1p2`, etc.) for your setup.

---

## ⚙️ Preparations (Before You Start)

1. **Create a Bootable USB**  
   Use tools like **Rufus**, **BalenaEtcher**, or `dd` to flash the Arch ISO.

2. **Boot into UEFI Mode**  
   Disable *Secure Boot* if necessary.

3. **Have the Arch Wiki Ready**  
   Keep another device (phone/laptop) open on the [Arch Wiki](https://wiki.archlinux.org/) for reference.

---

## 1 — Boot & Network

Boot into the Arch live environment from your USB.

### Check Connectivity
For **wired**, it usually connects automatically:
```bash
ping -c 3 archlinux.org
For Wi-Fi, use iwctl:

bash
Copy code
iwctl
# inside iwctl:
station wlan0 scan
station wlan0 get-networks
station wlan0 connect <Your_SSID>
exit
ping -c 3 archlinux.org
2 — Set the System Clock
bash
Copy code
timedatectl set-ntp true
3 — Partition the Disk (UEFI Example)
⚠️ Warning: This erases all data on the disk. Verify device names carefully.

Open the partition tool:

bash
Copy code
cfdisk /dev/nvme0n1
Recommended Layout:
Size	Type	Filesystem	Mountpoint
512 MB	EFI System Partition	FAT32	/boot
Remaining	Linux filesystem	ext4 or btrfs	/

Example device names:

EFI → /dev/nvme0n1p1

ROOT → /dev/nvme0n1p2

4 — Format Partitions
bash
Copy code
# EFI as FAT32
mkfs.fat -F32 /dev/nvme0n1p1

# Root as ext4
mkfs.ext4 /dev/nvme0n1p2
5 — Mount Filesystems
bash
Copy code
mount /dev/nvme0n1p2 /mnt
mkdir -p /mnt/boot
mount /dev/nvme0n1p1 /mnt/boot
If you have /home or others, mount them accordingly.

6 — Select Mirrors (Optional)
To speed up installation:

bash
Copy code
nano /etc/pacman.d/mirrorlist
# Move fastest mirrors to the top
7 — Install the Base System
bash
Copy code
pacstrap /mnt base linux linux-firmware vim nano networkmanager
You may add base-devel if you plan to build packages.

8 — Generate fstab
bash
Copy code
genfstab -U /mnt >> /mnt/etc/fstab
cat /mnt/etc/fstab
9 — Enter the New System
bash
Copy code
arch-chroot /mnt
All further commands are executed inside the chroot.

10 — Timezone & Hardware Clock
bash
Copy code
ln -sf /usr/share/zoneinfo/Asia/Kolkata /etc/localtime
hwclock --systohc
11 — Locale Setup
bash
Copy code
echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen
locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf
12 — Hostname & Hosts
bash
Copy code
echo "myhostname" > /etc/hostname

cat > /etc/hosts <<EOF
127.0.0.1   localhost
::1         localhost
127.0.1.1   myhostname.localdomain myhostname
EOF
Replace myhostname with your chosen hostname.

13 — Set Root Password
bash
Copy code
passwd
14 — Create a User Account
bash
Copy code
useradd -m -G wheel -s /bin/bash youruser
passwd youruser
Allow sudo access:

bash
Copy code
pacman -S --noconfirm sudo
EDITOR=nano visudo
# uncomment: %wheel ALL=(ALL) ALL
15 — Install & Configure Bootloader (UEFI — GRUB)
bash
Copy code
pacman -S --noconfirm grub efibootmgr
mkdir -p /boot/efi
mount /dev/nvme0n1p1 /boot/efi
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=Arch
grub-mkconfig -o /boot/grub/grub.cfg
🜂 Tip: For pure UEFI setups, systemd-boot is simpler.

16 — Enable Network Service
bash
Copy code
systemctl enable NetworkManager
17 — Install Display Stack (Optional)
Choose Xorg or Wayland.

Minimal Xorg:

bash
Copy code
pacman -S --noconfirm xorg xorg-xinit mesa
Wayland + Hyprland (example):

bash
Copy code
pacman -S --noconfirm hyprland wayland-protocols wlroots swaybg waybar kitty rofi
18 — Install Useful Tools
bash
Copy code
pacman -S --noconfirm network-manager-applet git vim htop neofetch
19 — Enable Display Manager (if using)
GDM Example:

bash
Copy code
pacman -S --noconfirm gdm
systemctl enable gdm
For SDDM or LightDM, enable their respective services.

20 — (Optional) AUR Helper & Dotfiles
After reboot and login as your normal user:

bash
Copy code
sudo pacman -S --noconfirm base-devel
git clone https://aur.archlinux.org/yay.git /tmp/yay
cd /tmp/yay
makepkg -si
Restore your dotfiles if you have them.

21 — Finalize Installation
bash
Copy code
pacman -Syu
exit
umount -R /mnt
reboot
Remove the USB when prompted to boot into your new Arch system.

