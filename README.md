### Installing Arch Linux and Sway
#### Initial preparation
    iwctl  # Connect to Wi-Fi.
        device list
        station {wi-fi device} connect {SSID}
        # Next, you will be prompted to enter a password for Wi-Fi.
        exit
    timedatectl set-ntp true

#### Creating file systems, mounting and creating BTRFS subvolumes
First partition (`/dev/sda1`) for `/boot/efi` (UEFI only), second partition
(`/dev/sda2`) for root (`/`).

    mkfs.fat -F32 -n EFI /dev/sda1  # For UEFI only.
    mkfs.btrfs -f -L arch /dev/sda2 && mount -t btrfs /dev/sda2 /mnt
    for i in @{,snapshots{,_home},home,tmp{,_var},cache,log}; do
        btrfs subvolume create /mnt/$i
    done
    umount -R /mnt
    # The `ssd` flag is for SSD respectively.
    mount -t btrfs -o subvol=@,defaults,compress=zstd,ssd /dev/sda2 /mnt
    mkdir -p /mnt/{home,tmp,var/{tmp,cache,log}}
    mount -t btrfs -o subvol=@home,defaults,compress=zstd,ssd /dev/sda2 /mnt/home
    mount -t btrfs -o subvol=@tmp,defaults,compress=zstd,ssd /dev/sda2 /mnt/tmp
    mount -t btrfs -o subvol=@tmp_var,defaults,compress=zstd,ssd /dev/sda2 /mnt/var/tmp
    mount -t btrfs -o subvol=@cache,defaults,compress=zstd,ssd /dev/sda2 /mnt/var/cache
    mount -t btrfs -o subvol=@log,defaults,compress=zstd,ssd /dev/sda2 /mnt/var/log
    mount /dev/sda1 /mnt/boot/efi  # For UEFI only.

Note: we didn't mount `@snapshots` and `@snapshots_home` specifically so that we
would not remount them to configure `snapper` but simply mount them or reboot the
system (see `fstab` in the next section).

#### Basic installation and setup
    pacstrap /mnt base base-devel linux linux-firmware btrfs-progs grub zsh neovim \
        sudo man-db man-pages ripgrep
    # `intel-ucode` for Intel, `linux-firmware` for AMD.
    # `ntfs-3g` for NTFS support.
    # `os-prober` to find third-party OS (Windows, Mac OS, ...).

    genfstab -U /mnt >> /mnt/etc/fstab
    arch-chroot /mnt zsh
    ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
    hwclock --systohc
    nvim /etc/locale.gen  # Uncomment `#en_US.UTF-8 UTF-8`.
    locale-gen
    nvim /etc/locale.conf  # Add `LANG=en_US.UTF-8`.
    echo "arch" > /etc/hostname

    cat > /etc/hosts << EOF
    127.0.0.1   localhost
    ::1         localhost
    127.0.1.1   arch.localdomain    arch
    EOF

    nvim /etc/fstab

[Example](./fstab) for fstab.

    grub-install --recheck /dev/sda  # For BIOS only.
    grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=grub  # For UEFI only.
    grub-mkconfig -o /boot/grub/grub.cfg
    nvim /etc/pacman.conf +33  # Uncomment `#Color` and add `ILoveCandy` after `#VerbosePkgLists`.

#### Installing DE/WM and other
- [KDE Plasma](DE/KDE%20Plasma.md)
- [Sway](WM/Sway.md)
- etc.

#### Configuring users
    chsh -s /usr/bin/zsh
    passwd
    # Instead of `username` your username.
    useradd -d /home/username \
        -s /usr/bin/zsh \
        -G wheel,audio,video,input,adbusers \
        -m -N username
    passwd username
    sed -i '/^# %wheel ALL=(ALL) ALL$/s/^# //g' /etc/sudoers

#### End of installation
    exit
    umount -R /mnt && swapoff -a && reboot

#### Installing yay — the package manager for [AUR](https://aur.archlinux.org/)
    git clone https://aur.archlinux.org/yay.git ~
    cd ~/yay && makepkg -si && cd .. && rm -rf yay

#### Setting up snapper
    pacman -Sy --needed snapper snap-pac
    nvim /etc/fstab  # See an example for `fstab`.
    snapper -c root create-config /
    snapper -c home create-config /home
    btrfs subvolume delete /.snapshots
    btrfs subvolume delete /home/.snapshots
    mkdir /{,home/}.snapshots
    mount /.snapshots
    mount /home/.snapshots

Note: instead of the last two actions you can reboot.
