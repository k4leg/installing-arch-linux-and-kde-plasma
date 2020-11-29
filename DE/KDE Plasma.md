#### Installing KDE Plasma and other
    pacman -Sy --needed xorg-server
    # You don't need to install this if you will be using Wayland.
    # `xf86-video-ati` for AMD, `xf86-video-nouveau`, `nvidia` or `nvidia-lts` for NVIDIA.

    pacman -Sy --needed networkmanager pulseaudio youtube-dl android-tools \
        android-udev git xdg-user-dirs wget systemd-swap cups gutenprint
    pacman -Sy --needed plasma kde-applications partitionmanager kvantum-qt5 \
        latte-dock kitty inter-font ttf-jetbrains-mono adobe-source-han-sans-jp-fonts
    pacman -Sy --needed firefox{,-{adblock-plus,extension-https-everywhere,tridactyl}} \
        transmission-qt thunderbird vlc gimp qalculate-gtk telegram-desktop \
        libreoffice-still mesa-vdpau translate-shell ipython python-black \
        python-isort pycharm-community-edition python-neovim python-jedi \
        torbrowser-launcher discord anki flameshot lyx texlive-langcyrillic \
        zsh-{autosuggestions,history-substring-search,syntax-highlighting,theme-powerlevel10k}

#### Setting up `systemd-swap`
    echo 'swapfc_enabled=1' > /etc/systemd/swap.conf.d/overrides.conf

#### Setting up `/etc/environment`
    cat >> /etc/environment << EOF
    VISUAL=nvim
    EDITOR=nvim
    EOF

#### Service launch
    systemctl enable NetworkManager bluetooth sddm systemd-swap org.cups.cupsd fstrim.timer
    # `org.cups.cupsd` for a printer, `fstrim.timer` for SSD.
