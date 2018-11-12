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
startx
```

The `.xinitrc` should already be fine because I installed my dot-files.

### Enable/Start NetworkManager

```shell
systemctl enable NetworkManager.service
systemctl start NetworkManager.service
```

### Install xscreensaver

1. Install xscreensaver-aerial theme and download the 1080p videos to disk.  
```shell
yay -S xscreensaver-aerial && \
sudo mkdir /opt/ATV4
```

2. Add download script and download the Videos for offline use:  
```shell
sudo touch /opt/ATV4/download.sh && \
cat << EOF | sudo tee /opt/ATV4/download.sh && \
sudo chmod +x /opt/ATV4/download.sh && \
cd /opt/ATV4 && \
sudo ./download.sh
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

path_vid='/opt/ATV4'
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
```


# TODO
* Audio (Mic)
	* Get external Microphones working!
	* Fix audio controls in xfce
* Screensaver
	* Add screenlock command shortcut