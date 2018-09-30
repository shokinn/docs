# ZNC

ZNC is an advanced IRC bouncer that is left connected so an IRC client can disconnect/reconnect without losing the chat session.

OS: Ubuntu 18.04 LTS (Bionic Beaver)

## Initial Server setup

### Update System

```shell
apt update && apt full-upgrade -y && \
apt autoremove -y
```

### Make vim colored with dark background settings

```shell
cat << EOF >> /etc/vim/vimrc

" Set background to dark for better readability in SSH connections
set background=dark
EOF
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

### Create a new user `user`

```shell
adduser user && \
usermod -aG sudo user && \
mkdir /home/user/.ssh && \
chmod 700 /home/user/.ssh && \
cp /root/.ssh/authorized_keys /home/user/.ssh/ && \
chmod 400 /home/user/.ssh/authorized_keys && \
chown -R user:user /home/user/.ssh/
```

### Change the SSH deamon to allow only SSH-keys

```shell
cat << EOF > /etc/ssh/sshd_config && systemctl restart sshd.service
#	$OpenBSD: sshd_config,v 1.101 2017/03/14 07:19:07 djm Exp $

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
AuthorizedKeysFile	.ssh/authorized_keys .ssh/authorized_keys2

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
Subsystem	sftp	/usr/lib/openssh/sftp-server

# Example of overriding settings on a per-user basis
#Match User anoncvs
#	X11Forwarding no
#	AllowTcpForwarding no
#	PermitTTY no
#	ForceCommand cvs server
EOF
```

#### Delete root's `authorized_keys` file

!!! Attention
	Please check before if you can login to the user `user` with your ssh-key!

```shell
rm ~/.ssh/authorized_keys
```

!!! Important
	Log out and re login as `user`!

### Install base packages

```shell
sudo apt install -y \
p7zip-full \
p7zip-rar \
zip \
unzip \
unrar \
screen \
tmux \
htop
```

## Install ZNC

You can find a detailed install instruction here: <https://wiki.znc.in/Installation#Ubuntu>

### Pre-requiemetns

Install python-software-properties:  
```shell
sudo apt install -y software-properties-common
```

Create user for running znc:  
```shell
sudo useradd -r -s /usr/sbin/nologin znc
```

Create a config directory for znc:  
```shell
sudo mkdir /etc/znc && \
sudo chown znc:znc /etc/znc; \
sudo chmod 750 /etc/znc
```

### Install ZNC

Add PPA and install:  
```shell
sudo add-apt-repository ppa:teward/znc && \
sudo apt update && \
sudo apt install -y znc znc-dev znc-perl znc-python znc-tcl
```

### Configure ZNC

```shell
sudo -u znc znc -c -d /etc/znc
```

Configuration Options:  
```
[ .. ] Checking for list of available modules...
[ ** ]
[ ** ] -- Global settings --
[ ** ]
[ ?? ] Listen on port (1025 to 65534): 6667
[ !! ] WARNING: Some web browsers reject port 6667. If you intend to
[ !! ] use ZNC's web interface, you might want to use another port.
[ ?? ] Proceed with port 6667 anyway? (yes/no) [yes]:
[ ?? ] Listen using SSL (yes/no) [no]: yes
[ ?? ] Listen using both IPv4 and IPv6 (yes/no) [yes]:
[ .. ] Verifying the listener...
[ ** ] Unable to locate pem file: [/etc/znc/znc.pem], creating it
[ .. ] Writing Pem file [/etc/znc/znc.pem]...
[ ** ] Enabled global modules [webadmin]
[ ** ]
[ ** ] -- Admin user settings --
[ ** ]
[ ?? ] Username (alphanumeric): admin
[ ?? ] Enter password:
[ ?? ] Confirm password:
[ ?? ] Nick [admin]:
[ ?? ] Alternate nick [admin_]:
[ ?? ] Ident [admin]:
[ ?? ] Real name (optional):
[ ?? ] Bind host (optional):
[ ** ] Enabled user modules [chansaver, controlpanel]
[ ** ]
[ ?? ] Set up a network? (yes/no) [yes]:
[ ** ]
[ ** ] -- Network settings --
[ ** ]
[ ?? ] Name [freenode]:
[ ?? ] Server host [chat.freenode.net]: ^C
user@znc /etc $ sudo -u znc znc -c -d /etc/znc
[ .. ] Checking for list of available modules...
[ ** ]
[ ** ] -- Global settings --
[ ** ]
[ ?? ] Listen on port (1025 to 65534): 6667
[ !! ] WARNING: Some web browsers reject port 6667. If you intend to
[ !! ] use ZNC's web interface, you might want to use another port.
[ ?? ] Proceed with port 6667 anyway? (yes/no) [yes]:
[ ?? ] Listen using SSL (yes/no) [no]: yes
[ ?? ] Listen using both IPv4 and IPv6 (yes/no) [yes]:
[ .. ] Verifying the listener...
[ ** ] Enabled global modules [webadmin]
[ ** ]
[ ** ] -- Admin user settings --
[ ** ]
[ ?? ] Username (alphanumeric): admin
[ ?? ] Enter password:
[ ?? ] Confirm password:
[ ?? ] Nick [admin]:
[ ?? ] Alternate nick [admin_]:
[ ?? ] Ident [admin]:
[ ?? ] Real name (optional):
[ ?? ] Bind host (optional):
[ ** ] Enabled user modules [chansaver, controlpanel]
[ ** ]
[ ?? ] Set up a network? (yes/no) [yes]: no
[ ** ]
[ .. ] Writing config [/etc/znc/configs/znc.conf]...
[ ** ]
[ ** ] To connect to this ZNC you need to connect to it as your IRC server
[ ** ] using the port that you supplied.  You have to supply your login info
[ ** ] as the IRC server password like this: user/network:pass.
[ ** ]
[ ** ] Try something like this in your IRC client...
[ ** ] /server <znc_server_ip> +6667 admin:<pass>
[ ** ]
[ ** ] To manage settings, users and networks, point your web browser to
[ ** ] https://<znc_server_ip>:6667/
[ ** ]
[ ?? ] Launch ZNC now? (yes/no) [yes]: no
```

### ZNC service

```shell
cat << EOF | sudo tee /etc/systemd/system/znc.service && \
sudo systemctl daemon-reload && \
sudo systemctl enable znc
[Unit]
Description=ZNC, an advanced IRC bouncer
After=network-online.target
     
[Service]
ExecStart=/usr/bin/znc -f --datadir=/etc/znc
User=znc
     
[Install]
WantedBy=multi-user.target
EOF
```

## Nginx

### Install

Install all needed Packages for nginx tasks: 
```shell
sudo apt update && \
sudo apt install -y \
nginx \
python3-pip && \
sudo -H pip3 install --system --upgrade \
pip && \
sudo -H pip3 install --upgrade \
cryptography && \
sudo -H pip3 install --upgrade \
setuptools && \
sudo -H pip3 install \
certbot \
certbot-nginx
```

### Configure

#### Delete default entry

```shell
sudo rm /etc/nginx/sites-enabled/default
```

#### Add ssl_params

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

#### General HTTP to HTTPS redirector

This nginx entry will rewrite all traffic from HTTP to HTTPS.
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

#### ZNC web interface on Port 443

Add server directive for Portainer:  
```shell
cat << EOF | sed 's/\\t/\t/g' | sudo tee /etc/nginx/sites-available/10-znc.mischaufen.de.conf && \
sudo ln -s ../sites-available/10-znc.mischaufen.de.conf /etc/nginx/sites-enabled/10-znc.mischaufen.de.conf
server {
\tlisten\t\t443 ssl http2;
\tlisten\t\t[::]:443 ssl http2;
\tserver_name\tznc.mischaufen.de;

\taccess_log\t/var/log/nginx/znc.mischaufen.de_access.log combined gzip=9;
\terror_log\t/var/log/nginx/znc.mischaufen.de_error.log warn;

\tlocation / {
\t\tproxy_pass\t\thttps://[::1]:6667/;
\t\tproxy_set_header\tHost \$host;
\t\tproxy_set_header\tX-Forwarded-Host \$server_name;
\t\tproxy_set_header\tX-Forwarded-For \$proxy_add_x_forwarded_for;
\t}

\tinclude\t\t/etc/nginx/ssl_params;

}
EOF
```

Install a Let's encrypt SSL Certificate:

!!! Tip
    Don't add a redirect to HTTPS.

```shell
sudo certbot --nginx -d znc.mischaufen.de && \
sudo sed -i '/ssl_certificate_key/a \ \ \ \ ssl_trusted_certificate /etc/letsencrypt/live/znc.mischaufen.de/chain.pem;' /etc/nginx/sites-available/10-znc.mischaufen.de.conf && \
sudo systemctl reload nginx.service
```

### Add cronjob for renewing cetificates

This cronjob renews the certificat and create one for znc.

`sudo crontab -e`:  
```
0 */12 * * * /usr/local/bin/certbot renew && cat /etc/letsencrypt/live/znc.mischaufen.de/{privkey,cert,chain}.pem > /etc/znc/znc.pem
```

## ZNC SSL config

### Initial copy of our let's encrypt certificates to znc

```shell
sudo cat /etc/letsencrypt/live/znc.mischaufen.de/{privkey,cert,chain}.pem | sudo tee /etc/znc/znc.pem
```

### Run ZNC for the 1st time

```shell
sudo systemctl start znc.service
```

