#### Installing KDE Plasma and other packages
```bash
# You don't need to install `xorg-server` if you will be using Wayland.
# `xf86-video-ati` for AMD, `xf86-video-nouveau`, `nvidia` or `nvidia-lts` for
# NVIDIA.
pacman -S --needed xorg-server noto-fonts{,-{cjk,emoji}} ttf-jetbrains-mono \
    latte-dock kitty mesa-vdpau plasma kde-applications
```

#### Service launch
```bash
systemctl enable bluetooth sddm
```

