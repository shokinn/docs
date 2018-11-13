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

Start xfce with:  
```shell
echo "exec startxfce4" >> ~/.xinitrc
startx
```

The `.xinitrc` should already be fine because I installed my dot-files.

### Install xscreensaver

1. Install xscreensaver-aerial theme and download the 1080p videos to disk.  
```shell
yay -S xscreensaver-aerial
```

2. Add download script and download the Videos for offline use:  
```shell
sudo mkdir -p /home/sddm/ATV4 && \
sudo ln -s /home/sddm/ATV4 /opt/ && \
sudo touch /home/sddm/ATV4/download.sh && \
cat << EOF | sudo tee /home/sddm/ATV4/download.sh && \
sudo chmod +x /home/sddm/ATV4/download.sh && \
cd /home/sddm/ATV4 && \
sudo ./download.sh && \
cd ~ && \
sudo chmod 700 /home/sddm && \
sudo chown -R sddm:sddm /home/sddm
#!/bin/sh
# run this from /opt/ATV4 which you created and assigned 755 premissions manually

_url1="http://a1.v2.phobos.apple.com.edgesuite.net/us/r1000/000/Features/atv/AutumnResources/videos/"
_url3="http://sylvan.apple.com/Aerials/2x/Videos/"

for i in b2-1.mov b5-1.mov b6-1.mov comp_GL_G010_C006_v08_6Mbps.mov b1-1.mov \
	b2-2.mov b4-1.mov b6-2.mov b7-1.mov b8-1.mov b1-2.mov b3-1.mov b5-2.mov \
	b6-3.mov b1-3.mov b2-3.mov b3-2.mov b4-2.mov b7-2.mov b1-4.mov b2-4.mov \
	b3-3.mov b4-3.mov b5-3.mov b6-4.mov b7-3.mov b8-2.mov b8-3.mov b9-2.mov \
	b9-3.mov b10-3.mov; do
	wget "\$_url1/\$i"
	chmod 644 \$(pwd)/\$i
done

for i in comp_CH_C007_C011_PSNK_v02_SDR_PS_FINAL_20180709_SDR_2K_HEVC.mov \
	comp_CH_C002_C005_PSNK_v05_SDR_PS_FINAL_20180709_SDR_2K_HEVC.mov \
	comp_CH_C007_C004_PSNK_v02_SDR_PS_FINAL_20180709_SDR_2K_HEVC.mov \
	DB_D008_C010_2K_SDR_HEVC.mov DB_D001_C001_2K_SDR_HEVC.mov \
	DB_D011_C010_2K_SDR_HEVC.mov DB_D002_C003_2K_SDR_HEVC.mov \
	DB_D001_C005_2K_SDR_HEVC.mov DB_D011_C009_2K_SDR_HEVC.mov \
	GL_G004_C010_2K_SDR_HEVC.mov GL_G002_C002_2K_SDR_HEVC.mov \
	HK_B005_C011_2K_SDR_HEVC.mov HK_H004_C010_2K_SDR_HEVC.mov \
	HK_H004_C013_2K_SDR_HEVC.mov HK_H004_C001_2K_SDR_HEVC.mov \
	HK_H004_C008_2K_SDR_HEVC.mov \
	comp_GMT312_162NC_139M_1041_AFRICA_NIGHT_v14_SDR_FINAL_20180706_SDR_2K_HEVC.mov \
	comp_A103_C002_0205DG_v12_SDR_FINAL_20180706_SDR_2K_HEVC.mov \
	comp_GMT306_139NC_139J_3066_CALI_TO_VEGAS_v07_SDR_FINAL_22062018_SDR_4K_HEVC.mov \
	comp_A108_C001_v09_SDR_FINAL_22062018_SDR_2K_HEVC.mov \
	comp_GMT308_139K_142NC_CARIBBEAN_DAY_v09_SDR_FINAL_22062018_SDR_2K_HEVC.mov \
	comp_GMT329_113NC_396B_1105_CHINA_v04_SDR_FINAL_20180706_F900F2700_SDR_2K_HEVC.mov \
	comp_A083_C002_1130KZ_v04_SDR_PS_FINAL_20180725_SDR_2K_HEVC.mov \
	comp_GMT329_117NC_401C_1037_IRELAND_TO_ASIA_v48_SDR_PS_FINAL_20180725_F0F6300_SDR_2K_HEVC.mov \
	comp_GMT026_363A_103NC_E1027_KOREA_JAPAN_NIGHT_v17_SDR_FINAL_25062018_SDR_2K_HEVC.mov \
	comp_A105_C003_0212CT_FLARE_v10_SDR_PS_FINAL_20180711_SDR_2K_HEVC.mov \
	comp_A009_C001_010181A_v09_SDR_PS_FINAL_20180725_SDR_2K_HEVC.mov \
	comp_A114_C001_0305OT_v10_SDR_FINAL_22062018_SDR_2K_HEVC.mov \
	comp_A001_C004_1207W5_v23_SDR_FINAL_20180706_SDR_2K_HEVC.mov \
	LA_A006_C008_2K_SDR_HEVC.mov LA_A009_C009_2K_SDR_HEVC.mov LA_A008_C004_2K_SDR_HEVC.mov \
	comp_LA_A006_C004_v01_SDR_FINAL_PS_20180730_SDR_2K_HEVC.mov LA_A005_C009_2K_SDR_HEVC.mov \
	LA_A011_C003_2K_SDR_HEVC.mov; do
	wget "\$_url3/\$i"
  chmod 644 \$(pwd)/\$i
done
EOF
```

3. Run `xscreensaver-demo` and close it afterwards. This will create a `.xscreensaver` configuration file in your home directory.

4. Edit `~/.xscreensaver` to add support for it to see this script. Look for the line that beings with "programs" and simply add the following to the file:  
```shell
"ATV4-1080" atv4-1080 -root \n\
"ATV4-4k" atv4-4k -root \n\
```

### Install login manager

Install sddm:  
```shell
sudo pacman -S gst-libav \
phonon-qt5-gstreamer \
gst-plugins-good \
qt5 \
plasma \
sddm
yay -S sddm-config-editor-git
```

Install Aerial theme:  
```shell
sudo git clone https://github.com/3ximus/aerial-sddm-theme.git /usr/share/sddm/themes/aerial-sddm-theme
```

Use local files for videos:  
```shell
cat << EOF > /tmp/create_playlists.sh && \
sudo /bin/sh /tmp/create_playlists.sh
#!/bin/sh

path_vid='/home/$USER/Videos/ATV4'
path_playlist='/usr/share/sddm/themes/aerial-sddm-theme/playlists'

echo "" > \$path_playlist/day.m3u
for i in b1-1.mov b1-3.mov b2-1.mov b2-2.mov b3-2.mov b3-3.mov b4-1.mov b4-2.mov \
b5-1.mov b5-2.mov b6-1.mov b6-3.mov b7-1.mov b7-2.mov b8-2.mov b8-3.mov b9-1.mov \
b9-3.mov b10-1.mov b10-3.mov comp_CH_C002_C005_PSNK_v05_SDR_PS_FINAL_20180709_SDR_2K_HEVC.mov \
comp_CH_C007_C004_PSNK_v02_SDR_PS_FINAL_20180709_SDR_2K_HEVC.mov \
comp_CH_C007_C011_PSNK_v02_SDR_PS_FINAL_20180709_SDR_2K_HEVC.mov comp_GL_G010_C006_v08_6Mbps.mov \
comp_LA_A006_C004_v01_SDR_FINAL_PS_20180730_SDR_2K_HEVC.mov \
DB_D001_C001_2K_SDR_HEVC.mov DB_D001_C005_2K_SDR_HEVC.mov DB_D002_C003_2K_SDR_HEVC.mov \
DB_D008_C010_2K_SDR_HEVC.mov DB_D011_C009_2K_SDR_HEVC.mov DB_D011_C010_2K_SDR_HEVC.mov \
GL_G002_C002_2K_SDR_HEVC.mov GL_G004_C010_2K_SDR_HEVC.mov HK_B005_C011_2K_SDR_HEVC.mov \
HK_H004_C001_2K_SDR_HEVC.mov HK_H004_C008_2K_SDR_HEVC.mov HK_H004_C010_2K_SDR_HEVC.mov \
HK_H004_C013_2K_SDR_HEVC.mov LA_A005_C009_2K_SDR_HEVC.mov LA_A006_C008_2K_SDR_HEVC.mov \
LA_A008_C004_2K_SDR_HEVC.mov LA_A009_C009_2K_SDR_HEVC.mov \
comp_A001_C004_1207W5_v23_SDR_FINAL_20180706_SDR_2K_HEVC.mov \
comp_A009_C001_010181A_v09_SDR_PS_FINAL_20180725_SDR_2K_HEVC.mov \
comp_A083_C002_1130KZ_v04_SDR_PS_FINAL_20180725_SDR_2K_HEVC.mov \
comp_A103_C002_0205DG_v12_SDR_FINAL_20180706_SDR_2K_HEVC.mov \
comp_A105_C003_0212CT_FLARE_v10_SDR_PS_FINAL_20180711_SDR_2K_HEVC.mov \
comp_A108_C001_v09_SDR_FINAL_22062018_SDR_2K_HEVC.mov \
comp_A114_C001_0305OT_v10_SDR_FINAL_22062018_SDR_2K_HEVC.mov \
comp_GMT026_363A_103NC_E1027_KOREA_JAPAN_NIGHT_v17_SDR_FINAL_25062018_SDR_2K_HEVC.mov \
comp_GMT306_139NC_139J_3066_CALI_TO_VEGAS_v07_SDR_FINAL_22062018_SDR_4K_HEVC.mov \
comp_GMT308_139K_142NC_CARIBBEAN_DAY_v09_SDR_FINAL_22062018_SDR_2K_HEVC.mov \
comp_GMT312_162NC_139M_1041_AFRICA_NIGHT_v14_SDR_FINAL_20180706_SDR_2K_HEVC.mov \
comp_GMT329_113NC_396B_1105_CHINA_v04_SDR_FINAL_20180706_F900F2700_SDR_2K_HEVC.mov \
comp_GMT329_117NC_401C_1037_IRELAND_TO_ASIA_v48_SDR_PS_FINAL_20180725_F0F6300_SDR_2K_HEVC.mov; do
	echo "\$path_vid/\$i" >> \$path_playlist/day.m3u
done

echo "" > \$path_playlist/night.m3u
for i in b1-2.mov b1-4.mov b2-3.mov b2-4.mov b3-1.mov b4-2.mov b5-3.mov \
b6-2.mov b6-4.mov b7-3.mov b10-4.mov b9-2.mov b10-2.mov b8-1.mov \
LA_A011_C003_2K_SDR_HEVC.mov \
comp_A001_C004_1207W5_v23_SDR_FINAL_20180706_SDR_2K_HEVC.mov \
comp_A009_C001_010181A_v09_SDR_PS_FINAL_20180725_SDR_2K_HEVC.mov \
comp_A083_C002_1130KZ_v04_SDR_PS_FINAL_20180725_SDR_2K_HEVC.mov \
comp_A103_C002_0205DG_v12_SDR_FINAL_20180706_SDR_2K_HEVC.mov \
comp_A105_C003_0212CT_FLARE_v10_SDR_PS_FINAL_20180711_SDR_2K_HEVC.mov \
comp_A108_C001_v09_SDR_FINAL_22062018_SDR_2K_HEVC.mov \
comp_A114_C001_0305OT_v10_SDR_FINAL_22062018_SDR_2K_HEVC.mov \
comp_GMT026_363A_103NC_E1027_KOREA_JAPAN_NIGHT_v17_SDR_FINAL_25062018_SDR_2K_HEVC.mov \
comp_GMT306_139NC_139J_3066_CALI_TO_VEGAS_v07_SDR_FINAL_22062018_SDR_4K_HEVC.mov \
comp_GMT308_139K_142NC_CARIBBEAN_DAY_v09_SDR_FINAL_22062018_SDR_2K_HEVC.mov \
comp_GMT312_162NC_139M_1041_AFRICA_NIGHT_v14_SDR_FINAL_20180706_SDR_2K_HEVC.mov \
comp_GMT329_113NC_396B_1105_CHINA_v04_SDR_FINAL_20180706_F900F2700_SDR_2K_HEVC.mov \
comp_GMT329_117NC_401C_1037_IRELAND_TO_ASIA_v48_SDR_PS_FINAL_20180725_F0F6300_SDR_2K_HEVC.mov; do
	echo "\$path_vid/\$i" >> \$path_playlist/night.m3u
done
EOF
```

Enable SDDM service, delete xfce from the `.xinitrc` and reboot:  
```shell
systemctl enable sddm.service
sed -i '$d' ~/.xinitrc
reboot
```

### Install bspwm, sxhkd, compton, rofi, polybar

```shell
sudo pacman -S bspwm sxhkd compton rofi
yay -S polybar
```

### Deactivate xfwm4
- Open 'Session and Startup' in XFCE settings manager and go to the 'Session' tab
- Select xfwm4, click 'Immediately' and change it to the 'Never' option
- Click the button: 'Save Session'

### Remove XFCE hotkeys
- Open 'Keyboard', and click the 'Application Shortcuts' tab
- Remove **ALL** keyboard shortcuts - if you don't do this, sxhkd won't work

### Autostart required applications in XFCE
I'm pretty sure there is a more efficient / elegant way to initiate these applications by editing xprofile or some such but since I'm a n00b and lazy, leveraging XFCE to autostart these applications is the easiest / most convenient option.
- Open 'Session and Startup' in XFCE settings manager
- Navigate to 'Application Autostart'

#### Add bspwm
- Name: bspwm
- Description: tiling-window-manager
- Command: bspwm

#### Add compton
- Name: compton
- Description: composite-manager
- Command: compton --config /home/xfcebspwm/.config/compton/compton.conf -b

### Set Workspaces to 10
- Open 'Workspaces'
- Increase number of workspaces to 10 (or the same value which you set in your bspwm config)

The --config flag directs compton to start using the config file located in the user's home directory. You need to spell out the full user directory path. XFCE will not accept '`~`' as a user's home directory. The -b flag sets compton to run in the background.

### Dotfile notes
#### bspwm | bspwmrc
There are a lot of resources out there to understand how this file works. For me, the most helpful guidance was understanding how this file controls mouse actions. The lines below allow the mouse to manipulate windows when the 'alt' key is drpressed: floating windows can be resized with the 'left click' button; floating windows can be moved with the 'right click' button; tiling windows can be resized with the 'left click' button.
- bspc config pointer_modifier mod1
- bspc config pointer_action1 resize_side
- bspc config pointer_action1 resize_corner
- bspc config pointer_action3 move

The other settings I have in this file set the number of workspaces, window padding, etc.
Lastly, I also like having a dock. I use [Docky](http://wiki.go-docky.com/index.php?title=Welcome_to_the_Docky_wiki). In order to set Docky to appear above all other windows, I set this rule:

`bspc rule -a Docky layer=above manage=on border=off focus=off locked=on`

I also have Docky locked with focus turned off, this helps me to prevent accidentally closing Docky. 

#### sxhkd | sxhkdrc
sxhkd is great and it's also really easy to use. You assign a keybinding on one line and then in the line below, you indent, and then list the action you want to execute after pressing that keybinding. After reloading the config file (assigned to alt + Escape in my config file) the command will work or it won't. If it doesn't, something is wrong with the action you want to execute or there is a keybinding conflict.

I like to leverage the function keys to execute the main features of bspwm and these are the settings I have defined in my sxhkdrc file:
- F1: rotate windows
- F2: circulate windows
- F3: flip windows horizontal
- F4: flip windows vertical
- F5: alternate between the tiled and monocle layout
- F6: balance windows
- F7: increase window gap
- F8: decrease window gap
- F9: set the window state - floating
- F10: set the window state - tiled
- F11: set the window state - pseudo_tiled
- F12: set the window state - fullscreen

I've also set up some hotkeys to move through workspaces:
- alt + left or right arrow

And some hotkeys to cycle through windows on a workspace:
- alt + up or down arrow

**Note:** I haven't even scratched the surface of bspwm by using the small number of commands listed above. bspwm is such an incredibly versatile WM I'm almost ashamed to only be using these features. Explore the man page, search google, and look at other dots to fully leverage bspwm. Do better than me. You're worth it.

#### compton | compton.conf
I honestly don't know what 90% of the shit in this config file even does, the most important part of this file (for me) was to set inactive window transparency. I wanted to set all 'unfocused' windows to be transparent. To do this, I had to adjust the opacity settings:

```
#################################
#
# Opacity
#
#################################
menu-opacity = 1;
inactive-window-opacity = 1;
inactive-opacity = 0.60;
active-opacity = 1;
frame-opacity = 1;
```

I also had to make sure these settings equaled 'false':  

`mark-wmwin-focused = false;`  
`mark-ovredir-focused = false;`  

#### rofi | config
I really only use rofi to switch windows. 
**Note to anyone reading - please let me know if you have a cool way to switch windows other than rofi!!!**
I've enabled rofi to run by hitting 'alt-tab' [to switch windows] and 'alt-return' [to run applications] both defined in sxhkdrc. Rofi was also always transparent, even when focused. To fix this, I had to add the following settings to **compton.conf**:

`# Specify a list of conditions of windows that should always be considered focused.`

`focus-exclude = ["name = 'rofi'"];`

The [rofi website](https://davedavenport.github.io/rofi/p11-Generator.html) has a really cool page to generate a custom theme. I like black and green. I selected these colors then copypasta into the rofi dot.

### Additional notes
bspwm uses roman numerals to name workspaces. I like this naming convention. To adjust this setting, open XFCE's settings manager, select workspaces, adjust the number of workspaces to match the number of workspaces you have defined in your bspwmrc file and rename them with roman numerals by simply clicking the 'workspace' name.

I was a little confused about CaSe sensitivity respective to bspwmrc and sxhkdrc when launching applications. [I asked about this on the bspwm subreddit](https://www.reddit.com/r/bspwm/comments/6sqw66/case_sensitivity_bspwmrc_and_sxhkdrc/) and received a very helpful response:

`From what i know, in the sxhkdrc file you use the same command that you use when starting the application. In my case for`
`example, that could be firefox or firefox-esr. But, in the bspwmrc I use the the output of xprop second "WM_CLASS(STRING)". In my`
`case, that's Firefox-esr.`

`Or, with libreoffice, I would put libreoffice or loffice in the sxhkd file but i have`

`bspc rule -a libreoffice-startcenter desktop=^3`

`in my bspwmrc since that's the output from xprop.`

`If you don't know about xprop, type xprop in the terminal and select the a window. The last string after "WM_CLASS(STRING)" is`
`what I put in bspwmrc. On firefox, xprop says this for eaxmple:`

`WM_CLASS(STRING) = "Navigator", "Firefox-esr"`

`(It's the same when excluding shadows or changing opacity for specifik aplications with compton btw. You have to use the output`
`from xprop. "urxvt" or "firefox-esr" won't work. It has to be Firefox-esr or URxvt.`

`Hope that helps :)`

`(Btw, if you want to start a GUI app from the terminal without terminal output and so on (and don't use rofi), put something like`

`alias gimp='((gimp > /dev/null 2>&1)&)'`

`in your .bash_aliases file.)`


**The bspwm setup in xfce is based on this Guide:** <https://github.com/bgdawes/bspwm-xfce-dotfiles/wiki>

## Post Config

### Enable network manager

```shell
sudo systemctl enable NetworkManager.service
sudo systemctl start NetworkManager.service
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
nemo-share \
quassel-client

gpg --recv-keys --keyserver sks-keyservers.net 0xDB1187B9DD5F693B

yay -S keepass-plugin-rpc \
keepass-plugin-haveibeenpwned \
keepass-plugin-http \
keepass2-plugin-tray-icon \
nextcloud-client \
franz-bin \
thunderbird-enigmail-bin \
nemo-compare \
spotify-stable
```


# TODO
* Audio (Mic)
	* Get external Microphones working!
	* Fix audio controls in xfce
