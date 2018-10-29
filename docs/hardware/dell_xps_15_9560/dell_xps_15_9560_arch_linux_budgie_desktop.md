# Arch Linux Dell XPS 15 (9560) - Budgie (Desktop)

## Install Desktop

### Install X

```shell
sudo pacman -Syu
sudo pacman -S xorg \
xorg-server \
xorg-xinit
```

### Install Budgie

```shell
sudo pacman -S gnome \
gnome-screensaver \
budgie-desktop
```

### Install xscreensaver

```shell
sudo pacman -S xscreensaver-aerial \
xscreensaver-aerial-videos
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

## Post Config

### Enable network manager

```shell
sudo systemctl enable NetworkManager
```

### Install packages

```shell
sudo pacman -S firefox \
keepass \
keepass-plugin-keeagent \
inkscape \
gimp \
vlc \
qt4 \
thunderbird \
pinentry \
gpa \
nemo \
nemo-fileroller \
nemo-image-converter \
nemo-preview \
nemo-seahorse \
nemo-share

gpg --recv-keys --keyserver sks-keyservers.net 0xDB1187B9DD5F693B

yay -S keepass-plugin-rpc \
keepass-plugin-haveibeenpwned \
keepass-plugin-http \
keepass2-plugin-tray-icon \
nextcloud-client \
franz-bin \
thunderbird-enigmail \
gtkhash-nemo \
nemo-compare
```