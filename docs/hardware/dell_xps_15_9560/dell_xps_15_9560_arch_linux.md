# Arch Linux Dell XPS 15 (9560)

## Pre-installation

### UEFI

Before installing it is necessary to modify some UEFI Settings. They can be accessed by pressing the F2 key repeatedly when booting.

* Change the SATA Mode from the default "RAID" to "AHCI". This will allow Linux to detect the NVME SSD. If dual booting with an existing Windows installation, Windows will not boot after the change but this can be fixed without a reinstallation.
* Change Fastboot to "Thorough" in "POST Behaviour". This prevents intermittent boot failures.
* Disable secure boot to allow Linux to boot.

Installation of Arch Linux can proceed normally. Refer to the [Installation guide](https://wiki.archlinux.org/index.php/Installation_guide) for more information. 

### Change Keyboard layout

If you don't have an EN:Intl. Keyboard layout you should change it to your layout. In my case it is `de-latin1-nodeadkeys1`.

```shell
loadkeys de-latin1-nodeadkeys1
```

### Connect to a wireless network during installation (temporary)

Change to TTY2 with `ALT`+`F2` and use the following example to connect to your network.

```shell
wpa_supplicant -D nl80211,ext -i wlp2s0 -c <(wpa_passphrase 'YourWififNetwork' 'YourWifiPassword')
```

Afterwards you establish the connection change back to TTY1 with `ALT`+`F1`.

Run dhclient to reciev an ip via dhcp:  
```shell
dhclient wlp2s0
```

### Update the system clock

Use [timedatectl(1)](http://jlk.fjfi.cvut.cz/arch/manpages/man/timedatectl.1) to ensure the system clock is accurate:  
```shell
# timedatectl set-ntp true
# systemctl start systemd-timesyncd
```

To check the service status, use timedatectl status.

## Installation

### Preparing Disks

I'll encrypt my Data so my Partition layout looks like this:

```
+----------------+----------------+----------------+-----------------------------------------------------------------------------------------------+
|                |                |                |                       |                       |                       |                       |
|                |                |                | LUKS encrypted volume | LUKS encrypted volume | LUKS encrypted volume | LUKS encrypted volume |
|                |                |                | /dev/mapper/swap      | /dev/mapper/tmp       | /dev/mapper/root      | /dev/mapper/home      |
|                |                |                |_ _ _ _ _ _ _ _ _ _ _ _|_ _ _ _ _ _ _ _ _ _ _ _|_ _ _ _ _ _ _ _ _ _ _ _|_ _ _ _ _ _ _ _ _ _ _ _|
|                |                |                |                       |                       |                       |                       |
|                |                |                |         16 GiB        |         30 GiB        |         50 GiB        |        100%FREE       |
|                |                |                |    Logical volume1    |    Logical volume2    |    Logical volume3    |    Logical volume4    |
|                |                |                | /dev/mapper/osvg-swap | /dev/mapper/osvg-tmp  | /dev/mapper/osvg-root | /dev/mapper/osvg-home |
|                |                |      EF02      |_ _ _ _ _ _ _ _ _ _ _ _|_ _ _ _ _ _ _ _ _ _ _ _|_ _ _ _ _ _ _ _ _ _ _ _|_ _ _ _ _ _ _ _ _ _ _ _|
|      EF00      |      8300      |      8 MiB     |                                                                                               |
|     550 MiB    |     500 MiB    |      BIOS      |                                             8E00                                              |
|  EFI partition | boot partition | Boot partition |                                           100%FREE                                            |
| /dev/nvme0n1p1 | /dev/nvme0n1p2 | /dev/nvme0n1p3 |                                        /dev/nvme0n1p4                                         |
+----------------+----------------+----------------+-----------------------------------------------------------------------------------------------+
```

Create these partitions with `gdisk` so that you have `GPT` instead of `MBR`.

#### Wipe Data on all partitions

```shell
# dd if=/dev/zero of=/dev/nvme0n1p1 bs=1M status=progress
# dd if=/dev/zero of=/dev/nvme0n1p2 bs=1M status=progress
# dd if=/dev/zero of=/dev/nvme0n1p3 bs=1M status=progress
# dd if=/dev/zero of=/dev/nvme0n1p4 bs=1M status=progress
```

#### Preparing the logical volumes

```shell
# pvcreate /dev/nvme0n1p4
# vgcreate osvg /dev/nvme0n1p4
# lvcreate -L 16G -n swap osvg
# lvcreate -L 30G -n tmp osvg
# lvcreate -L 50G -n root osvg
# lvcreate -l 100%FREE -n home osvg
```

#### Setup `LUKS` for our partitions

Create an encrypted root with a key you can remember.  
```shell
# cryptsetup luksFormat /dev/mapper/osvg-root
```

Open the root filesystem to create an encryption key for all other partitions.  
```shell
# cryptsetup luksOpen /dev/mapper/osvg-root root
# mkfs.ext4 /dev/mapper/root
# mnt /dev/mapper/root /mnt
# mkdir -pm 700 /mnt/etc/luks-keys
# dd if=/dev/random of=/mnt/etc/luks-keys/home bs=1 count=1024 status=progress
# chmod 000 /mnt/etc/luks-keys/home
```

Use the encrypted key to create our encrypted home and add a pass phrase for emergency access:  
```shell
# cryptsetup luksFormat /dev/mapper/osvg-home
# cryptsetup luksAddKey /dev/mapper/osvg-home --key-file=/mnt/etc/luks-keys/home
```

Now we open home, make a filesystem and mount it to `/mnt/home`.  
```shell
# cryptsetup luksOpen --key-file=/mnt/etc/luks-keys/home /dev/mapper/osvg-home home
# mkfs.ext4 /dev/mapper/home
# mkdir -p /mnt/home
# mount /dev/mapper/home /mnt/home
```

and rebuild the EFI and boot partition:  
```shell
# mkfs.vfat /dev/nvme0n1p1
# mkfs.ext4 /dev/nvme0n1p2
```

Setup and mount the boot/EFI partition:  
```shell
# mkdir /mnt/boot
# mount /dev/nvme0n1p2 /mnt/boot
# mkdir /mnt/boot/EFI
# mkdir /mnt/esp
# mount /dev/nvme0n1p1 /mnt/esp
```

### Install Arch

#### Sort mirrors by speed

The [pacman](https://www.archlinux.org/packages/?name=pacman) package provides a Bash script, `/usr/bin/rankmirrors`, which can be used to rank the mirrors according to their connection and opening speeds to take advantage of using the fastest local mirror.  
Back up the existing `/etc/pacman.d/mirrorlist`:  
```shell
# cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.bak
```

Edit `/etc/pacman.d/mirrorlist.bak` and uncomment mirrors for testing with `rankmirrors`.  
Run the following `sed` line to uncomment every mirror:  
```shell
# sed -i 's/^Server/#Server/' /etc/pacman.d/mirrorlist.bak
```

Finally, rank the mirrors. Operand `-n 6` means only output the 6 fastest mirrors:  
```shell
# rankmirrors -n 6 /etc/pacman.d/mirrorlist.bak > /etc/pacman.d/mirrorlist
```

#### Install the base packages

Use the [pacstrap](https://projects.archlinux.org/arch-install-scripts.git/tree/pacstrap.in) script to install the [`base`](https://www.archlinux.org/groups/x86_64/base/) package group:  
```shell
# pacstrap /mnt base
```
This group does not include all tools from the live installation, such as [`btrfs-progs`](https://www.archlinux.org/packages/?name=btrfs-progs) or specific wireless firmware; see [packages.both](https://projects.archlinux.org/archiso.git/tree/configs/releng/packages.both) for comparison.
To [install](#Install_Arch) packages and other groups such as [`base-devel`](https://www.archlinux.org/groups/x86_64/base-devel/), append the names to *pacstrap* (space separated) or to individual [pacman](https://wiki.archlinux.org/index.php/Pacman) commands after the [#Chroot](#Chroot) step.

### Configure the system

#### Gen fstab

Generate an [fstab](https://wiki.archlinux.org/index.php/Fstab) file (use `-U` or `-L` to define by [**UUID**](https://wiki.archlinux.org/index.php/UUID) or labels, respectively):  
```shell
# genfstab -U /mnt >> /mnt/etc/fstab
```
Check the resulting file in `/mnt/etc/fstab` afterwards, and edit it in case of errors.

Add the following at the end of the `/mnt/etc/fstab` configuration.  
```
/dev/mapper/tmp		/tmp		tmpfs	defaults		0	0
/dev/mapper/swap	none		swap	sw				0	0
/esp/EFI			/boot/EFI	none	defaults,bind 	0	0
```

#### Modify `/mnt/etc/crypttab`

Add these at the end to the `/mnt/etc/crypttab`.  
```
swap	/dev/mapper/MyVol-swap	/dev/urandom		swap,cipher=aes-xts-plain64,size=256
tmp		/dev/mapper/MyVol-tmp	/dev/urandom		tmp,cipher=aes-xts-plain64,size=256
home	/dev/mapper/MyVol-home	/etc/luks-keys/home
```

#### Chroot

[Change root](https://wiki.archlinux.org/index.php/Change_root) into the new system:  
```shell
# arch-chroot /mnt
```

#### Uncomment pacman multilib option

In `/etc/pacman.conf` uncomment the following lines.  
```
[...]
[multilib]
Include = /etc/pacman.d/mirrorlist
[...]
```

#### Install Packages 

```shell
pacman -Sy
pacman -S base-devel grub efibootmgr dialog networkmanager network-manager-applet wireless_tools intel-ucode zsh w3m vim powertop bc
```

#### Create my User

```shell
useradd -m -G wheel,audio,video,users,uucp,disk,optical,storage,rfkill -s /bin/zsh phg

passwd phg
```

#### Disable root login

```shell
# passwd -l root
```

#### Allow group wheel so user sudo

Uncomment `%wheel ALL=(ALL) ALL`. Use `visudo` to edit the `/etc/sudoers` file.

##### Keep `http_proxy` variables

Add the following at the end of the 'Defaults' section.

```
Defaults env_keep += "http_proxy"
Defaults env_keep += "https_proxy"
```

#### Install yaourt

Zuerst muss das Paket [package-query](https://aur.archlinux.org/packages/package-query) installiert werden, denn yaourt beruht darauf:  
```shell
# curl -O https://aur.archlinux.org/cgit/aur.git/snapshot/package-query.tar.gz
# tar -xvzf package-query.tar.gz
# cd package-query
# sudo -u phg makepkg -si
```

Danach wechselt man wieder ins nächst höhere Verzeichnis und installiert yaourt:  
```shell
# cd ..
# curl -O https://aur.archlinux.org/cgit/aur.git/snapshot/yaourt.tar.gz
# tar -xvzf yaourt.tar.gz
# cd yaourt
# sudo -u phg makepkg -si
```

#### Time zone

Set the time zone:  
```shell
# ln -sf /usr/share/zoneinfo/Europe/Berlin /etc/localtime
```

Run hwclock(8) to generate /etc/adjtime:  
```shell
# hwclock --systohc
```

This command assumes the hardware clock is set to UTC. See Time#Time standard for details.

Enable `timedated` service:  
```
# systemctl enable sysdemd-timesyncd
``` 

#### Locale

Uncomment the following [localizations](https://wiki.archlinux.org/index.php/Localization) in `/etc/locale.gen`:  
```
[...]
de_DE.UTF-8 UTF-8
de_DE ISO-8859-1
[...]
en_US.UTF-8 UTF-8
en_US ISO-8859-1
[...]
```

and generate them with:  
```shell
# locale-gen
```

Set the `LANG` variable in [`locale.conf(5)`](http://jlk.fjfi.cvut.cz/arch/manpages/man/locale.conf.5) accordingly, for example:  
`/etc/locale.conf`  
```
LANG=en_US.UTF-8
LANGUAGE=en_US:en
LC_TIME=de_DE.UTF-8
LC_CTYPE=en_US.UTF-8
LC_COLLATE=C
LC_MONETARY=de_DE.UTF-8
LC_MESSAGES=en_US.UTF-8
LC_PAPER=de_DE.UTF-8
LC_NAME=de_DE.UTF-8
LC_ADDRESS=de_DE.UTF-8
LC_TELEPHONE=de_DE.UTF-8
LC_MEASUREMENT=de_DE.UTF-8
LC_IDENTIFICATION=de_DE.UTF-8
```

#### Setup console

##### Install console powerline font

Install the console powerline fonts.
```shell
# yaourt -S powerline-console-fonts ttf-ms-fonts
```

Edit the `/etc/vconsole.conf` file and add the following to the *TOP* of the file:
```
FONT=ter-powerline-v14n
[...]
```

##### Set keymap

If you [set the keyboard](https://wiki.archlinux.org/index.php/Installation_guide#Set_the_keyboard_layout) layout, make the changes persistent in [`vconsole.conf(5)`](http://jlk.fjfi.cvut.cz/arch/manpages/man/vconsole.conf.5):  
`/etc/vconsole.conf`  
```shell
[...]
KEYMAP=de-latin1-nodeadkeys
```

#### Hostname

Create the [`hostname(5)`](http://jlk.fjfi.cvut.cz/arch/manpages/man/hostname.5) file:  
`/etc/hostname`
```
yoetunheimr
```

Consider adding a matching entry to [hosts(5)](http://jlk.fjfi.cvut.cz/arch/manpages/man/hosts.5):  
```
/etc/hosts

127.0.0.1	localhost.localdomain	localhost
::1		localhost.localdomain	localhost
127.0.1.1	yoetunheimr.sao.local	yoetunheimr
```

See also [Network configuration#Set the hostname](https://wiki.archlinux.org/index.php/Network_configuration#Set_the_hostname).

#### Edit mkinitpico

Add the `keyboard`, `keymap`, `lvm2` and `encrypt` hooks to [`mkinitcpio.conf`](https://wiki.archlinux.org/index.php/Mkinitcpio.conf):

`/etc/mkinitcpio.conf`:  
```shell
HOOKS=(... keyboard keymap block lvm2 encrypt ... filesystems ...)
```

#### Generate initramfs

```shell
# mkinitcpio -p linux
```

#### Install GRUB

```shell
# grub-install --target=x86_64-efi --efi-directory=/esp --bootloader-id=grub
```

```shell
# mount --bind /esp/EFI /boot/EFI
```

##### Configuring the boot loader

In order to unlock the encrypted root partition at boot, the following kernel parameters need to be set by the boot loader (`/etc/default/grub`):  
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet acpi_rev_override=1 pci=nommconf"
GRUB_CMDLINE_LINUX="cryptdevice=/dev/mapper/osvg-root:root root=/dev/mapper/root"
```

??? Note 'Command explanation'
	* `acpi_rev_override=1` is needed to get the NVIDIA graphics card working resp. to disable it.
	* The kernel option `pci=nommconf` disables Memory-Mapped PCI Configuration Space, which is available in Linux since kernel 2.6. Very roughly, all PCI devices have an area that describe this device (which you see with lspci -vv), and the originally method to access this area involves going through I/O ports, while PCIe allows this space to be mapped to memory for simpler access.
	* `cryptdevice=/dev/mapper/osvg-root:root root=/dev/mapper/root` configures the crypt device.

##### Generate GRUB config

```
# grub-mkconfig -o /boot/grub/grub.cfg
```

See [Dm-crypt/System configuration#Boot](https://wiki.archlinux.org/index.php/Dm-crypt/System_configuration#Boot_loader) loader for details.

#### Reboot into the installed system

Leave the `chroot` environment.
```shell
exit
```

Unmount all partitions and reboot.
```shell
# umount -R /mnt
# reboot
```

## Post installation configuration

### Get graphicscard and X working

#### ACPI Override

Add these to the kernel parameter in the `/etc/default/grub` configuration.
```
GRUB_CMDLINE_LINUX_DEFAULT="[...] acpi_rev_override=1"
```

Now rebuild the GRUB configuration.
```shell
# grub-mkconfig -o /boot/grub/grub.cfg
```

#### Install graphic card tools

```shell
# pacman -S bbswitch bumblebee primus lib32-primus
```
Use `libglvnd` as libgl provider.

Enable the bumblebee service.  
```shell
# systemctl enable bumblebeed.service
```

Afterwards `reboot` the system.

!!! Tip
	You may need to reboot twice for the firmware to notice `acpi_rev_override`.

#### Graphics card drivers/utils

```shell
# yaourt -Sy
# yaourt -S nvidia-beta nvidia-utils-beta lib32-nvidia-utils-beta
```

#### Install X

```shell
# pacman -S xorg xorg-xinit xf86-input-synaptics
```

#### Install login manager

```shell
# pacman -S lightdm
# yaourt -S lightdm-webkit-theme-litarvan
# systemctl enable lightdm
```

##### Enable Webkit2 greeter

Create the `lightdm.conf.d` folder:  
```shell
# mkdir /etc/lightdm/lightdm.conf.d/
```

Set the default greeter by adding the following LightDM configuration file, like so:

`/etc/lightdm/lightdm.conf.d/50-lightdm-webkit2-greeter.conf`  
```
[Seat:*]
greeter-session=lightdm-webkit2-greeter
```

##### LightDM Webkit2 theme 'litarvan'

Edit the file `/etc/lightdm/lightdm-webkit2-greeter.conf` and set the "`webkit-theme`" property to "`litarvan`", like so:
```
[...]
webkit_theme	= litarvan
[...]
```

##### BSPWM Session

#### Change X keyboard layout to german

`/etc/X11/xorg.conf.d/00-keyboard.conf`
```
Section "InputClass"
	Identifier "keyboard"
	MatchIsKeyboard "yes"
	Option "XkbLayout" "de"
	Option "XkbVariant" "nodeadkeys"
EndSection
```

#### Add Touchpad options

`/etc/X11/xorg.conf.d/50-synaptics.conf`:
```
Section "InputClass"
	Identifier "touchpad catchall"
	Driver "synaptics"
	MatchIsTouchpad "on"
	# enable clik zone and configure 3 buttons on bottom
	Option "ClickPad" "1"
	Option "SoftButtonAreas" "60% 0 82% 0 40% 60% 82% 0"
	# other commons options than you may want to configure
	# scroll with two fingers (enabled vertically, disabled horizontally)
	Option "VertTwoFingerScroll" "1"
	Option "HorizTwoFingerScroll" "0"
	# enable natural scrolling
	Option "VertScrollDelta" "-111"
	Option "HorizScrollDelta" "-111"
	# enable tap as click: 1 finger -> left button, 2 fingers -> right, 3 fingers -> middle
	Option "TapButton1" "1"
	Option "TapButton2" "3"
	Option "TapButton3" "2"
	# idem but for click with 1,2,3 fingers. Use "0" to disable. 
	Option "ClickFinger1" "1"
	Option "ClickFinger2" "3"
	Option "ClickFinger3" "2"
	# palm detection. These parameters somehow works, YMMV. 
	Option "PalmDetect" "1"
	Option "PalmMinWidth" "10"
	Option "PalmMinZ" "200"
EndSection
```

#### Install xorg-xbacklight replacement

!!! Note
	`xorg-xbacklight` does not work with the modesetting driver!

```shell
# yaourt -S acpilight
```

Add the following file `/etc/udev/rules.d/90-backlight.rules`:
```
SUBSYSTEM=="backlight", ACTION=="add", \
  RUN+="/bin/chgrp video %S%p/brightness", \
  RUN+="/bin/chmod g+w %S%p/brightness"
```

#### Window manager

Install bspwm, sxhkd, feh and fonts + dependencies.
```shell
# pacman -S bspwm sxhkd feh i3lock
# yaourt -S polybar
```

### Get my dot files
Clone my dotfiles.
```shell
# pacman -Sy
# pacman -S git python-pip
# cd ~
# git clone git@github.com:shokinn/.files.git ~/.files
# cd .files
# sudo pip install -r dotdrop/requirements.txt
# alias dotdrop='eval $(grep -v "^#" ~/dotfiles/.env.public) ~/dotfiles/dotdrop.sh'
# dotdrop install
```

### Enable lockscreen at suspend

1. Create a service file
`/etc/systemd/system/i3lock@.service`:  
```
[Unit]
Description=Lock the screen on resume from suspend
Before=sleep.target

[Service]
User=%I
Type=forking
Environment=DISPLAY=:0
ExecStart=/home/%I/.config/bspwm/lock.sh

[Install]
WantedBy=suspend.target
```
2. Enable the service
```shell
# systemctl daemon-reload
# systemctl enable i3lock@phg
```

### Make powertop optimazations permanent

`/etc/systemd/system/powertop.service`:
```shell
[Unit]
Description=PowerTOP auto tune

[Service]
Type=idle
Environment="TERM=dumb"
ExecStart=/usr/sbin/powertop --auto-tune

[Install]
WantedBy=multi-user.target
```

```shell
# systemctl daemon-reload
# systemctl enable powertop.service
```

### Enable network manager

```shell
# systemctl enable NetworkManager
```

### Install packages

```shell
# pacman -S firefox \
alsa-utils \
alsa-tools \
alsa-plagins \
pulseaudio \
pulseaudio-alsa \
exfat-utils \
openssh \
net-tools \
keepass \
keepass-plugin-keeagent \
inkscape \
gimp \
vlc \
qt4 \
libsecret \
gnome-keyring \
libgnome-keyring \
thunderbird \
pinentry \
gpa \
dmenu \
mc \
nemo \
nemo-fileroller \
nemo-image-converter \
nemo-preview \
nemo-seahorse \
nemo-share \
linux-headers \
wireguard-dkms \
wireguard-tools \
wmname

# gpg --recv-keys --keyserver sks-keyservers.net 0xDB1187B9DD5F693B

# yay -S tldr \
keepass-plugin-rpc \
keepass-plugin-haveibeenpwned \
keepass-plugin-http \
keepass2-plugin-tray-icon \
nextcloud-client-git \
franz-bin \
thunderbird-enigmail \
gtkhash-nemo \
nemo-compare
```

### Enable SSH-Agent Serive

```shell
# systemctl --user daemon-reload
# systemctl --user enable ssh-agent.service
# systemctl --user start ssh-agent.service
```

# TODO
* Crypto fixen
  * Use an Hardware device for 2nd Factor authentication
  * Maybe use TOTP as 2nd Factor?
* Audio (Mic)
  * Get external Microphones working!