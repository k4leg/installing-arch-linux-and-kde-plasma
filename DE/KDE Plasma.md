#### Installing KDE Plasma and other
    pacman -S --needed xorg-server
    # You don't need to install this if you will be using Wayland.
    # `xf86-video-ati` for AMD, `xf86-video-nouveau`, `nvidia` or `nvidia-lts`
    # for NVIDIA.

    pacman -S --needed apparmor firewalld
    pacman -S --needed networkmanager pulseaudio{,-bluetooth} youtube-dl \
        httpie git android-{tools,udev} systemd-swap cups gutenprint \
        xdg-user-dirs rkhunter openssh unhide reflector github-cli pass pwgen \
        pacman-contrib
    pacman -S --needed noto-fonts{,-{cjk,emoji}} ttf-jetbrains-mono plasma \
        mesa-vdpau kde-applications latte-dock kvantum-qt5 kitty
    pacman -S --needed transmission-qt thunderbird vlc krita qalculate-gtk \
        telegram-desktop libreoffice-still translate-shell discord ipython \
        pycharm-community-edition code clang anki lyx torbrowser-launcher \
        docker nnn fzf rsync texlive-{langcyrillic,fontsextra} \
        python{,-{pip,pylint,black,isort,neovim,jedi}} \
        firefox{,-{adblock-plus,extension-https-everywhere,tridactyl}} \
        zsh-theme-powerlevel10k

#### Setting up `apparmor`
    # Add "apparmor=1 lsm=lockdown,yama,apparmor,bpf" to
    # `GRUB_CMDLINE_LINUX_DEFAULT`.
    nvim /etc/default/grub

#### Setting up `systemd-swap`
    nvim /etc/systemd/swap.conf  # Set `swapfc_enabled=1`.

#### Setting up `/etc/environment`
    cat >> /etc/environment << EOF
    EDITOR=nvim
    VISUAL=nvim
    EOF

#### Service launch
    systemctl enable apparmor firewalld NetworkManager bluetooth sddm \
        systemd-swap cups fstrim.timer docker
    # `cups` for a printer, `fstrim.timer` for SSD.
