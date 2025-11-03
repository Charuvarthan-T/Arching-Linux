<p align="center">
  <img src="https://upload.wikimedia.org/wikipedia/commons/a/a5/Arch_Linux_%22Crystal%22_icon.svg" alt="Arch Linux Logo" width="200"/>
</p>
# 🜂 Arching-Linux — Manual Installation Guide

A **clear and concise** step-by-step guide for installing Arch Linux **manually** from the live ISO.
Each step is self-contained and easy to follow. Commands are provided in blocks — modify disk names (`/dev/nvme0n1p2`, etc.) for your setup.

---

## ⚙️ Preparations (Before You Start)

1.  **Create a Bootable USB**
    Use tools like **Rufus**, **BalenaEtcher**, or `dd` to flash the Arch ISO.

2.  **Boot into UEFI Mode**
    Disable *Secure Boot* in your BIOS/UEFI settings if necessary.

3.  **Have the Arch Wiki Ready**
    Keep another device (phone/laptop) open on the [Arch Wiki](https://wiki.archlinux.org/) for reference.

---

## 1 — Boot & Network

Boot into the Arch live environment from your USB.

### Check Connectivity
For a **wired** connection, it should work automatically. Test it:
```bash
ping -c 3 archlinux.org
For Wi-Fi, use the iwctl utility:

Bash

iwctl
Once inside iwctl, run these commands to connect:

Bash

station wlan0 scan
station wlan0 get-networks
station wlan0 connect <Your_SSID>
exit
Then, verify your connection:

Bash

ping -c 3 archlinux.org
2 — Set the System Clock
Update the system clock using NTP (Network Time Protocol):

Bash

timedatectl set-ntp true
3 — Partition the Disk (UEFI Example)
⚠️ Warning: This erases all data on the disk. Verify device names carefully.

Open the partition tool (e.g., cfdisk for /dev/nvme0n1):

Bash

cfdisk /dev/nvme0n1
Recommended Layout:

512 MB — Type: EFI System Partition

Remaining Size — Type: Linux filesystem

Example device names:

EFI → /dev/nvme0n1p1

ROOT → /dev/nvme0n1p2

4 — Format Partitions
Format the partitions with the correct filesystems.

Bash

# Format the EFI partition (p1) as FAT32
mkfs.fat -F32 /dev/nvme0n1p1

# Format the Root partition (p2) as ext4
mkfs.ext4 /dev/nvme0n1p2
5 — Mount Filesystems
Mount the root partition to /mnt. Then create the /boot directory and mount the EFI partition.

Bash

mount /dev/nvme0n1p2 /mnt
mkdir -p /mnt/boot
mount /dev/nvme0n1p1 /mnt/boot
If you have a separate /home or other partitions, mount them now (e.g., mount /dev/nvme0n1p3 /mnt/home).

6 — Select Mirrors (Optional)
To speed up the download, you can edit the mirror list.

Bash

nano /etc/pacman.d/mirrorlist
Move the fastest mirrors (e.g., for your country) to the top of the file.

7 — Install the Base System
Install the core Arch packages, kernel, and firmware.

Bash

pacstrap /mnt base linux linux-firmware vim nano networkmanager
You may also add base-devel if you plan to build packages from the AUR.

8 — Generate fstab
Generate the fstab file, which defines how disk partitions are mounted.

Bash

genfstab -U /mnt >> /mnt/etc/fstab
Check the file to ensure it looks correct:

Bash

cat /mnt/etc/fstab
9 — Enter the New System
chroot (change root) into your new system to configure it.

Bash

arch-chroot /mnt
All further commands are executed inside the new system.

10 — Timezone & Hardware Clock
Set your timezone (replace Asia/Kolkata as needed) and set the hardware clock.

Bash

ln -sf /usr/share/zoneinfo/Asia/Kolkata /etc/localtime
hwclock --systohc
11 — Locale Setup
Set up your system's language and character encoding.

Bash

echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen
locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf
12 — Hostname & Hosts
Set your computer's name (replace myhostname).

Bash

echo "myhostname" > /etc/hostname
Then, configure the hosts file to match.

Bash

cat > /etc/hosts <<EOF
127.0.0.1    localhost
::1          localhost
127.0.1.1    myhostname.localdomain myhostname
EOF
Replace myhostname with your chosen name in both places.

13 — Set Root Password
Set the password for the root (administrator) account.

Bash

passwd
14 — Create a User Account
Create your personal user account (replace youruser).

Bash

useradd -m -G wheel -s /bin/bash youruser
passwd youruser
Install sudo and allow your new user (as part of the wheel group) to use it.

Bash

pacman -S --noconfirm sudo
EDITOR=nano visudo
Inside the visudo editor, find and uncomment this line: %wheel ALL=(ALL) ALL

15 — Install & Configure Bootloader (UEFI — GRUB)
Install GRUB and efibootmgr for UEFI systems.

Bash

pacman -S --noconfirm grub efibootmgr
mkdir -p /boot/efi
mount /dev/nvme0n1p1 /boot/efi
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=Arch
grub-mkconfig -o /boot/grub/grub.cfg
🜂 Tip: For pure UEFI setups, systemd-boot is a simpler alternative to GRUB.

16 — Enable Network Service
Enable NetworkManager to start automatically on boot so you have internet.

Bash

systemctl enable NetworkManager
17 — Install Display Stack (Optional)
Choose a display server (Xorg or Wayland).

Minimal Xorg:

Bash

pacman -S --noconfirm xorg xorg-xinit mesa
Wayland + Hyprland (example):

Bash

pacman -S --noconfirm hyprland wayland-protocols wlroots swaybg waybar kitty rofi
18 — Install Useful Tools
Install some common and useful packages.

Bash

pacman -S --noconfirm network-manager-applet git vim htop neofetch
19 — Enable Display Manager (if using)
If you installed a desktop environment, enable its display manager (login screen).

GDM Example (for GNOME):

Bash

pacman -S --noconfirm gdm
systemctl enable gdm
For SDDM (KDE) or LightDM (XFCE), enable their respective services.

20 — (Optional) AUR Helper & Dotfiles
After you reboot and log in as your new user, you can install an AUR helper like yay.

Bash

sudo pacman -S --noconfirm base-devel git
git clone [https://aur.archlinux.org/yay.git](https://aur.archlinux.org/yay.git) /tmp/yay
cd /tmp/yay
makepkg -si
This is also a good time to restore your dotfiles if you have them.

21 — Finalize Installation
Run a final update, then exit the chroot, unmount all partitions, and reboot.

Bash

pacman -Syu
exit
umount -R /mnt
reboot
