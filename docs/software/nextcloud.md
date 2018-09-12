# NextCloud

OS: Ubuntu 18.04 LTS

Preinstalled with lvm and LUKS encryption.

## Initial Server setup

### Update System

```shell
apt update && apt full-upgrade -y
```

### Fix locale

```shell
cat << EOF | sudo tee -a /etc/environment

# Fix locale
LC_ALL=en_US.UTF-8
LANGUAGE=en_US:en
EOF
```

### Edit .bashrc

`/root/.bashrc` / `/etc/skel/.bashrc`:  
```
# ~/.bashrc: executed by bash(1) for non-login shells.
# see /usr/share/doc/bash/examples/startup-files (in the package bash-doc)
# for examples

# If not running interactively, don't do anything
[ -z "$PS1" ] && return

# don't put duplicate lines in the history. See bash(1) for more options
# ... or force ignoredups and ignorespace
HISTCONTROL=ignoredups:ignorespace

# append to the history file, don't overwrite it
shopt -s histappend

# for setting history length see HISTSIZE and HISTFILESIZE in bash(1)
HISTSIZE=999999
HISTFILESIZE=999999

# check the window size after each command and, if necessary,
# update the values of LINES and COLUMNS.
shopt -s checkwinsize

# make less more friendly for non-text input files, see lesspipe(1)
[ -x /usr/bin/lesspipe ] && eval "$(SHELL=/bin/sh lesspipe)"

# set variable identifying the chroot you work in (used in the prompt below)
if [ -z "$debian_chroot" ] && [ -r /etc/debian_chroot ]; then
    debian_chroot=$(cat /etc/debian_chroot)
fi

# set a fancy prompt (non-color, unless we know we "want" color)
case "$TERM" in
    xterm-color) color_prompt=yes;;
esac

# uncomment for a colored prompt, if the terminal has the capability; turned
# off by default to not distract the user: the focus in a terminal window
# should be on the output of commands, not on the prompt
force_color_prompt=yes

if [ -n "$force_color_prompt" ]; then
    if [ -x /usr/bin/tput ] && tput setaf 1 >&/dev/null; then
	# We have color support; assume it's compliant with Ecma-48
	# (ISO/IEC-6429). (Lack of such support is extremely rare, and such
	# a case would tend to support setf rather than setaf.)
	color_prompt=yes
    else
	color_prompt=
    fi
fi

if [ "$color_prompt" = yes ]; then
    # PS1='${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '
    if [ $UID == 0 ]; then
        PS1='${debian_chroot:+($debian_chroot)}\[\033[01;31m\]\u\[\033[01;33m\]@\[\033[01;36m\]\h \[\033[01;33m\]\w \[\033[01;35m\]\$ \[\033[00m\]'
    else
        PS1='${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u\[\033[01;33m\]@\[\033[01;36m\]\h \[\033[01;33m\]\w \[\033[01;35m\]\$ \[\033[00m\]'
    fi
else
    PS1='${debian_chroot:+($debian_chroot)}\u@\h:\w\$ '
fi
unset color_prompt force_color_prompt

# If this is an xterm set the title to user@host:dir
case "$TERM" in
xterm*|rxvt*)
    PS1="\[\e]0;${debian_chroot:+($debian_chroot)}\u@\h: \w\a\]$PS1"
    ;;
*)
    ;;
esac

# enable color support of ls and also add handy aliases
if [ -x /usr/bin/dircolors ]; then
    test -r ~/.dircolors && eval "$(dircolors -b ~/.dircolors)" || eval "$(dircolors -b)"
    alias ls='ls --color=auto'
    #alias dir='dir --color=auto'
    #alias vdir='vdir --color=auto'

    alias grep='grep --color=auto'
    alias fgrep='fgrep --color=auto'
    alias egrep='egrep --color=auto'
fi

# some more ls aliases
alias ll='ls -alF'
alias la='ls -A'
alias l='ls -CF'

# Alias definitions.
# You may want to put all your additions into a separate file like
# ~/.bash_aliases, instead of adding them here directly.
# See /usr/share/doc/bash-doc/examples in the bash-doc package.

if [ -f ~/.bash_aliases ]; then
    . ~/.bash_aliases
fi

# enable programmable completion features (you don't need to enable
# this, if it's already enabled in /etc/bash.bashrc and /etc/profile
# sources /etc/bash.bashrc).
#if [ -f /etc/bash_completion ] && ! shopt -oq posix; then
#    . /etc/bash_completion
#fi
```

### Make vim colored with dark background settinfs

```shell
cat << EOF | sudo tee -a /etc/vim/vimrc

" Set background to dark for better readability in SSH connections
set background=dark
EOF
```

### Change the SSH deamon to allow only SSH-keys

```shell
cat << EOF | sudo tee /etc/ssh/sshd_config && sudo systemctl restart sshd.service
# $OpenBSD: sshd_config,v 1.101 2017/03/14 07:19:07 djm Exp $

# This is the sshd server system-wide configuration file.  See
# sshd_config(5) for more information.

# This sshd was compiled with PATH=/usr/bin:/bin:/usr/sbin:/sbin

# The strategy used for options in the default sshd_config shipped with
# OpenSSH is to specify options with their default value where
# possible, but leave them commented.  Uncommented options override the
# default value.

Port 22
AddressFamily any
ListenAddress 0.0.0.0
ListenAddress ::

HostKey /etc/ssh/ssh_host_rsa_key
HostKey /etc/ssh/ssh_host_ecdsa_key
HostKey /etc/ssh/ssh_host_ed25519_key

# Ciphers and keying
#RekeyLimit default none

# Logging
#SyslogFacility AUTH
#LogLevel INFO

# Authentication:

#LoginGraceTime 2m
PermitRootLogin without-password
#StrictModes yes
#MaxAuthTries 6
#MaxSessions 10

PubkeyAuthentication yes

# Expect .ssh/authorized_keys2 to be disregarded by default in future.
AuthorizedKeysFile .ssh/authorized_keys .ssh/authorized_keys2

#AuthorizedPrincipalsFile none

#AuthorizedKeysCommand none
#AuthorizedKeysCommandUser nobody

# For this to work you will also need host keys in /etc/ssh/ssh_known_hosts
#HostbasedAuthentication no
# Change to yes if you don't trust ~/.ssh/known_hosts for
# HostbasedAuthentication
#IgnoreUserKnownHosts no
# Don't read the user's ~/.rhosts and ~/.shosts files
#IgnoreRhosts yes

# To disable tunneled clear text passwords, change to no here!
PasswordAuthentication no
PermitEmptyPasswords no

# Change to yes to enable challenge-response passwords (beware issues with
# some PAM modules and threads)
ChallengeResponseAuthentication no

# Kerberos options
#KerberosAuthentication no
#KerberosOrLocalPasswd yes
#KerberosTicketCleanup yes
#KerberosGetAFSToken no

# GSSAPI options
#GSSAPIAuthentication no
#GSSAPICleanupCredentials yes
#GSSAPIStrictAcceptorCheck yes
#GSSAPIKeyExchange no

# Set this to 'yes' to enable PAM authentication, account processing,
# and session processing. If this is enabled, PAM authentication will
# be allowed through the ChallengeResponseAuthentication and
# PasswordAuthentication.  Depending on your PAM configuration,
# PAM authentication via ChallengeResponseAuthentication may bypass
# the setting of "PermitRootLogin without-password".
# If you just want the PAM account and session checks to run without
# PAM authentication, then enable this but set PasswordAuthentication
# and ChallengeResponseAuthentication to 'no'.
UsePAM yes

#AllowAgentForwarding yes
#AllowTcpForwarding yes
#GatewayPorts no
X11Forwarding yes
#X11DisplayOffset 10
#X11UseLocalhost yes
#PermitTTY yes
PrintMotd no
#PrintLastLog yes
#TCPKeepAlive yes
#UseLogin no
#PermitUserEnvironment no
#Compression delayed
#ClientAliveInterval 0
#ClientAliveCountMax 3
#UseDNS no
#PidFile /var/run/sshd.pid
#MaxStartups 10:30:100
#PermitTunnel no
#ChrootDirectory none
#VersionAddendum none

# no default banner path
#Banner none

# Allow client to pass locale environment variables
AcceptEnv LANG LC_*

# override default of no subsystems
Subsystem sftp /usr/lib/openssh/sftp-server

# Example of overriding settings on a per-user basis
#Match User anoncvs
# X11Forwarding no
# AllowTcpForwarding no
# PermitTTY no
# ForceCommand cvs server
EOF
```

### Install base packages

```shell
sudo apt install -y \
p7zip-full \
p7zip-rar \
unzip \
unrar \
screen \
tmux \
htop
```

### Install and configure busybox/dropbear

Install busybox and dropbear:  
```shell
sudo apt update && sudo apt full-upgrade -y && sudo apt install -y busybox dropbear
```

Edit your `/etc/initramfs-tools/initramfs.conf` and set `BUSYBOX=y`:  
```shell
sudo sed -i -e 's/^BUSYBOX=.*$/BUSYBOX=y/' /etc/initramfs-tools/initramfs.conf && \
cat << EOF | sudo tee -a /etc/initramfs-tools/initramfs.conf
#
# DROPBEAR
#

DROPBEAR=y

EOF
```

Set dropbear to start:  
```shell
sudo sed -i 's/^NO_START.*$/NO_START=0/' /etc/default/dropbear
```

Change dropear port to `2222`:  
```shell
sudo sed -i 's/^#DROPBEAR_OPTIONS.*$/DROPBEAR_OPTIONS="-p 2222"/' /etc/dropbear-initramfs/config
```

??? Info "crypt_unlock.sh"
	```
	#!/bin/sh

	PREREQ="dropbear"

	prereqs() {
	  echo "$PREREQ"
	}

	case "$1" in
	  prereqs)
	    prereqs
	    exit 0
	  ;;
	esac

	. "${CONFDIR}/initramfs.conf"
	. /usr/share/initramfs-tools/hook-functions

	if [ "${DROPBEAR}" != "n" ] && [ -r "/etc/crypttab" ] ; then
	cat > "${DESTDIR}/bin/unlock" << EOF
	#!/bin/sh
	if PATH=/lib/unlock:/bin:/sbin /scripts/local-top/cryptroot; then
	kill \`ps | grep cryptroot | grep -v "grep" | awk '{print \$1}'\`
	# following line kill the remote shell right after the passphrase has
	# been entered.
	kill -9 \`ps | grep "\-sh" | grep -v "grep" | awk '{print \$1}'\`
	exit 0
	fi
	exit 1
	EOF
	  
	  chmod 755 "${DESTDIR}/bin/unlock"
	  
	  mkdir -p "${DESTDIR}/lib/unlock"
	cat > "${DESTDIR}/lib/unlock/plymouth" << EOF
	#!/bin/sh
	[ "\$1" == "--ping" ] && exit 1
	/bin/plymouth "\$@"
	EOF
	  
	  chmod 755 "${DESTDIR}/lib/unlock/plymouth"
	  
	  echo To unlock root-partition run "unlock" >> ${DESTDIR}/etc/motd
	  
	fi
	```

Get the unlock script:  
```shell
sudo wget -O /etc/initramfs-tools/hooks/crypt_unlock.sh https://gist.githubusercontent.com/gusennan/712d6e81f5cf9489bd9f/raw/fda73649d904ee0437fe3842227ad8ac8ca487d1/crypt_unlock.sh && \
sudo chmod +x /etc/initramfs-tools/hooks/crypt_unlock.sh && \
sudo update-initramfs -u && \
sudo systemctl disable dropbear
```

Add your ssh keys to the busybox authorized keys:  
```shell
cat << EOF | sudo tee /etc/dropbear-initramfs/authorized_keys
ssh-rsa AAAAB3NzaC1yc2EAAAABJQAAAgEAwVT82+FIbeVNjJ3Waa3Z0ysmG+DNhX6qUN5C2o3lT158WCcWgDU5qs/DgJDQFVK+m33cvozdaoV2sreSHmKLlM77k5zU+OTDswrEaJ53CRSu7tgT6ZRI2ggxzhvrw6xq1bmTlSuaojSmaJNDZllgFtrlG1XXGSgkgyDtyJuUk16MwKU4zLAWOluwrZBMIvlafNnJS4K385rI2NNIMUvWHjfxYTxAFAtqBZFacz/kQ9jm4UYT+v7YOXu48Cgul6S51eNbK0rUe7DU7+1xzusdhyqHq7FnFugvW6OYy3ft3y/ri/7qvKAhtwQSo6A8cgLVlPnMKdDWq/QAC6dLGcUEHJPJDYumMpM6ijrfd1DCpB+ELr/dFHgR5l6++OJM2/kl4f3ue1gp5b//6osnfMhWrQXbmk8WLF31IXmaZnwlcKKEvgNQL5O9U39HRuZBvXh5aib/vFtQ5ge6l1wG+eFLrMHjeLPDYCYuNliPisjmLNUb+0KyfDR2KnrqwZVXMOiKdh+S4OaXW7T+woykZ3u7FODfZwcRdnEgZFxYLSHRh8U/7fFzbAD4jJH29D9nHz46hx0OdEtiDJoeujf+GLXw4c7P2G+IYlPVg6sPJ5W+oky5gboQh13IOnFXFXd5kYuWnzOU/4ITy4vHw2WowbCYMFY5GNrRzRMpmcIj55OXrwM= 71:72:8d:23:dd:13:ea:90:17:35:4e:0a:2b:c7:d5:91 Philip Henning (mail@philip-henning.com)
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQCZntAPXzfGddkOxodxRCv2zBgbqHo9aAqtvoiZD0NuuBkYj3tSMKELMi0corvtXHy3iTa6EAKVc/AQqsNrggOeIYHwCSXZZ67sa0YxItDD3+YBm/v65EK/sspvq+xPD8IyZyYWDy6CmK1HiBlv6TFnGlN5PuFKfPJFBYEkx0Dzjl3i1r+xSpPKOcVgtPDs3MIYvBB1Y8Sig+JZehBELiioBQaWjKDPBJYsLQzqjGPLcBb/h1H729P6B4oW7A1LYsStJd5UDfknOgdx4pPoSCFBE5aEDO/0efqPcN7jQWlrrsc+OEWRW7k7EA45+9x6vjjUCO8IgKQFAIiOJadDPEaNAJnza4IpCspePeITZ3iyXo2/w9/BrzyNnW+arJ1QWjZ79W9mPXg4CTvUsnMQ55BRw2meLBxQqgkr9H8Cf/IxCEB2bPYPecCcVZj4djjgVbzlY/2/vFPDmB3Idx+AagACJZoDoWqaqlloC+fPyovG9iCjA+7iMEg4OR7kD/GCKGRmNzPmKNgkBwp62eyP4L9197bhIywMkaVEofqzHNOFly1f7KTEK/Dgo6GHCCsFLnvnPyZDvJ8o7bG7svan4e1KmY5JLSKZrtkPMJzvNmEUGsGlwFMWtNEhiGG01YiTIBcG3Z7N8lpi8Bcoti2Lq5chzApAwhHkykasHq218QPnpQ== a0:ce:a9:b4:c1:e8:c7:0e:ed:02:15:a0:58:56:34:64 matze
ssh-rsa AAAAB3NzaC1yc2EAAAABJQAAAgEAyhhxTowK157f2bZiV5W9Vy0wemmo07EuH2un6boKjzQ8oNDB3AKUeLgef76spBZZQ668kkwL+qDcCssPuRUdrxuclTw7bgf+RtYV1dkxRxCXfMqT4Gxsz2VOAClImWY9cmWSkigzZfS0/R2JTTO+H2ohOqRGqLH0dNjswRYW33mMMCy6ksZgv4Bin9B5j6u9fsj4AcwtEMiNVKXMX9z6FELo9/ayWwvnEmzL9Bs8qR9+7jLbhaJJeJNZEh/teZu3w5/ahHtj0rWHZ0F/XNmAKFCS9M/buExXw3RF0PAByAJN+J4uMzcqx64guKoe5J1Sz5OZy+Tn/IICLv0oKggnfT1BPXwUAHm+8jGW/q1NFKXc2IanhS6wkBlJUWawEupJHj2TIdTI55mscsUH9g9a0WI7dSiZkkER8HoJYg5vZZ9y7OoklK8zPuGWPeOcIStCROXkTT/o0W/S7SVPKXeXrj3FNzV2Ibw8V3YSMJpA9f52kMkVp1Jzr8/L0i7vSuLIU628rVj1QBvmE1funKMEa9uWdqSgOZyVYVBzXzlahA8D/lBK8cX2Qplq9Wt/pUGPS+OczPwDW2IcRIdkVt/crCgxiYOaZxh853zMR1BHL9bZcwnL6voSgCi0uRzfBA4wEGCiZN7qoIVAs1axraet3G95rASzLI4qzlc5vtiUq98= 54:7e:57:18:15:f4:59:0f:d1:81:69:d9:dc:53:6e:0a robin@beismann.biz
EOF
```

Regenerate the initramfs:  
```shell
sudo update-initramfs -u && \
sudo update-grub && \
sudo grub-install /dev/vda
```

### Mount external storage

#### pre requirements

Install cifs-utils:  
```shel
sudo apt install -y cifs-utils
```

#### Mount mounts

```shell
while true; do \
unset pw; \
unset pw_confirm; \
read -p "cifs user: " cifs_user && \
read -s -p "cifs password: " pw; echo "" && \
read -s -p "Confirm admin user password: " pw_confirm; echo "" && \
if [[ "$pw" == "$pw_confirm" ]]; then \
break; \
else \
echo "Your passwords don't match. Try again!"; \
echo ""; \
fi; \
done; \
if [[ ! -d /mnt/.cifs ]]; then sudo mkdir /mnt/.cifs; fi; \
sudo chmod 400 /mnt/.cifs; \
cat << EOF | sudo tee -a /etc/fstab && \
sudo mount -a


# Cifs shared storage
//io.servercow.de/home /mnt/.cifs cifs username=$cifs_user,password=$pw,uid=www-data,gid=www-data,file_mode=0660,dir_mode=0770 0 0
EOF
```

#### Install rclone

Install rclone:  
```shell
curl https://rclone.org/install.sh | sudo bash
```

create mountpoint:  
```shell
sudo mkdir /mnt/cifs && \
sudo chmod 400 /mnt/cifs
```

#### configure

Open rclone configuration:  
```shell
if [[ ! -d /mnt/.cifs ]]; then sudo mkdir -p /var/www/.conf/rclone/; fi && \
sudo chmod 700 /var/www/.conf/rclone/ && \
sudo chown www-data:www-data /var/www/.conf/rclone/ && \
sudo -u www-data rclone config --config /var/www/.conf/rclone/rclone.conf && \
sudo chmod 600 /var/www/.conf/rclone/rclone.conf
```

Configuration log:  
```
2018/09/09 21:09:35 NOTICE: Config file "/var/www/.conf/rclone/rclone.conf" not found - using defaults
No remotes found - make a new one
n) New remote
s) Set configuration password
q) Quit config
n/s/q> n
name> cifs_crypt
Type of storage to configure.
Enter a string value. Press Enter for the default ("").
Choose a number from below, or type in your own value
 1 / Alias for a existing remote
   \ "alias"
 2 / Amazon Drive
   \ "amazon cloud drive"
 3 / Amazon S3 Compliant Storage Providers (AWS, Ceph, Dreamhost, IBM COS, Minio)
   \ "s3"
 4 / Backblaze B2
   \ "b2"
 5 / Box
   \ "box"
 6 / Cache a remote
   \ "cache"
 7 / Dropbox
   \ "dropbox"
 8 / Encrypt/Decrypt a remote
   \ "crypt"
 9 / FTP Connection
   \ "ftp"
10 / Google Cloud Storage (this is not Google Drive)
   \ "google cloud storage"
11 / Google Drive
   \ "drive"
12 / Hubic
   \ "hubic"
13 / JottaCloud
   \ "jottacloud"
14 / Local Disk
   \ "local"
15 / Mega
   \ "mega"
16 / Microsoft Azure Blob Storage
   \ "azureblob"
17 / Microsoft OneDrive
   \ "onedrive"
18 / OpenDrive
   \ "opendrive"
19 / Openstack Swift (Rackspace Cloud Files, Memset Memstore, OVH)
   \ "swift"
20 / Pcloud
   \ "pcloud"
21 / QingCloud Object Storage
   \ "qingstor"
22 / SSH/SFTP Connection
   \ "sftp"
23 / Webdav
   \ "webdav"
24 / Yandex Disk
   \ "yandex"
25 / http Connection
   \ "http"
Storage> 8
Remote to encrypt/decrypt.
Normally should contain a ':' and a path, eg "myremote:path/to/dir",
"myremote:bucket" or maybe "myremote:" (not recommended).
Enter a string value. Press Enter for the default ("").
remote> /mnt/.cifs
How to encrypt the filenames.
Enter a string value. Press Enter for the default ("standard").
Choose a number from below, or type in your own value
 1 / Don't encrypt the file names.  Adds a ".bin" extension only.
   \ "off"
 2 / Encrypt the filenames see the docs for the details.
   \ "standard"
 3 / Very simple filename obfuscation.
   \ "obfuscate"
filename_encryption> 2
Option to either encrypt directory names or leave them intact.
Enter a boolean value (true or false). Press Enter for the default ("true").
Choose a number from below, or type in your own value
 1 / Encrypt directory names.
   \ "true"
 2 / Don't encrypt directory names, leave them intact.
   \ "false"
directory_name_encryption> 1
Password or pass phrase for encryption.
y) Yes type in my own password
g) Generate random password
n) No leave this optional password blank
y/g/n> g
Password strength in bits.
64 is just about memorable
128 is secure
1024 is the maximum
Bits> 1024
Your password is: ***
Use this password? Please note that an obscured version of this
password (and not the password itself) will be stored under your
configuration file, so keep this generated password in a safe place.
y) Yes
n) No
y/n> y
Password or pass phrase for salt. Optional but recommended.
Should be different to the previous password.
y) Yes type in my own password
g) Generate random password
n) No leave this optional password blank
y/g/n> g
Password strength in bits.
64 is just about memorable
128 is secure
1024 is the maximum
Bits> 1024
Your password is: ***
Use this password? Please note that an obscured version of this
password (and not the password itself) will be stored under your
configuration file, so keep this generated password in a safe place.
y) Yes
n) No
y/n> y
Edit advanced config? (y/n)
y) Yes
n) No
y/n> y
For all files listed show how the names encrypt.
Enter a boolean value (true or false). Press Enter for the default ("false").
show_mapping>
Remote config
--------------------
[cifs_crypt]
type = crypt
remote = /mnt/.cifs
filename_encryption = standard
directory_name_encryption = true
password = *** ENCRYPTED ***
password2 = *** ENCRYPTED ***
--------------------
y) Yes this is OK
e) Edit this remote
d) Delete this remote
y/e/d> y
Current remotes:

Name                 Type
====                 ====
cifs_crypt           crypt

e) Edit existing remote
n) New remote
d) Delete remote
r) Rename remote
c) Copy remote
s) Set configuration password
q) Quit config
e/n/d/r/c/s/q> q
```

Test if Rclone can mount:  
```shell
screen -S 'rclone' sudo rclone mount --config /var/www/.conf/rclone/rclone.conf --uid $(id -u www-data) --gid $(id -g www-data) --umask 002 --allow-other cifs_crypt: /mnt/cifs
```

Unmount it:  
```shell
sudo fusermount -uz /mnt/cifs
```

Create systemd startup script `/etc/systemd/system/rclone.service`:  
```
cat << EOF | sudo tee /etc/systemd/system/rclone.service
[Unit]
Description=rclone encryption mount
AssertPathIsDirectory=/mnt/cifs

[Service]
Type=simple
ExecStart=/usr/bin/rclone mount --config /var/www/.conf/rclone/rclone.conf --uid $(id -u www-data) --gid $(id -g www-data) --umask 007 --allow-other cifs_crypt: /mnt/cifs
ExecStop=/bin/fusermount -uz /mnt/cifs
Restart=on-abort
RestartSec=5
StartLimitInterval=60s
StartLimitBurst=3

[Install]
WantedBy=default.target
EOF
```

Refresh your daemons:  
```shell
sudo systemctl daemon-reload
```

Activate the auto startup option and start the service:  
```shell
sudo systemctl enable rclone.service && \
sudo systemctl start rclone.service
```

## Install Nextcloud

### Nginx/certnpt

Install nginx:  
```shell
sudo apt update && \
sudo apt install -y \
nginx \
python3 \
python3-pip && \
sudo -H pip3 install \
certbot \
certbot-nginx
```

### PHP 7.2

Install php7.2:  
```shell
sudo apt install software-properties-common -y && \
sudo add-apt-repository ppa:ondrej/php -y && \
sudo apt install php7.2-fpm php7.2-curl php7.2-cli php7.2-mysql php7.2-gd php7.2-iconv php7.2-xsl php7.2-json php7.2-intl php-pear php-imagick php7.2-dev php7.2-common php7.2-mbstring php7.2-zip php7.2-soap php-apcu -y
```

Restart php-fpm and nginx:  
```shell
sudo systemctl restart php7.2-fpm.service nginx.service
```

Modify php.ini's:  
```shell
sudo sed -i -e 's/^;date.timezone =*$/date.timezone = Europe\/Berlin/' /etc/php/7.2/fpm/php.ini && \
sudo sed -i -e 's/^.*cgi\.fix_pathinfo=.*$/cgi.fix_pathinfo=0/' /etc/php/7.2/fpm/php.ini && \
sudo sed -i -e 's/^.*opcache\.enable=.*$/opcache.enable=1/' /etc/php/7.2/fpm/php.ini && \
sudo sed -i -e 's/^.*opcache\.enable_cli=.*$/opcache.enable_cli=1/' /etc/php/7.2/fpm/php.ini && \
sudo sed -i -e 's/^.*opcache\.interned_strings_buffer=.*$/opcache.interned_strings_buffer=8/' /etc/php/7.2/fpm/php.ini && \
sudo sed -i -e 's/^.*opcache\.max_accelerated_files=.*$/opcache.max_accelerated_files=10000/' /etc/php/7.2/fpm/php.ini && \
sudo sed -i -e 's/^.*opcache\.memory_consumption=.*$/opcache.memory_consumption=128/' /etc/php/7.2/fpm/php.ini && \
sudo sed -i -e 's/^.*opcache\.save_comments=.*$/opcache.save_comments=1/' /etc/php/7.2/fpm/php.ini && \
sudo sed -i -e 's/^.*opcache\.revalidate_freq=.*$/opcache.revalidate_freq=1/' /etc/php/7.2/fpm/php.ini && \
sudo sed -i -e 's/^;date.timezone =*$/date.timezone = Europe\/Berlin/' /etc/php/7.2/cli/php.ini && \
sudo sed -i -e 's/^.*cgi\.fix_pathinfo=.*$/cgi.fix_pathinfo=0/' /etc/php/7.2/cli/php.ini 
```

Uncomment those lines below:  
`vim /etc/php/7.1/fpm/pool.d/www.conf`:  
```
env[HOSTNAME] = $HOSTNAME
env[PATH] = /usr/local/bin:/usr/bin:/bin
env[TMP] = /tmp
env[TMPDIR] = /tmp
env[TEMP] = /tmp
```

Restart php-fpm:  
```shell
sudo systemctl restart php7.2-fpm && \
sudo systemctl enable php7.2-fpm
```

### MariaDB

Install MariaDB:  
```shell
sudo apt install mariadb-server mariadb-client -y
```

Secure MariaDB installtion:  
```shell
sudo mysql_secure_installation
```

Log:  
```
NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user.  If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none):
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.

Set root password? [Y/n] y
New password:
Re-enter new password:
Password updated successfully!
Reloading privilege tables..
 ... Success!


By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] y
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] y
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] y
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
```

Login to MariaDB as `root`:  
```shell
sudo mysql -u root -p
```

!!! Warning
	CHANGE THE DEAFAULT PASSWORD!

Create a Nextcloud DB:  
```mysql
create database nextcloud;
create user nextcloud@localhost identified by 'PASSWORD';
grant all privileges on nextcloud.* to nextcloud@localhost identified by 'PASSWORD';
flush privileges;
```

Exit MariaDB client with `CTRL+D`.

### Nginx config

Remove alll current servers:  
```shell
sudo rm /etc/nginx/sites-enabled/*
```

All general HTTP to HTTPS redirector:  
```shell
read -r -d '' read_tmp<<"EOF"
server {
\tlisten 80 default_server;
\tlisten [::]:80 default_server;
\tserver_name _;
\treturn 301 https://$host$request_uri;
}
EOF
echo -e "$read_tmp" | sudo tee /etc/nginx/sites-available/99-https-rewrite.conf && \
sudo ln -s ../sites-available/99-https-rewrite.conf /etc/nginx/sites-enabled/99-https-rewrite.conf
```

Add `ssl_params` file:  
```shell
cat << EOF | sudo tee /etc/nginx/ssl_params
# Session settings
ssl_session_timeout 1d;
ssl_session_cache shared:SSL:50m;
ssl_session_tickets off;

# modern configuration. tweak to your needs.
ssl_protocols TLSv1.2;
ssl_ciphers 'ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256';
ssl_prefer_server_ciphers on;

# HSTS (ngx_http_headers_module is required) (15768000 seconds = 6 months)
add_header Strict-Transport-Security max-age=15768000;

# OCSP Stapling ---
# fetch OCSP records from URL in ssl_certificate and cache them
ssl_stapling on;
ssl_stapling_verify on;
EOF
```

Add nextcloud server:  
```shell
cat << EOF | sed 's/\\t/\t/g' | sudo tee /etc/nginx/sites-available/10-cloud.pphg.tech.conf && \
sudo ln -s ../sites-available/10-cloud.pphg.tech.conf /etc/nginx/sites-enabled/10-cloud.pphg.tech.conf
upstream php-handler {
\t#server 127.0.0.1:9000;
\tserver unix:/run/php/php7.2-fpm.sock;
}

server {
\tlisten\t\t443 ssl http2;
\tlisten\t\t[::]:443 ssl http2;
\tserver_name\tcloud.pphg.tech;

\taccess_log\t/var/log/nginx/cloud.pphg.tech_access.log combined gzip=9;
\terror_log\t/var/log/nginx/cloud.pphg.tech_error.log warn;

\t# Add headers to serve security related headers
\t# Before enabling Strict-Transport-Security headers please read into this
\t# topic first.
\t# add_header Strict-Transport-Security "max-age=15552000;
\t# includeSubDomains; preload;";
\t#
\t# WARNING: Only add the preload option once you read about
\t# the consequences in https://hstspreload.org/. This option
\t# will add the domain to a hardcoded list that is shipped
\t# in all major browsers and getting removed from this list
\t# could take several months.
\tadd_header X-Content-Type-Options nosniff;
\tadd_header X-XSS-Protection "1; mode=block";
\tadd_header X-Robots-Tag none;
\tadd_header X-Download-Options noopen;
\tadd_header X-Permitted-Cross-Domain-Policies none;
\tadd_header 'Referrer-Policy' 'strict-origin';


\t# Path to the root of your installation
\troot /var/www/nextcloud/;

\tlocation = /data/htaccesstest.txt {
\t\tallow all;
\t\tlog_not_found off;
\t\taccess_log off;
\t}

\tlocation = /robots.txt {
\t\tallow all;
\t\tlog_not_found off;
\t\taccess_log off;
\t}

\tlocation = /.well-known/carddav {
\t\treturn 301 \$scheme://\$host/remote.php/dav;
\t}
\tlocation = /.well-known/caldav {
\t\treturn 301 \$scheme://\$host/remote.php/dav;
\t}

\t# set max upload size
\tclient_max_body_size 512M;
\tfastcgi_buffers 64 4K;

\t# Enable gzip but do not remove ETag headers
\tgzip on;
\tgzip_vary on;
\tgzip_comp_level 4;
\tgzip_min_length 256;
\tgzip_proxied expired no-cache no-store private no_last_modified no_etag auth;
\tgzip_types application/atom+xml application/javascript application/json application/ld+json application/manifest+json application/rss+xml application/vnd.geo+json application/vnd.ms-fontobject application/x-font-ttf application/x-web-app-manifest+json application/xhtml+xml application/xml font/opentype image/bmp image/svg+xml image/x-icon text/cache-manifest text/css text/plain text/vcard text/vnd.rim.location.xloc text/vtt text/x-component text/x-cross-domain-policy;

\tlocation / {
\t\trewrite ^ /index.php\$request_uri;
\t}

\tlocation ~ ^/(?:build|tests|config|lib|3rdparty|templates|data)/ {
\t\tdeny all;
\t}
\tlocation ~ ^/(?:\.|autotest|occ|issue|indie|db_|console) {
\t\tdeny all;
\t}

\tlocation ~ ^/(?:index|remote|public|cron|core/ajax/update|status|ocs/v[12]|updater/.+|ocs-provider/.+)\.php(?:\$|/) {
\t\tfastcgi_split_path_info ^(.+?\.php)(/.*)\$;
\t\tinclude fastcgi_params;
\t\tfastcgi_param SCRIPT_FILENAME \$document_root\$fastcgi_script_name;
\t\tfastcgi_param PATH_INFO \$fastcgi_path_info;
\t\tfastcgi_param HTTPS on;
\t\t#Avoid sending the security headers twice
\t\tfastcgi_param modHeadersAvailable true;
\t\tfastcgi_param front_controller_active true;
\t\tfastcgi_pass php-handler;
\t\tfastcgi_intercept_errors on;
\t\tfastcgi_request_buffering off;
\t}

\tlocation ~ ^/(?:updater|ocs-provider)(?:\$|/) {
\t\ttry_files \$uri/ =404;
\t\tindex index.php;
\t}

\t# Adding the cache control header for js and css files
\t# Make sure it is BELOW the PHP block
\tlocation ~ \.(?:css|js|woff|svg|gif)\$ {
\t\ttry_files \$uri /index.php\$request_uri;
\t\tadd_header Cache-Control "public, max-age=15778463";
\t\t# Add headers to serve security related headers (It is intended to
\t\t# have those duplicated to the ones above)
\t\t# Before enabling Strict-Transport-Security headers please read into
\t\t# this topic first.
\t\t# add_header Strict-Transport-Security "max-age=15768000; includeSubDomains; preload;";
\t\t#
\t\t# WARNING: Only add the preload option once you read about
\t\t# the consequences in https://hstspreload.org/. This option
\t\t# will add the domain to a hardcoded list that is shipped
\t\t# in all major browsers and getting removed from this list
\t\t# could take several months.
\t\tadd_header X-Content-Type-Options nosniff;
\t\tadd_header X-XSS-Protection "1; mode=block";
\t\tadd_header X-Robots-Tag none;
\t\tadd_header X-Download-Options noopen;
\t\tadd_header X-Permitted-Cross-Domain-Policies none;
\t\t# Optional: Don't log access to assets
\t\taccess_log off;
\t}

\tlocation ~ \.(?:png|html|ttf|ico|jpg|jpeg)\$ {
\t\ttry_files \$uri /index.php\$request_uri;
\t\t# Optional: Don't log access to other assets
\t\taccess_log off;
\t}

\tinclude\t\t/etc/nginx/ssl_params;

}
EOF
```

Check if the nginx configuration is correct:  
```shell
sudo nginx -t
```

Install a Let's encrypt SSL Certificate:

!!! Tip
	Don't add a redirect to HTTPS.

```shell
sudo certbot --nginx -d cloud.pphg.tech && \
sudo sed -i '/ssl_certificate_key/a \ \ \ \ ssl_trusted_certificate /etc/letsencrypt/live/cloud.pphg.tech/chain.pem;' /etc/nginx/sites-available/10-cloud.pphg.tech.conf && \
sudo systemctl reload nginx.service
```

Add cronjob for renewing cetificates:  
`sudo crontab -e`:  
```
0 */12 * * * /usr/local/bin/certbot renew
```

### Download Nextcloud

Install required packages:  
```shell
sudo apt install wget unzip zip -y
```

Download newstest Nextcloud stable release into `/var/www/`:  
```shell
cd /var/www/ && \
sudo wget https://download.nextcloud.com/server/releases/latest.zip && \
sudo unzip latest.zip && \
sudo rm latest.zip && \
sudo chown -R www-data:www-data /var/www/nextcloud/
```

Create a Nextcloud Data directory:  
```shell
sudo -u www-data mkdir /mnt/cifs/ncdata
```

Add the following to the `/var/www/nextcloud/config/config.php` file:  
```php
'memcache.local' => '\OC\Memcache\APCu',
```

### Install NextCloud

Go to <https://cloud.pphg.tech> and use the following varaibles:  
* User: `root`
* PW: `***`
* Data folder: `/mnt/cifs/ncdata/`
* Database user: `nextcloud`
* Database pw: `***`
* Database name: `nextcloud`

* Enable the Audit / Logging App!

Afterwards go to `Settings > Basic settings` and setup the email server:  
* Background jobs
	* Cron
* Email server
	* to your needs



## Security

### iptables

??? Tip "Explanation iptables rules"
    ```shell
    # Allow loopback
    iptables -A OUTPUT -o lo -j ACCEPT
    iptables -A INPUT -i lo -j ACCEPT

    # Allow SSH incoming
    iptables -t filter -A INPUT -i ens3 -p tcp --dport 22 -m state --state NEW,ESTABLISHED,RELATED -j ACCEPT

    # Allow quassel incoming
    iptables -t filter -A INPUT -i ens3 -p tcp --dport 4242 -m state --state NEW,ESTABLISHED,RELATED -j ACCEPT

    # Allow ESTABLISHED and RELATED connection (important for outgoing connections!)
    iptables -t filter -A INPUT -i ens3 -m state --state ESTABLISHED,RELATED -j ACCEPT

    # Policy DROP INPUT on
    iptables -P INPUT DROP

    # Policy ACCEPT OUTPUT
    iptables -P OUTPUT ACCEPT
    ```

Set up needed iptables rules:  
```shell
sudo iptables -A OUTPUT -o lo -j ACCEPT; \
sudo iptables -A INPUT -i lo -j ACCEPT; \
sudo iptables -t filter -A INPUT -i ens3 -p tcp --dport 22 -m state --state NEW,ESTABLISHED,RELATED -j ACCEPT; \
sudo iptables -t filter -A INPUT -i ens3 -p tcp -m multiport --dports 80,443 -m state --state NEW,ESTABLISHED,RELATED -j ACCEPT; \
sudo iptables -t filter -A INPUT -i ens3 -m state --state ESTABLISHED,RELATED -j ACCEPT; \
sudo iptables -t filter -A INPUT -i ens3 -p icmp -j ACCEPT; \
sudo iptables -P INPUT DROP; \
sudo iptables -P OUTPUT ACCEPT
```
Set up needed ip6tables rules:  
```shell
sudo ip6tables -A OUTPUT -o lo -j ACCEPT; \
sudo ip6tables -A INPUT -i lo -j ACCEPT; \
sudo ip6tables -t filter -A INPUT -i ens3 -p tcp --dport 22 -m state --state NEW,ESTABLISHED,RELATED -j ACCEPT; \
sudo ip6tables -t filter -A INPUT -i ens3 -p tcp -m multiport --dports 80,443 -m state --state NEW,ESTABLISHED,RELATED -j ACCEPT; \
sudo ip6tables -t filter -A INPUT -i ens3 -m state --state ESTABLISHED,RELATED -j ACCEPT; \
sudo ip6tables -t filter -A INPUT -i ens3 -p ipv6-icmp -j ACCEPT; \
sudo ip6tables -P INPUT DROP; \
sudo ip6tables -P OUTPUT ACCEPT
```

Persist iptables rules:  
```shell
sudo apt install -y iptables-persistent && \
sudo netfilter-persistent save && \
sudo netfilter-persistent reload
```

### fail2Ban (SSH/Nextcloud)



### Backup Nextcloud