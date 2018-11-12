# Arch Linux Dell XPS 15 (9560) - bspwm in xfce (Desktop)

## Install Desktop

### Install xfce4

```shell
sudo pacman -Syu
sudo pacman -S xorg-server xorg-xinit
lspci |grep VGA
sudo pacman -S xf86-video-intel
sudo localectl set-x11-keymap de pc105 nodeadkeys
sudo pacman -S xfce4 xfce4-goodies human-icon-theme
```

### Install xscreensaver

```shell
yay -S xscreensaver-aerial \
xscreensaver-aerial-videos-1080
```

### Install login manager

Install sddm:  
```shell
sudo pacman -S gst-libav \
phonon-qt5-gstreamer \
gst-plugins-good \
sddm
yay -S sddm-config-editor-git
```

Install Aerial theme:  
```shell
sudo git clone https://github.com/3ximus/aerial-sddm-theme.git /usr/share/sddm/themes/aerial-sddm-theme
```

\#**TODO** use local videos from the `xscreensaver-arial-videos` package
