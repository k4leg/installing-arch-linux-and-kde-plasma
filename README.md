### Installing Arch Linux
#### Initial preparation
```bash
iwctl  # Connect to Wi-Fi.
    device list
    station {wi-fi device} connect {SSID}
timedatectl set-ntp true
```

#### Creating file systems and mounting them
Partition `/dev/sda1` for `/boot/efi` (UEFI only), partition `/dev/sda2` for
root.

```bash
mkfs.ext4 -L arch /dev/sda2
mount /dev/disk/by-label/arch /mnt
# For UEFI only.
mkfs.fat -n boot -F 32 /dev/sda1
mkdir -p /mnt/boot/efi
mount /dev/disk/by-label/boot /mnt/boot/efi
```

#### Basic installation and setup
[Example](fstab-example) for fstab.

```bash
# Add `intel-ucode` for Intel, `amd-ucode` for AMD.
pacstrap /mnt base linux-{lts,firmware} e2fsprogs networkmanager neovim \
    man-{db,pages} texinfo grub zsh sudo
genfstab -U /mnt >> /mnt/etc/fstab  # Check /mnt/etc/fstab.
arch-chroot /mnt zsh
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
hwclock --systohc
nvim /etc/locale.gen  # Uncomment `#en_US.UTF-8 UTF-8`.
locale-gen
echo LANG=en_US.UTF-8 > /etc/locale.conf
echo arch > /etc/hostname
# `arch` is hostname.  If the system has a permanent IP address, it should be
# used instead of 127.0.1.1.
cat > /etc/hosts << EOF
127.0.0.1   localhost
::1         localhost
127.0.1.1   arch.localdomain    arch
EOF
grub-install --recheck /dev/sda  # For BIOS only.
grub-install --recheck \
    --target=x86_64-efi \
    --efi-directory=/boot/efi \
    --bootloader-id=grub  # For UEFI only.
grub-mkconfig -o /boot/grub/grub.cfg
nvim /etc/pacman.conf  # Uncomment `#Color`, `#ParallelDownloads`.
chsh -s "$(which zsh)"
passwd
useradd -d /home/username \
    -s "$(which zsh)" \
    -G wheel,audio,video,input,users \
    -m \
    username
passwd username
sed -i '/^# %wheel ALL=(ALL) ALL$/s/^# //g' /etc/sudoers
logout
umount -R /mnt && reboot
# After booting and logging in as username in Arch Linux.
systemctl enable --now NetworkManager
nmtui  # Connect to Wi-Fi.
```

#### Installing other packages and DE/WM
```bash
pacman -S --needed base-devel apparmor networkmanager pipewire{,-pulse} \
    youtube-dl wget git{,-delta} android-{tools,udev} systemd-swap cups \
    gutenprint xdg-user-dirs openssh reflector pass pwgen pacman-contrib
pacman -S --needed qbittorrent thunderbird vlc gimp inkscape qalculate-gtk \
    telegram-desktop libreoffice-still translate-shell discord ipython anki \
    torbrowser-launcher docker wireshark-qt texlive-{langcyrillic,fontsextra} \
    python-{pip,pylint,black,isort,neovim} \
    firefox{,-{i18n-en-us,clearurls,extension-https-everywhere,tridactyl,ublock-origin}} \
    zsh-theme-powerlevel10k
```

- [KDE Plasma](DE/KDE%20Plasma.md)

#### Setting up `apparmor`
```bash
# Add "lsm=landlock,lockdown,yama,apparmor,bpf" to
# `GRUB_CMDLINE_LINUX_DEFAULT`.
nvim /etc/default/grub
```

#### Setting up `systemd-swap`
```bash
nvim /etc/systemd/swap.conf  # Set `swapfc_enabled=1`.
```

#### Service launch
```bash
systemctl enable apparmor systemd-swap cups fstrim.timer docker
# `cups` for a printer, `fstrim.timer` for SSD.
```

#### Setting up users groups
```bash
usermod -aG docker,wireshark,adbusers username
```

#### Installing [yay](https://github.com/Jguer/yay)
```bash
export PATH_TO_BUILD_DIR_YAY="$(mktemp -d)"
git clone https://aur.archlinux.org/yay.git "$PATH_TO_BUILD_DIR_YAY"
(cd "$PATH_TO_BUILD_DIR_YAY" && makepkg -si) && rm -rf "$PATH_TO_BUILD_DIR_YAY"
```

