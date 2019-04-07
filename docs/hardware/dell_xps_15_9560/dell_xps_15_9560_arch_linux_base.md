# Arch Linux Dell XPS 15 (9560) - Base Arch 

## Pre-installation

### UEFI

Before installing it is necessary to modify some UEFI Settings. They can be accessed by pressing the F2 key repeatedly when booting.

* Change the SATA Mode from the default "RAID" to "AHCI". This will allow Linux to detect the NVME SSD. If dual booting with an existing Windows installation, Windows will not boot after the change but this can be fixed without a reinstallation.
* Change Fastboot to "Thorough" in "POST Behaviour". This prevents intermittent boot failures.
* Disable secure boot to allow Linux to boot.

Installation of Arch Linux can proceed normally. Refer to the [Installation guide](https://wiki.archlinux.org/index.php/Installation_guide) for more information. 

### Change Keyboard layout

If you don't have an EN:Intl. Keyboard layout you should change it to your layout. In my case it is `de-latin1-nodeadkeys1`.

```bash
loadkeys de-latin1-nodeadkeys
```

### Connect to a wireless network during installation (temporary)

Run `wifi-menu` to get a graphical interface to connect to your wifi.

```bash
wifi-menu
```

Afterwards you establish the connection change back to TTY1 with `ALT`+`F1`.

Run dhclient to receive an ip via dhcp:  
```bash
dhclient wlp2s0
```

### Update the system clock

Use [timedatectl(1)](http://jlk.fjfi.cvut.cz/arch/manpages/man/timedatectl.1) to ensure the system clock is accurate:  
```bash
timedatectl set-ntp true
systemctl restart systemd-timesyncd
```

To check the service status, use timedatectl status.

## Installation

### Preparing Disks

I'll encrypt my Data so my Partition layout looks like this:

```
+----------------+----------------+-----------------------------------------------------------------------------------------------+
|                |                |                       |                       |                       |                       |
|                |                | LUKS encrypted volume | LUKS encrypted volume | LUKS encrypted volume | LUKS encrypted volume |
|                |                | /dev/mapper/swap      | /dev/mapper/tmp       | /dev/mapper/root      | /dev/mapper/home      |
|                |                |_ _ _ _ _ _ _ _ _ _ _ _|_ _ _ _ _ _ _ _ _ _ _ _|_ _ _ _ _ _ _ _ _ _ _ _|_ _ _ _ _ _ _ _ _ _ _ _|
|                |                |                       |                       |                       |                       |
|                |                |         16 GiB        |         30 GiB        |         50 GiB        |        100%FREE       |
|                |                |    Logical volume1    |    Logical volume2    |    Logical volume3    |    Logical volume4    |
|                |                | /dev/mapper/osvg-swap | /dev/mapper/osvg-tmp  | /dev/mapper/osvg-root | /dev/mapper/osvg-home |
|                |      EF02      |_ _ _ _ _ _ _ _ _ _ _ _|_ _ _ _ _ _ _ _ _ _ _ _|_ _ _ _ _ _ _ _ _ _ _ _|_ _ _ _ _ _ _ _ _ _ _ _|
|      EF00      |      8 MiB     |                                                                                               |
|    1050 MiB    |      BIOS      |                                             8E00                                              |
|  EFI partition | Boot partition |                                           100%FREE                                            |
| /dev/nvme0n1p1 | /dev/nvme0n1p2 |                                        /dev/nvme0n1p3                                         |
+----------------+----------------+-----------------------------------------------------------------------------------------------+
```

Create these partitions with `gdisk` so that you have `GPT` instead of `MBR`.

!!! info "Old partition table"
	If you get the message that the kernel is still using the old partition table run:  
	```bash
	partprobe /dev/nvme0n1
	```
  If you still get the message the the kernel is using the old partition table you have to reboot your device and perform all steps except the partitioning again.

#### Wipe Data on all partitions

```bash
dd if=/dev/zero of=/dev/nvme0n1p1 bs=1M status=progress
dd if=/dev/zero of=/dev/nvme0n1p2 bs=1M status=progress
dd if=/dev/uransom of=/dev/nvme0n1p3 bs=1M status=progress
```

#### Preparing the logical volumes

```bash
pvcreate /dev/nvme0n1p3
vgcreate osvg /dev/nvme0n1p3
lvcreate -L 16G -n swap osvg
lvcreate -L 30G -n tmp osvg
lvcreate -L 50G -n root osvg
lvcreate -l 100%FREE -n home osvg
```

#### Setup `LUKS` for our partitions

Create an encrypted root with a key you can remember.  
```bash
cryptsetup luksFormat /dev/mapper/osvg-root
```

Open the root filesystem to create an encryption key for all other partitions.  
```bash
cryptsetup luksOpen /dev/mapper/osvg-root root
mkfs.ext4 /dev/mapper/root
mount /dev/mapper/root /mnt
mkdir -pm 700 /mnt/etc/luks-keys
dd if=/dev/random of=/mnt/etc/luks-keys/home bs=1 count=1024 status=progress
chmod 000 /mnt/etc/luks-keys/home
```

Use the encrypted key to create our encrypted home and add a pass phrase for emergency access:  
```bash
cryptsetup luksFormat /dev/mapper/osvg-home
cryptsetup luksAddKey /dev/mapper/osvg-home /mnt/etc/luks-keys/home
```

Now we open home, make a filesystem and mount it to `/mnt/home`.  
```bash
cryptsetup luksOpen --key-file=/mnt/etc/luks-keys/home /dev/mapper/osvg-home home
mkfs.ext4 /dev/mapper/home
mkdir /mnt/home
mount /dev/mapper/home /mnt/home
```

and rebuild the EFI and boot partition:  
```bash
mkfs.fat -F32 /dev/nvme0n1p1
```

Setup and mount the boot/EFI partition:  
```bash
mkdir /mnt/boot
mkdir /mnt/efi
mount /dev/nvme0n1p1 /mnt/efi
mkdir -p /mnt/efi/EFI/arch
mount --bind /mnt/efi/EFI/arch /mnt/boot
```

### Install Arch

#### Sort mirrors by speed

The [pacman](https://www.archlinux.org/packages/?name=pacman) package provides a Bash script, `/usr/bin/rankmirrors`, which can be used to rank the mirrors according to their connection and opening speeds to take advantage of using the fastest local mirror.  
Back up the existing `/etc/pacman.d/mirrorlist`:  
```bash
cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.bak
```

Download a german mirrorlist:  

??? info "Full URL"
    <https://www.archlinux.org/mirrorlist/?country=DE&protocol=http&protocol=https&ip_version=4&ip_version=6&use_mirror_status=on>

```bash
curl -L -o /etc/pacman.d/mirrorlist "https://bit.ly/1yFCU2R"
```

Edit `/etc/pacman.d/mirrorlist.bak` and uncomment mirrors for testing with `rankmirrors`.  
Run the following `sed` line to uncomment every mirror:  
```bash
sed -i 's/^#Server/Server/' /etc/pacman.d/mirrorlist
```

#### Install the base packages

Use the [pacstrap](https://projects.archlinux.org/arch-install-scripts.git/tree/pacstrap.in) script to install the [`base`](https://www.archlinux.org/groups/x86_64/base/) package group:  
```bash
pacstrap /mnt base base-devel intel-ucode wpa_supplicant dialog
```
This group does not include all tools from the live installation, such as [`btrfs-progs`](https://www.archlinux.org/packages/?name=btrfs-progs) or specific wireless firmware; see [packages.both](https://projects.archlinux.org/archiso.git/tree/configs/releng/packages.both) for comparison.
To [install](#Install_Arch) packages and other groups such as [`base-devel`](https://www.archlinux.org/groups/x86_64/base-devel/), append the names to *pacstrap* (space separated) or to individual [pacman](https://wiki.archlinux.org/index.php/Pacman) commands after the [#Chroot](#Chroot) step.

### Configure the system

#### Gen fstab

Generate an [fstab](https://wiki.archlinux.org/index.php/Fstab) file (use `-U` or `-L` to define by [**UUID**](https://wiki.archlinux.org/index.php/UUID) or labels, respectively):  
```bash
genfstab -U /mnt >> /mnt/etc/fstab
```
Check the resulting file in `/mnt/etc/fstab` afterwards, and edit it in case of errors.

Add the following at the end of the `/mnt/etc/fstab` configuration Adjust the 30G to the size of your /tmp partition.  
```
/dev/mapper/tmp		/tmp		tmpfs	defaults,size=30G		0	0
/dev/mapper/swap	none		swap	sw			            0	0
```

#### Modify `/mnt/etc/crypttab`

Add these at the end to the `/mnt/etc/crypttab`.  
```
swap	/dev/mapper/osvg-swap	/dev/urandom		swap,cipher=aes-xts-plain64,size=256
tmp		/dev/mapper/osvg-tmp	/dev/urandom		tmp,cipher=aes-xts-plain64,size=256
home	/dev/mapper/osvg-home	/etc/luks-keys/home
```

#### Chroot

[Change root](https://wiki.archlinux.org/index.php/Change_root) into the new system:  
```bash
arch-chroot /mnt
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

```bash
pacman -Sy
pacman -S grub efibootmgr networkmanager network-manager-applet wireless_tools zsh w3m vim powertop tlp bc git
```

#### Fix vim dark colors stuff

```bash
cat << EOF >> /etc/vimrc

" Set background to dark for better readability in SSH connections
set background=dark
EOF
```

#### set vim to default editor

```bash
cat << EOF >> /etc/environment
EDITOR=vim
EOF
```

#### Create my User

```bash
useradd -m -G wheel,audio,video,users,uucp,disk,optical,storage,rfkill -s /bin/zsh phg

passwd phg
```

#### Disable root login

```bash
passwd -l root
```

#### Allow group wheel so user sudo

Uncomment `%wheel ALL=(ALL) ALL`. Use `visudo` to edit the `/etc/sudoers` file.

##### Keep `http_proxy` variables

Add the following at the end of the 'Defaults' section.

```
Defaults env_keep += "http_proxy"
Defaults env_keep += "https_proxy"
```

#### Install yay

Clone the yay sources. Build, install and delete it.  
```bash
git clone https://aur.archlinux.org/yay.git
chown phg:phg yay
cd yay
sudo -u phg makepkg -si
cd ..
rm -rf yay
```

#### Time zone

Set the time zone:  
```bash
ln -sf /usr/share/zoneinfo/Europe/Berlin /etc/localtime
```

Run hwclock(8) to generate /etc/adjtime:  
```bash
hwclock --systohc
```

This command assumes the hardware clock is set to UTC. See Time#Time standard for details.

Enable `timedated` service:  
```
systemctl enable systemd-timesyncd
``` 

#### Locale

Uncomment the following [localizations](https://wiki.archlinux.org/index.php/Localization) in `/etc/locale.gen`:  
```
[...]
de_DE.UTF-8 UTF-8
de_DE ISO-8859-1
de_DE@euro ISO-8859-15
[...]
en_US.UTF-8 UTF-8
en_US ISO-8859-1
[...]
```

and generate them with:  
```bash
locale-gen
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
```bash
sudo -u phg yay -S powerline-console-fonts ttf-ms-fonts ttf-dejavu
```

Edit the `/etc/vconsole.conf` file and add the following to the *TOP* of the file:
```
FONT=ter-powerline-v14n
[...]
```

##### Set keymap

If you [set the keyboard](https://wiki.archlinux.org/index.php/Installation_guide#Set_the_keyboard_layout) layout, make the changes persistent in [`vconsole.conf(5)`](http://jlk.fjfi.cvut.cz/arch/manpages/man/vconsole.conf.5):  
`/etc/vconsole.conf`  
```bash
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
`HOOKS=(base udev autodetect `**`keyboard keymap`**` consolefont modconf block `**`encrypt lvm2 `**`filesystems fsck)`

#### Generate initramfs

```bash
mkinitcpio -p linux
```

#### Install GRUB

```bash
grub-mkconfig -o /boot/grub/grub.cfg
grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=grub
```

##### Configuring the boot loader

In order to unlock the encrypted root partition at boot, the following kernel parameters need to be set by the boot loader (`/etc/default/grub`):  
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet pcie_aspm=off acpi_rev_override=1 modprobe.blacklist=nouveau nouveau.modeset=0 i915.modeset=1 enable_fbc=1 enable_psr=1 disable_power_well=0 pcie_port_pm=off"
GRUB_CMDLINE_LINUX="cryptdevice=/dev/mapper/osvg-root:root root=/dev/mapper/root"
```

??? info "Command explanation"
    * `pcie_aspm=off` Forcibly enable or disable PCIe Active State Power Management.
    * `acpi_rev_override=1` is needed to get the NVIDIA graphics card working resp. to disable it.
    * `modprobe.blacklist=nouveau` blacklists nouveau driver (open source NVIDIA driver).
    * `nouveau.modeset=0` disables modeset for nouveau (just in case).
    * `i915.modeset=1` enables modeset for the i915 module (integrated intel graphics).
    * `enable_fbc=1` enables frame buffer compression.
    * `enable_psr=1` enables panel self refresh.
    * `disable_power_well=0` Try to disable possible power wells. 
    * The kernel option `pci=nommconf` disables Memory-Mapped PCI Configuration Space, which is available in Linux since kernel 2.6. Very roughly, all PCI devices have an area that describe this device (which you see with lspci -vv), and the originally method to access this area involves going through I/O ports, while PCIe allows this space to be mapped to memory for simpler access (currently unused, stays in documentation for documentation purposes).
    * `cryptdevice=/dev/mapper/osvg-root:root root=/dev/mapper/root` configures the crypt device.

##### Generate GRUB config

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

See [Dm-crypt/System configuration#Boot](https://wiki.archlinux.org/index.php/Dm-crypt/System_configuration#Boot_loader) loader for details.

#### Reboot into the installed system

Leave the `chroot` environment.
```bash
exit
```

Unmount all partitions and reboot.
```bash
umount -R /mnt
reboot
```

##### Modify UEFI

Open UEFI Configuration menu. (F12 -> Setup)
Go to:
```
- General
 | - Boot Sequence
    \ Boot List Option
     \ Add Boot Option
      | - Boot Option Name
      |  \ "Linux"
      | - File System List
      |  \ PciRoot(0x0)/[...]
      | - File Name
      |  \ "\EFI\grub\grubx64.efi"
```

Afterwards set "Linux" in the Boot Sequence to the top.

## Post installation configuration

### Install useful services

```bash
pacman -S acpid dbus avahi cups cronie
systemctl enable acpid avahi-daemon org.cups.cupsd.service cronie
```

### Get graphics card and X working

#### Install graphic card tools

!!! Tip "Run graphics accelerated applications"
    ??? Info "Steam launch command explanation"
        - `vblank_mode=0` - This let's render your graphics card faster then your screen refresh rate.
        - `primusrun` - Use primusrun to run the program on the second graphics card.
        - `%command%` - Runs the start command which steam uses to launch the game.
    **Steam:**  
    1. Right click on `$GAME` in your Steam library.  
    2. `SET LAUNCH OPTIONS...`  
    3. Add the following command.  
    ```bash
    vblank_mode=0 primusrun %command%
    ```

    **Other programs:**  
    ```bash
    vblank_mode=0 primusrun <program>
    ```


```bash
sudo pacman -S bbswitch bumblebee primus lib32-primus
```
Use `libglvnd` as libgl provider.

Replace (or comment) the `Bridge=auto` option with:  
`/etc/bumblebee/bumblebee.conf`:  
```bash
Bridge=primus
```

Enable the bumblebee service.  
```bash
sudo systemctl enable bumblebeed.service
```

Add user to bumblebee group.  
```bash
sudo usermod -aG bumblebee phg
```

#### Graphics card drivers/utils

```bash
yay -Sy
yay -S nvidia nvidia-utils nvidia-xrun lib32-nvidia-utils
```

nvidia-xrun config:  
```bash
Section "Device"
    Identifier "nvidia"
    Driver "nvidia"
    BusID "PCI:1:0:0"
EndSection
Section "Screen"
    Identifier "nvidia"
    Device "nvidia"
    #  Option "AllowEmptyInitialConfiguration" "Yes"
    #  Option "UseDisplayDevice" "none"
EndSection
```

### Get my dot files
Clone my dotfiles.
```bash
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
sudo pacman -Sy
sudo pacman -S python-pip
cd ~
git clone --recurse https://github.com/shokinn/.files.git ~/.files
cd .files
git remote -v
git remote set-url origin git@github.com:shokinn/.files.git
pip install --user -r dotdrop/requirements.txt
alias dotdrop='eval $(grep -v "^#" ~/.files/.env.public) ~/.files/dotdrop.sh'
dotdrop install
git clone https://github.com/bhilburn/powerlevel9k.git ~/.oh-my-zsh/custom/themes/powerlevel9k
```

### tlp (powermanagement)

#### tlp service

```bash
systemctl enable tlp.service
```

#### tlp configuration

Uncomment these options and set the variable to these ones:  
`/etc/default/tlp`:  
```bash
[...]
TLP_DEFAULT_MODE=BAT
[...]
RUNTIME_PM_BLACKLIST="01:00.0"
[...]
RUNTIME_PM_ON_AC=auto
```

### Powertop

#### Optimize power comsumption

Run the following command to calibrate your power consumption:  
```bash
sudo powertop --calibrate
```

!!! Note
	Your screen will turn several times black and keep black for a couple minutes!

	Don't worry :)

#### Make powertop optimazations permanent

`/etc/systemd/system/powertop.service`:
```bash
cat <<EOF | sudo tee /etc/systemd/system/powertop.service
[Unit]
Description=PowerTOP auto tune

[Service]
Type=oneshot
ExecStart=/usr/sbin/powertop --auto-tune

[Install]
WantedBy=multi-user.target
EOF
```

!!! Warning "Don't start/enable powertop for now"
    Powertop is breaking bumblebee from Kernel 4.16.13. -- And tlp should also do a good job for saving power.
    See here for more details:  
    <https://bbs.archlinux.org/viewtopic.php?id=238131>

```bash
sudo systemctl daemon-reload
#sudo systemctl start powertop.service
#sudo systemctl enable powertop.service
```

### Install packages

#### Install audio packages

```bash
sudo pacman -S \
alsa-utils \
alsa-oss
```

#### Install various packages

```bash
sudo pacman -S \
exfat-utils \
openssh \
net-tools \
libsecret \
gnome-keyring \
libgnome-keyring \
mc \
htop \
linux-headers \
wireguard-dkms \
wireguard-tools \
rsync \
blueman

yay -S tldr
```

Enable bluetooth service: 
```bash
systemctl enable bluetooth.service
```

### Enable SSH-Agent Serive

```bash
systemctl --user daemon-reload
systemctl --user enable ssh-agent.service
systemctl --user start ssh-agent.service
```

# TODO
* Crypto fixen
    * Use an Hardware device for 2nd Factor authentication
    * Maybe use TOTP as 2nd Factor?