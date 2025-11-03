# Arching-Linux

# 🜂 Arching-Linux — Manual Installation Steps (Clear & Concise)

> This document lists **only the manual, one-by-one steps** to install Arch Linux from the live ISO.  
> Follow each numbered step in order. Commands to run are shown in code blocks. Read prompts and examples carefully — adapt device names (e.g., `/dev/nvme0n1p2`) to your machine.

---

## Preparations (Before you start)
1. Verify you have a working bootable Arch ISO on a USB (use Rufus / BalenaEtcher / `dd`).
2. Boot your machine in **UEFI** mode (recommended). Disable Secure Boot if necessary.
3. Optional: Have another device (phone/laptop) with the Arch Wiki open for reference.

---

## 1 — Boot & Network
1. Boot into the Arch live environment from the USB.
2. Confirm internet connectivity.

```bash
# For wired: usually automatic; test:
ping -c 3 archlinux.org

# For Wi-Fi (iwctl):
iwctl
# inside iwctl:
station wlan0 scan
station wlan0 get-networks
station wlan0 connect <Your_SSID>
exit
ping -c 3 archlinux.org

## 2 — Set the clock
timedatectl set-ntp true

## 3 — Partition the disk (UEFI example)

Warning: This erases data on the target disk. Double-check device names.

Start gdisk, fdisk, or cfdisk to partition (example with cfdisk):

cfdisk /dev/nvme0n1


Create partitions (example layout):

512M — EFI System Partition — type EFI System (FAT32)

Remaining — Root partition — type Linux filesystem (ext4 or btrfs)

Write and quit after partitioning.

Example device names:

EFI: /dev/nvme0n1p1

ROOT: /dev/nvme0n1p2

## 4 — Format partitions
# EFI as FAT32
mkfs.fat -F32 /dev/nvme0n1p1

# Root as ext4 (example)
mkfs.ext4 /dev/nvme0n1p2


(If you prefer btrfs/LVM/LUKS, format and configure here.)

## 5 — Mount filesystems
mount /dev/nvme0n1p2 /mnt
mkdir -p /mnt/boot
mount /dev/nvme0n1p1 /mnt/boot


If you have additional partitions (home, var, etc.) mount them under /mnt accordingly.

## 6 — Select mirrors (optional but recommended)

Edit /etc/pacman.d/mirrorlist or use reflector from the live environment (if available). Example (manual edit):

nano /etc/pacman.d/mirrorlist
# place fastest mirrors near top

## 7 — Install the base system
pacstrap /mnt base linux linux-firmware vim nano networkmanager


You can add base-devel if you plan to build packages.

## 8 — Generate fstab
genfstab -U /mnt >> /mnt/etc/fstab
cat /mnt/etc/fstab   # review entries

## 9 — Chroot into the new system
arch-chroot /mnt


From now on the commands run inside the chroot (they affect the installed system).

## 10 — Timezone & hardware clock
ln -sf /usr/share/zoneinfo/Asia/Kolkata /etc/localtime
hwclock --systohc


(Change the timezone path if not in Asia/Kolkata.)

## 11 — Locale
# Enable locales
echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen
locale-gen

# Set LANG
echo "LANG=en_US.UTF-8" > /etc/locale.conf

## 12 — Hostname & hosts
echo "myhostname" > /etc/hostname

cat > /etc/hosts <<EOF
127.0.0.1   localhost
::1         localhost
127.0.1.1   myhostname.localdomain myhostname
EOF


Replace myhostname with your chosen hostname.

## 13 — Root password
passwd


Enter a secure password.

## 14 — Create a normal user (recommended)
useradd -m -G wheel -s /bin/bash youruser
passwd youruser

# Allow wheel group to use sudo:
pacman -S --noconfirm sudo
EDITOR=nano visudo
# uncomment: %wheel ALL=(ALL) ALL

## 15 — Install and configure bootloader (UEFI — GRUB example)
pacman -S --noconfirm grub efibootmgr

# Ensure mount point
mkdir -p /boot/efi
mount /dev/nvme0n1p1 /boot/efi

grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=Arch
grub-mkconfig -o /boot/grub/grub.cfg


(Alternative: systemd-boot is simpler for UEFI-only systems.)

## 16 — Enable networking service
systemctl enable NetworkManager

## 17 — Install X/Wayland and desktop essentials (example)

Choose your stack. Minimal Xorg example:

pacman -S --noconfirm xorg xorg-xinit mesa


Example Wayland + Hyprland (optional):

pacman -S --noconfirm hyprland wayland-protocols wlroots swaybg waybar kitty rofi


(Adjust packages to your preference: GNOME, KDE, XFCE, or tiling WMs.)

## 18 — Install common tools
pacman -S --noconfirm network-manager-applet git vim htop neofetch

## 19 — Enable display manager (if using one)

Example for GDM:

pacman -S --noconfirm gdm
systemctl enable gdm


For SDDM, LightDM — install and enable corresponding service.

## 20 — (Optional) AUR helper & dotfiles — after first boot

AUR helpers like yay require base-devel and a user account:

pacman -S --noconfirm base-devel
# as your normal user:
git clone https://aur.archlinux.org/yay.git /tmp/yay
cd /tmp/yay
makepkg -si


Restore dotfiles to your home directory (via git clone or copying).

## 21 — Final steps: update & reboot
# inside chroot
pacman -Syu

# exit chroot, unmount, reboot
exit
umount -R /mnt
reboot


Remove the USB when appropriate so the system boots from the installed disk.
