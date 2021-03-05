#### Installing Sway and other
    pacman -Sy --needed networkmanager pulseaudio youtube-dl android-tools \
        android-udev git xdg-user-dirs wget systemd-swap cups gutenprint \
        mesa-vdpau rkhunter
    pacman -Sy --needed apparmor firewalld plasma-firewall
    pacman -Sy --needed lightdm-webkit-theme-litarvan sway grim slurp \
        wl-clipboard jq mako playerctl wofi network-manager-applet kitty \
        xorg-server-xwayland swaylock polkit-gnome qt5ct gammastep waybar \
        otf-font-awesome inter-font ttf-jetbrains-mono adobe-source-han-sans-jp-fonts
    pacman -Sy --needed firefox{,-{adblock-plus,extension-https-everywhere,tridactyl}} \
        transmission-gtk thunderbird vlc gimp qalculate-gtk telegram-desktop \
        ipython python-black python-isort pycharm-community-edition python-neovim \
        python-jedi libreoffice-still torbrowser-launcher discord anki lyx texlive-langcyrillic \
        zsh-{autosuggestions,history-substring-search,syntax-highlighting,theme-powerlevel10k}

#### Setting up `apparmor`
    # Add "apparmor=1 lsm=lockdown,yama,apparmor,bpf" to
    # `GRUB_CMDLINE_LINUX_DEFAULT`.
    nvim /etc/default/grub

#### Setting up lightdm
    sed -i 's/^#greeter-session=example-gtk-gnome$/greeter-session=lightdm-webkit2-greeter/g' /etc/lightdm/lightdm.conf
    sed -i '/^webkit_theme *= antergos$/s/antergos$/litarvan/g' /etc/lightdm/lightdm-webkit2-greeter.conf

#### Setting up systemd-swap
    echo 'swapfc_enabled=1' > /etc/systemd/swap.conf.d/overrides.conf

#### Setting up `/etc/environment`
    cat >> /etc/environment << EOF
    VISUAL=nvim
    EDITOR=nvim

    QT_QPA_PLATFORMTHEME=qt5ct

    _JAVA_AWT_WM_NONREPARENTING=1
    EOF

#### Service launch
    systemctl enable NetworkManager bluetooth fstrim.timer lightdm org.cups.cupsd systemd-swap
    # `org.cups.cupsd` for a printer, `fstrim.timer` for SSD.
