### Установка Arch Linux с KDE Plasma
#### Начальная подготовка
    iwctl   # Подключится к wi-fi.
        device list
        station {wi-fi device} connect {SSID}
        # Далее появится приглашение на ввод пароля для wi-fi.
        exit
    timedatectl set-ntp true
#### Создание файловых систем, монтирование и создание подтомов BTRFS
Первый раздел (`/dev/sda1`) для `/boot/efi` (только для UEFI), 2-й раздел (`/dev/sda2`) для swap, 3-й раздел (`/dev/sda3`) для корня (`/`).

    mkfs.fat -F32 -n EFI /dev/sda1      # Только для UEFI.
    mkswap -L swap /dev/sda2 && swapon /dev/sda2
    mkfs.btrfs -f -L arch /dev/sda3 && mount -t btrfs /dev/sda3 /mnt
    for i in @{,snapshots{,_home},home,tmp{,_var},cache,log}; do
        btrfs subvolume create /mnt/$i
    done
    umount -R /mnt
    # Флаг `ssd`, соответственно, для SSD.
    mount -t btrfs -o subvol=@,defaults,compress=zstd,ssd /dev/sda3 /mnt
    mkdir -p /mnt/{home,tmp,var/{tmp,cache,log}}
    mount -t btrfs -o subvol=@home,defaults,compress=zstd,ssd /dev/sda3 /mnt/home
    mount -t btrfs -o subvol=@tmp,defaults,compress=zstd,ssd /dev/sda3 /mnt/tmp
    mount -t btrfs -o subvol=@tmp_var,defaults,compress=zstd,ssd /dev/sda3 /mnt/var/tmp
    mount -t btrfs -o subvol=@cache,defaults,compress=zstd,ssd /dev/sda3 /mnt/var/cache
    mount -t btrfs -o subvol=@log,defaults,compress=zstd,ssd /dev/sda3 /mnt/var/log
    mount /dev/sda1 /mnt/boot/efi   # Только для UEFI.
Примечание: мы не подмонтировали `@snapshots` и `@snapshots_home` специально для того, чтобы потом не перемонтировать их для настройки `snapper`, а просто примонтировать их или перезагрузить систему (см. `fstab` в следующем разделе).
#### Базовая установка и настройка
    pacstrap /mnt base base-devel linux linux-firmware btrfs-progs ntfs-3g grub \
        os-prober zsh neovim sudo intel-ucode man-db man-pages ripgrep openssh
                                # `intel-ucode` для Intel, а для AMD — `linux-firmware`.
                                # `ntfs-3g` для поддержки NTFS.
                                # `ripgrep` замена для `grep`.
                                # `os-prober` для нахождения сторонних ОС (Windows,
                                # Mac OS, ...).

    genfstab -U /mnt >> /mnt/etc/fstab
    arch-chroot /mnt zsh
    ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
    hwclock --systohc
    nvim /etc/locale.gen        # Раскомментируем `#en_US.UTF-8 UTF-8`.
    locale-gen
    nvim /etc/locale.conf       # Добавляем `LANG=en_US.UTF-8`.
    echo "arch" > /etc/hostname
    nvim /etc/hosts     # То, что нужно туда добавить отмечено отступом.
        127.0.0.1   localhost
        ::1         localhost
        127.0.1.1   arch.localdomain    arch

    nvim /etc/fstab

[Пример](./fstab) для fstab.

    grub-install --recheck /dev/sda     # Только для BIOS.
    grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=grub     # Только для UEFI.
    grub-mkconfig -o /boot/grub/grub.cfg
    nvim /etc/pacman.conf +33   # Раскомментируем `#Color` и добавляем, после
                                # `#VerbosePkgLists` `ILoveCandy`.
#### Установка сервисов, рабочего окружения и приложений
    pacman -Sy --needed xorg-server xf86-video-intel xf86-video-ati
                                # Не нужно это устанавливать, если вы будете
                                # использовать Wayland.
                                # `xf86-video-ati` для AMD, для NVIDIA —
                                # `xf86-video-nouveau`.
    pacman -Sy --needed networkmanager pulseaudio youtube-dl android-tools \
        android-udev imagemagick git xdg-user-dirs wget
    pacman -Sy --needed plasma kde-applications partitionmanager kvantum-qt5 \
        latte-dock
    pacman -Sy --needed firefox{,-adblock-plus,-extension-https-everywhere} \
        qbittorrent thunderbird vlc deadbeef gimp qalculate-gtk telegram-desktop \
        pycharm-community-edition anki cups libreoffice-fresh python-black \
        torbrowser-launcher discord ipython gutenprint fzf mesa-vdpau \
        ttf-jetbrains-mono adobe-source-han-sans-jp-fonts \
        zsh-{autosuggestions,history-substring-search,syntax-highlighting,theme-powerlevel10k}
#### Запуск сервисов
    systemctl enable NetworkManager bluetooth sddm org.cups.cupsd fstrim.timer
    # `org.cups.cupsd` для принтера.
    # `fstrim.timer` для SSD.
#### Настройка пользователей
    chsh -s /usr/bin/zsh
    passwd
    useradd -d /home/username \
        -s /usr/bin/zsh \
        -G wheel,audio,video,input,adbusers \
        -m -N username
    passwd username     # Вместо `username` своё имя пользователя.
    sed -i '/^# %wheel ALL=(ALL) ALL$/s/^# //g' /etc/sudoers
#### Конец установки
    exit
    umount -R /mnt && swapoff -a && reboot
#### Установка yay — пакетного менеджера для [AUR](https://aur.archlinux.org/)
    git clone https://aur.archlinux.org/yay.git ~
    cd ~/yay && makepkg -si && cd - && rm -rf ~/yay
#### Настройка snapper'а
    pacman -S snapper snap-pac
    nvim /etc/fstab     # См. пример для `fstab`.
    snapper -c root create-config /
    snapper -c home create-config /home
    btrfs subvolume delete /.snapshots
    btrfs subvolume delete /home/.snapshots
    mkdir /{,home/}.snapshots
    mount /.snapshots
    mount /home/.snapshots
Примечание: вместо двух последних действий можно перезагрузится.

