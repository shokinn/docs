# Seedbox

This Guide contains instructions to set up a seedbox based on [QuickBox](https://quickbox.io/) with [Perferct Privacy](https://www.perfect-privacy.com/) as all outgoing VPN on a VPS or a bare metal server.

This Guide is not sponsored by QuickBox nor by Perfect Privacy.

This Guide also requires that the server is already installed and access via SSH is working.

## QuickBox

[QuickBox](https://quickbox.io/) is an easy to setup seedbox 'script', which also allows you to add several services (e.g. [Sonarr](https://sonarr.tv/) and [Radarr](https://radarr.video/)) just by clicking 'install' at the webinterface.

### Installation

!!! warning "Run all commads as root"
	You have to be logged in as `root` to install QuckBox!

Now we will update all installed software and install git (for cloning the QuickBox repo) as well as lsb-release (a little tool to get the release version).

```shell
apt update; apt -y full-upgrade; apt -y install git lsb-release; \
git clone https://github.com/QuickBox/QB /etc/QuickBox; \
bash /etc/QuickBox/setup/quickbox-setup
```

If you get a locale warning like this one, just ignore it ;)  
```
perl: warning: Setting locale failed.
perl: warning: Please check that your locale settings:
	LANGUAGE = (unset),
	LC_ALL = (unset),
	LC_CTYPE = "UTF-8",
	LANG = "en_US.UTF-8"
    are supported and installed on your system.
perl: warning: Falling back to a fallback locale ("en_US.UTF-8").
perl: warning: Setting locale failed.
perl: warning: Please check that your locale settings:
	LANGUAGE = (unset),
	LC_ALL = (unset),
	LC_CTYPE = "UTF-8",
	LANG = "en_US.UTF-8"
    are supported and installed on your system.
perl: warning: Falling back to a fallback locale ("en_US.UTF-8").
```

When the installer opens install and configure QuickBox for your needs.  
This is my setup log:  
```

[QuickBox]  QuickBox Seedbox Installation

                 Heads Up!
     QuickBox works with the following
     Ubuntu 15.10 | 16.04


Checking distribution ...
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 16.04.4 LTS
Release:	16.04
Codename:	xenial

Congrats! You're running as root. Let's continue ...

Do you wish to write to a log file? (Default: Y) y
Output is being sent to /root/quickbox.2175.log
Please enter a hostname for this server (Hit ENTER to make no changes):
No hostname supplied, no changes made!!

Do you wish to use user quotas? (Default: Y) n
Quotas will not be installed

Press ENTER when you're ready to begin or Ctrl+Z to cancel
```

!!! warning "Password is shown as clear text"
	Please be aware that the password which you have to setup for the 'admin' account will be shown as clear text.

??? tip "Public Trackers needed?"
	If you don't plan to use public trackers like The Pirate Bay, you should set the option `Block Public Tracers` to yes to protect yourself.

??? tip "FFmpeg needed?"
	If you don't need FFmpeg (used for generating screen shots) don't install it, because it will be compiled from source which takes some time.


```
Is this a 10 gigabit server? (Default: N) n
Who can afford that stuff anyway?

1) rtorrent 0.9.6
2) rtorrent 0.9.4
3) rtorrent 0.9.3
What version of rtorrent do you want? (Default 1): 1
We will be using rtorrent-0.9.6/libtorrent-0.13.6

1) Deluge repo (fastest)
2) Deluge with libtorrent 1.0.11 (stable)
3) Deluge with libtorrent 1.1.3 (dev)
4) Do not install Deluge
What version of Deluge do you want? (Default 1): 1
We will be using Deluge with Libtorrent REPO

Would you like to install transmission?  [y]es or [n]o: n

1) QuickBox - smoked :: Dark theme
2) QuickBox - defaulted :: Light theme
Pick your QuickBox Dashboard Theme (Default 1): 1
We will be using QuickBox Theme : smoked

Add a Master Account user to sudoers
Username: dedibox
Password: (hit enter to generate a password) ***
setting password to ***

Would you like to install ffmpeg? (Used for screenshots) [y]es or [n]o: y

Please, write your public server IP (used for ftp) (Default: 0.0.0.0)

Block Public Trackers?: [y]es or [n]o: y
[ -  Blocking public trackers  - ]

Would you like to install bbr? (Used for Congestion Control) [y]es or [n]o: y
```

Wait until it's installed.  
On my machine it tooks 19 minuts.  
* 1 CPU (Intel Xeon - Skylake@2.1GHz)
* 2GB RAM
* SSD

```
QuickBox will now install, this may take between
10 and 45 minutes depending on your systems specs

Pulling QuickBox Ecosystem ... [ DONE ]
Updating system ... [ DONE ]
Installing all needed dependencies ... [ DONE ]
Setting up system executables ... [ DONE ]
Building required user directories ... [ DONE ]
Setting up Limited Shell environment ... [ DONE ]
Building ffmpeg from source for screenshots ... [ DONE ]
Installing xmlrpc-c-1.33.12 ... [ DONE ]
Installing libtorrent-0.13.6 ... [ DONE ]
Installing rtorrent-0.9.6 ... [ DONE ]
Installing rutorrent into /srv ... [ DONE ]
Installing rutorrent plugins ... [ DONE ]
Installing deluge ... [ DONE ]
Installing mktorrent from source ... [ DONE ]
Installing quickbox dashboard ... [ DONE ]
Building system file indexer (h5ai) ... [ DONE ]
Setting up seedbox.conf for apache ... [ DONE ]
Fix SSL Cert for apache ... [ DONE ]
Installing .rtorrent.rc for dedibox ... [ DONE ]
Adjusting fileupload & filemanager plugins ... [ DONE ]
Installing autodl-irssi ... [ DONE ]
Making dedibox directory structure ... [ DONE ]
Writing dedibox rutorrent config.php file ... [ DONE ]
Installing vsftpd ... [ DONE ]
Setting up vsftpd ... [ DONE ]
Setting irssi/rtorrent to start on boot ... [ DONE ]
Setting permissions on dedibox ... [ DONE ]
install BBR ... [ DONE ]
```

After finishing the installation __don't__ reboot the machine, if you want to use port `22` for SSH. If you're fine to use port `4747` for SSH just reboot.

```
     [quickbox] Seedbox & GUI Installation Completed
            INSTALLATION COMPLETED in 19/min


  Valid Commands:
  -------------------

 createSeedboxUser - creates a shelled seedbox user
 deleteSeedboxUser - deletes a created seedbox user and their directories
 changeUserpass - change users SSH/FTP/ruTorrent password
 setdisk - set your disk quota for any given user
 showspace - shows the amount of space used by all users on the server
 reload - restarts your seedbox services, i.e; rtorrent & irssi
 upgradeBTSync - upgrades BTSync when new version is available
 upgradeOmbi - upgrades Ombi when new version is available
 upgradePlex - upgrades Plex when new version is available
 box install letsencrypt - installs a valid SSL certificate to be used with
                           a valid domain name.
 box - type 'box -h' for a summary of how to use box!



################################################################################################
#   Seedbox can be found at https://dedibox:***@138.201.173.51
#   (Also works for FTP:5757/SSH:4747)
#   If you need to restart rtorrent/irssi, you can type 'reload'
#   Reloading: sshd, apache, memcached, php7.0, vsftpd and fail2ban
################################################################################################

  Do you wish to reboot (recommended!): (Default Y) N
```

#### Optional set the SSH port back to `22`

Because the install script forces a SSH port change we have to change it back to 22 manually.

Just run this `sed` command to replace the port 4747 with 22.

```shell
sed -i -e 's/^Port 4747$/Port 22/g' /etc/ssh/sshd_config
```

Afterwards reboot the machine.

#### Optional use 'Let's Encrypt' for a proper SSL certificate

If you use a domain for your server you should set up an let's encrypt ssl certificate.

Run the following command and follow the process. It's really straight forward.  

```shell
box install letsencrypt
```

## IPset

With IPset (and Dnsmasq) we are able to filter on a "DNS name" basis.

### Installation

```shell
apt install -y ipset
```

### Configuration

First we have to create the IPsets which want to use. In our case we need 2 of them, the 1st will store the IPs of the Perfect Privacy servers and the 2nd will store the IPs of the letsencrypt acme challenge servers.

```shell
ipset create perfect_privacy hash:ip family inet; \
ipset create letsencrypt hash:ip family inet
```

Afterwards we have to save our rules because they only configured in RAM. We will also write a little script which reloads the rules at startup.

```shell
ipset save > /etc/ipset.up.rules; \
cat > /etc/network/if-pre-up.d/loadrules <<-EOF; chmod 755 /etc/network/if-pre-up.d/loadrules
#!/bin/sh
/sbin/ipset restore -! < /etc/ipset.up.rules
EOF
```

## Dnsmasq

Dnsmasq is a tiny DNS server which we will use to resolve the dns addresses and store them to a IPset.

### Installation

```shell
apt install -y dnsmasq
```

### Configuration

We configure Dnsmasq to listen only on 127.0.0.1 and to store the IP addresses of specific domains in our IPsets.

Uncomment the last line to activate reading the conf file in `/etc/dnsmasq.d/`:  
```shell
sed -i -e 's/^#conf-dir=\/etc\/dnsmasq\.d\/,\*\.conf/conf-dir=\/etc\/dnsmasq\.d\/,\*\.conf/g' /etc/dnsmasq.conf
```

Configure the interfaces which the DNS server should listen to:  
```shell
cat > /etc/dnsmasq.d/01_interfaces.conf <<-EOF
# If you want dnsmasq to listen for DHCP and DNS requests only on
# specified interfaces (and the loopback) give the name of the
# interface (eg eth0) here.
# Repeat the line for more than one interface.
#interface=
# Or you can specify which interface _not_ to listen on
except-interface=tun0
# Or which to listen on by address (remember to include 127.0.0.1 if
# you use this.)
listen-address=127.0.0.1
# If you want dnsmasq to provide only DNS service on an interface,
# configure it as shown above, and then use the following line to
# disable DHCP and TFTP on it.
#no-dhcp-interface=
EOF
```

Configure the server to store the matching IPs to the IPset:  
```shell
cat > /etc/dnsmasq.d/02_ipsets.conf <<-EOF
# Add the IPs of all queries to yahoo.com, google.com, and their
# subdomains to the vpn and search ipsets:
#ipset=/yahoo.com/google.com/vpn,search
ipset=/perfect-privacy.com/perfect_privacy
ipset=/letsencrypt.org/letsencrypt
EOF
```

## OpenVPN

OpenVPN in an open source vpn software.

### Installation

First install OpenVPN, go to the OpenVPN config directory (`/etc/openvpn`) and download the configuration files from Perfect Privacy.  
The script below also creates a `password.txt` with credentials and modifies the files so that they refer to the credentials.

```shell
apt install -y openvpn && \
cd /etc/openvpn/ && \
clear; \
echo "Please enter your Perfect Privacy credentials"; \
read -p 'Username: ' username && \
read -sp 'Password: ' password && \
wget -v --post-data "username=$username&password=$password&uri=/member/download/?file=linux_udp.tar.gz" -O linux_udp.tar.gz "https://www.perfect-privacy.com/member/" && \
echo "$username" > password.txt && \
echo "$password" >> password.txt && \
username=''; \
password=''; \
tar xfvz linux_udp.tar.gz && \
mv ./linux_udp/* ./ && \
rmdir ./linux_udp && \
rm -f ./linux_udp.tar.gz && \
chown root:root ./* && \
chmod 400 password.txt && \
for file in ./*.ovpn; do sed -i -e 's/^auth-user-pass.*$/auth-user-pass password.txt/g' $file; done && \
sed -i -e '/echo -n \"\$R\" \| \/sbin\/resolvconf -a \"\${dev}\.openvpn\"/a \ \ \ \ \ \ \ \ \/etc\/openvpn\/patch_ports\.sh' /etc/openvpn/update-resolv-confsed
```

Afterwards link the config which you want to use to `.conf` (e.g. `Rotterdam.ovpn` to `Rotterdam.conf`) and modify the defaults file for OpenVPN to start the VPN connection automatically.  

```shell
clear; \
echo "Enter the file name of the for your preferred server location."; \
read -p "File name (e.g. Rotterdam.ovpn): " config_file && \
if [ ! -f ./$config_file ]; \
then echo "File not found!"; \
else \
config_name=$(echo $config_file | awk '{split($0,a,"."); print a[1]}'); \
ln -s ./$config_file ./$config_name.conf; \
sed -i -e "/^#AUTOSTART=\"home office\"$/a AUTOSTART=\"$config_name\"" /etc/default/openvpn; \
fi && \
systemctl daemon-reload
```

Now we are adding a script to modify the torrent client ports.  
`/etc/openvpn/patch_ports.sh`:
```shell
#!/bin/bash
vpn_tun_dev="tun0"
user="dedibox"

# Get VPN internal IP Address
vpn_int_ip=$(ip -f inet a s $vpn_tun_dev | grep "inet\b" | awk '{print $2}' | cut -d'/' -f1)

# Generate ports
blck3_ip=$(echo "$vpn_int_ip" | cut -f3 -d'.')
blck4_ip=$(echo "$vpn_int_ip" | cut -f4 -d'.')

blck3_prt=$(($(($blck3_ip%16))*256)) # Perfect Privacy port calculation for the 3rd octet -- (<3rd Block Number> mod 16) * 256

# PP port calcucaltion -- <Base port>+<generated 3rd block>+<4th block>
vpn_port1=$((10000+$blck3_prt+$blck4_ip))
vpn_port2=$((20000+$blck3_prt+$blck4_ip))
vpn_port3=$((30000+$blck3_prt+$blck4_ip))

# Change rtorrent incoming port
sed -i -E "s/^network\.port_range\.set = [0-9]{1,5}-[0-9]{1,5}$/network\.port_range\.set = $vpn_port1-$vpn_port1/g" /home/$user/.rtorrent.rc
systemctl restart rtorrent@dedibox.service

# Change Deluge incoming port
systemctl start deluged@dedibox.service
deluge_line=$(grep -n 'listen_ports' /home/$user/.config/deluge/core.conf | cut -d":" -f1)
sed -i "$(($deluge_line+1))s/.*/    $vpn_port2,/" /home/$user/.config/deluge/core.conf
sed -i "$(($deluge_line+2))s/.*/    $vpn_port2/" /home/$user/.config/deluge/core.conf
systemctl stop deluged@dedibox.service
```

## iptables

iptables is a userpace program to configure the firewall.

### Configuration

??? Tip "Explanation iptables rules"
	```shell
	# Allow loopback
	iptables -A OUTPUT -o lo -j ACCEPT
	iptables -A INPUT -i lo -j ACCEPT

	# Allow SSH incoming
	iptables -t filter -A INPUT -i eth0 -p tcp --dport 22 -m state --state NEW,ESTABLISHED,RELATED -j ACCEPT
	iptables -t filter -A OUTPUT -o eth0 -p tcp --sport 22 -m state --state ESTABLISHED,RELATED -j ACCEPT

	# Allow DNS outgoing
	iptables -t filter -A INPUT -i eth0 -p udp --sport 53 -m state --state ESTABLISHED,RELATED -j ACCEPT
	iptables -t filter -A OUTPUT -o eth0 -p udp --dport 53 -m state --state NEW,ESTABLISHED,RELATED -j ACCEPT

	# Allow OpenVPN outgoing only to perfect-privacy servers
	iptables -t filter -A OUTPUT -o eth0 -p udp -m set --match-set perfect_privacy dst -m multiport --dports 148,149,150,151,1148,1149,1150,1151 -m state --state NEW,ESTABLISHED,RELATED -j ACCEPT
	iptables -t filter -A OUTPUT -o eth0 -p tcp -m set --match-set perfect_privacy dst -m multiport --dports 300,301,142,152,1142,1152 -m state --state NEW,ESTABLISHED,RELATED -j ACCEPT
	iptables -t filter -A INPUT -i eth0 -p udp -m multiport --sports 148,149,150,151,1148,1149,1150,1151 -m state --state ESTABLISHED,RELATED -j ACCEPT
	iptables -t filter -A INPUT -i eth0 -p tcp -m multiport --sports 300,301,142,152,1142,1152 -m state --state ESTABLISHED,RELATED -j ACCEPT

	# Allow traffic through OpenVPN
	iptables -t filter -A OUTPUT -o tun0 -j ACCEPT
	iptables -t filter -A INPUT -i tun0 -j ACCEPT

	# Allow web interface SeedBox incoming
	iptables -t filter -A INPUT -i eth0 -p tcp -m multiport --dports 80,443,10637 -m state --state NEW,ESTABLISHED,RELATED -j ACCEPT
	iptables -t filter -A OUTPUT -o eth0 -p tcp -m multiport --sports 80,443,10637 -m state --state ESTABLISHED,RELATED -j ACCEPT

	# Policy DROP (everything else)
	iptables -t filter -P OUTPUT DROP
	iptables -t filter -P INPUT DROP
	```

Set up needed iptables rules:  
```shell
iptables -A OUTPUT -o lo -j ACCEPT; \
iptables -A INPUT -i lo -j ACCEPT; \
iptables -t filter -A INPUT -i eth0 -p tcp --dport 22 -m state --state NEW,ESTABLISHED,RELATED -j ACCEPT; \
iptables -t filter -A OUTPUT -o eth0 -p tcp --sport 22 -m state --state ESTABLISHED,RELATED -j ACCEPT; \
iptables -t filter -A INPUT -i eth0 -p udp --sport 53 -m state --state ESTABLISHED,RELATED -j ACCEPT; \
iptables -t filter -A OUTPUT -o eth0 -p udp --dport 53 -m state --state NEW,ESTABLISHED,RELATED -j ACCEPT; \
iptables -t filter -A OUTPUT -o eth0 -p udp -m set --match-set perfect_privacy dst -m multiport --dports 148,149,150,151,1148,1149,1150,1151 -m state --state NEW,ESTABLISHED,RELATED -j ACCEPT; \
iptables -t filter -A OUTPUT -o eth0 -p tcp -m set --match-set perfect_privacy dst -m multiport --dports 300,301,142,152,1142,1152 -m state --state NEW,ESTABLISHED,RELATED -j ACCEPT; \
iptables -t filter -A INPUT -i eth0 -p udp -m multiport --sports 148,149,150,151,1148,1149,1150,1151 -m state --state ESTABLISHED,RELATED -j ACCEPT; \
iptables -t filter -A INPUT -i eth0 -p tcp -m multiport --sports 300,301,142,152,1142,1152 -m state --state ESTABLISHED,RELATED -j ACCEPT; \
iptables -t filter -A OUTPUT -o tun0 -j ACCEPT; \
iptables -t filter -A INPUT -i tun0 -j ACCEPT; \
iptables -t filter -A INPUT -i eth0 -p tcp -m multiport --dports 80,443,10637 -m state --state NEW,ESTABLISHED,RELATED -j ACCEPT; \
iptables -t filter -A OUTPUT -o eth0 -p tcp -m multiport --sports 80,443,10637 -m state --state ESTABLISHED,RELATED -j ACCEPT; \
iptables -t filter -P OUTPUT DROP; \
iptables -t filter -P INPUT DROP
```

!!! Warning "If you use fail2ban"
	If you use fail to ban you have to delete the fail2ban rules from the config file!

Save iptables rules because they also only configured in RAM:  
```shell
iptables-save > /etc/iptables.up.rules; \
echo "/sbin/iptables-restore < /etc/iptables.up.rules" >> /etc/network/if-pre-up.d/loadrules
```

## ip rule/route

### Configuration

Add the following at the end of the block for the `eth0` device.  
Replace `1.2.3.4` with the public IP from your server:  
```shell
post-up ip route add 172.31.1.1/32 dev eth0 proto kernel scope link src 1.2.3.4 table 1
post-up ip route add default via 172.31.1.1 table 1
post-up ip rule add table 1 from 1.2.3.4
post-up ip rule add fwmark 1 table 1
```

## Mount storage box

Install `cifs-utils`:  
```shell
apt install -y cifs-utils
```

Create directory within `/mnt`:  
```shell
mkdir /mnt/.nas && \
chmod 444 /mnt/.nas
```

Add a group for storage access and nice the group id:  
```shell
addgroup storage
```

Add mount to fstab:  
```shell
clear; \
echo "Please enter your Storage Box credentials"; \
read -p 'Storage Box Username: ' sbusername && \
read -sp 'Storage Box Password: ' sbpassword && \
echo "" && \
read -p 'Mount User ID: ' muid && \
read -p 'Mount Group ID: ' mgid && \
echo "" >> /etc/fstab && \
echo "# Storage Box - dedibox" >> /etc/fstab && \
echo "//$sbusername.your-storagebox.de/backup /mnt/.nas cifs iocharset=utf8,credentials=/root/.smbcredentials,uid=$muid,gid=$mgid,rw,file_mode=0775,dir_mode=0775 0 0" >> /etc/fstab && \
echo "username=$sbusername" > /root/.smbcredentials && \
echo "password=$sbpassword" >> /root/.smbcredentials && \
chmod 600 /root/.smbcredentials && \
cat << EOF > /root/mount.sh && chmod 700 /root/mount.sh
#!/bin/bash

# Check if script runs as root
if [ $UID == 0 ]; then
	echo "You have to run this script as root."
	exit 1
fi

# Mount all unmounted mounts
mount -a

# Mount Crypto
read -sp 'rclone config password: ' RCLONE_CONFIG_PASS
export RCLONE_CONFIG_PASS
screen -dmS rclone rclone mount --ask-password=false --uid $muid --gid $mgid --umask 002 --allow-other storage_box_crypt: /mnt/nas
unset RCLONE_CONFIG_PASS
EOF
```

Mount NAS storage:  
```shell
mount -a
```

## rclone

rclone is tiny tool to mount cloud storages. It also provides a strong encryption.

### Installation

```shell
curl https://rclone.org/install.sh | sudo bash
```

### Configuration

Open rclone configuration:  
```shell
rclone config
```

1. Select 'n' (new remote)
2. Name: `storage_box_crypt`
3. Storage Type: '7' (Encrypt/Decrypt a remote) "crypt"
4. remote: `/mnt/.nas`
5. Filename encryption: '2' (Encrypt the filenames see the docs for the details.) "standard"
6. Encrypt directories: '1' (Encrypt directory names.) "true"
7. Password: 'y' (Yes type in my own password)
8. Salt: 'y' (Yes type in my own password)
9. Confirm configuration: 'y' (Yes this is OK)
10. Set configuration password: 's'
11. Add Password: 'a'
12. Quit to main menu: 'q'
10. Quit configuration: 'q'

Create mount point:  
```shell
mkdir /mnt/nas && \
chmod 444 /mnt/nas
```

Mount rclone share:  
```shell
./mount.sh
```