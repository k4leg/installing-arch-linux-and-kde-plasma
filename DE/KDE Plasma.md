#### Installing KDE Plasma and other
    pacman -Sy --needed xorg-server
    # You don't need to install this if you will be using Wayland.
    # `xf86-video-ati` for AMD, `xf86-video-nouveau`, `nvidia` or `nvidia-lts`
    # for NVIDIA.

    pacman -Sy --needed networkmanager pulseaudio youtube-dl wget git \
        android-{tools,udev} systemd-swap cups gutenprint xdg-user-dirs \
        rkhunter openssh unhide reflector github-cli pass pwgen
    pacman -Sy --needed noto-fonts{,-{cjk,emoji}} ttf-jetbrains-mono plasma \
        mesa-vdpau kde-applications latte-dock kvantum-qt5 kitty
    pacman -Sy --needed apparmor firewalld plasma-firewall
    pacman -Sy --needed transmission-qt thunderbird vlc krita qalculate-gtk \
        telegram-desktop libreoffice-still translate-shell discord ipython \
        pycharm-community-edition code clang anki lyx texlive-langcyrillic \
        torbrowser-launcher docker nnn \
        python{,-{pip,pylint,black,isort,neovim,jedi}}
        firefox{,-{adblock-plus,extension-https-everywhere,tridactyl}} \
        zsh-{autosuggestions,history-substring-search,syntax-highlighting,theme-powerlevel10k}

#### Setting up `apparmor`
    # Add "apparmor=1 lsm=lockdown,yama,apparmor,bpf" to
    # `GRUB_CMDLINE_LINUX_DEFAULT`.
    nvim /etc/default/grub

#### Setting up `systemd-swap`
    echo 'swapfc_enabled=1' > /etc/systemd/swap.conf.d/overrides.conf

#### Setting up `/etc/environment`
    cat >> /etc/environment << EOF
    VISUAL=nvim
    EDITOR=nvim
    EOF

#### Service launch
    systemctl enable apparmor firewalld NetworkManager bluetooth sddm \
        systemd-swap org.cups.cupsd fstrim.timer
    # `org.cups.cupsd` for a printer, `fstrim.timer` for SSD.
