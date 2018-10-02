# Concourse dockerized

OS: Ubuntu 18.04 LTS (Bionic Beaver)

## Initial Server setup

### Update System

```shell
apt update && apt full-upgrade -y
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
```bash
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

## Docker

### Pre-requirements

```shell
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
```

### Install Docker

!!! info "The following steps are shortened!"
    Here is the full Guide to install docker on Ubuntu 18.04 (Bionic Beaver)  
    <https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-18-04>

```shell
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add - && \
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable" && \
sudo apt update && \
apt-cache policy docker-ce && \
echo "Verify that docker-ce will be installed from the docker repository instead of the ubuntu repo." && \
read -p "Press any key to continue or press CTRL-C to abort... " -n1 -s && \
sudo apt install -y docker-ce && \
sudo systemctl status docker
```

### Create a new docker network for concourse

```shell
sudo docker network create \
  -d bridge \
  -o "com.docker.network.bridge.name"="docker1" \
  -o "com.docker.network.bridge.enable_ip_masquerade"=true \
  -o "com.docker.network.bridge.enable_icc"=true \
  -o "com.docker.network.bridge.host_binding_ipv4"="0.0.0.0" \
  -o "com.docker.network.driver.mtu"="1500" \
  --scope=local \
  --subnet=172.23.0.0/16 \
  --ip-range=172.23.42.0/24 \
  --gateway=172.23.0.1 \
  concourse_net
```

### Install Portainer

```shell
sudo docker volume create portainer_data && \
sudo docker run \
  --name portainer \
  --volume /var/run/docker.sock:/var/run/docker.sock \
  --volume portainer_data:/data \
  --network concourse_net \
  --ip 172.23.0.10 \
  --restart unless-stopped \
  --detach \
  portainer/portainer
```

## PostgreSQL

### Install

```shell
PSQL_CONCOURSE_USER=$(cat /dev/urandom | tr -dc 'a-zA-Z0-9' | fold -w 32 | head -n 1) && \
PSQL_CONCOURSE_PW=$(cat /dev/urandom | tr -dc 'a-zA-Z0-9' | fold -w 32 | head -n 1) && \
sudo docker volume create pgdata && \
sudo docker run \
  --name concourse-db \
  --volume pgdata:/var/lib/postgresql/data \
  --network concourse_net \
  --hostname concourse-db \
  --ip 172.23.1.10 \
  --restart unless-stopped \
  --env POSTGRES_USER=$PSQL_CONCOURSE_USER \
  --env POSTGRES_PASSWORD=$PSQL_CONCOURSE_PW \
  --env POSTGRES_DB=atc \
  --detach \
  postgres:10.4-alpine && \
echo -e "User:\t\t$PSQL_CONCOURSE_USER"; \
echo -e "Password:\t$PSQL_CONCOURSE_PW"
```

### Backup

\#TODO

## Concourse

### Pre-requirements

Create a GitHub OAuth app for authentication:

Follow this Guide:  
<https://concourse-ci.org/install.html#github-auth-config>

### Install Web interface

```shell
echo "" && \
read -p "Enter your GitHub client ID: " GITHUB_CLIENT_ID && \
while true; do \
set GITHUB_CLIENT_SECRET=""; \
set GITHUB_CLIENT_SECRET_confirm=""; \
read -s -p "Enter your GitHub client secret: " GITHUB_CLIENT_SECRET; echo ""; \
read -s -p "Reenter your GitHub client secret: " GITHUB_CLIENT_SECRET_confirm; echo ""; \
if [[ "$GITHUB_CLIENT_SECRET" == "$GITHUB_CLIENT_SECRET_confirm" ]]; then \
break; \
else \
clear; \
echo "Your secrets don't match! Please try again."; \
echo ""; \
fi; \
done; \
CONCOURSE_ADMIN_USER=admin && \
CONCOURSE_ADMIN_PW=$(cat /dev/urandom | tr -dc 'a-zA-Z0-9' | fold -w 32 | head -n 1) && \
CONCOUSE_WEB_IP='172.23.1.11' && \
ORG_NAME="mischaufen" && \
TEAM_NAME="main" && \
sudo docker volume create concourse-keys && \
sudo ssh-keygen -t rsa -q -N '' -f  /var/lib/docker/volumes/concourse-keys/_data/tsa_host_key && \
sudo chmod 600 /var/lib/docker/volumes/concourse-keys/_data/tsa_host_key && \
sudo touch /var/lib/docker/volumes/concourse-keys/_data/authorized_worker_keys && \
sudo chmod 600 /var/lib/docker/volumes/concourse-keys/_data/authorized_worker_keys && \
sudo ssh-keygen -t rsa -q -N '' -f /var/lib/docker/volumes/concourse-keys/_data/worker_key && \
sudo chmod 600 /var/lib/docker/volumes/concourse-keys/_data/worker_key && \
sudo cat /var/lib/docker/volumes/concourse-keys/_data/worker_key.pub | sudo tee -a /var/lib/docker/volumes/concourse-keys/_data/authorized_worker_keys && \
sudo ssh-keygen -t rsa -q -N '' -f /var/lib/docker/volumes/concourse-keys/_data/session_signing_key && \
sudo chmod 600 /var/lib/docker/volumes/concourse-keys/_data/session_signing_key && \
sudo docker run \
  --name concourse-web \
  --volume concourse-keys:/concourse-keys \
  --network concourse_net \
  --hostname concourse-web \
  --ip $CONCOUSE_WEB_IP \
  --privileged \
  --restart unless-stopped \
  --detach \
  concourse/concourse:4.2.1 web \
  --tsa-host-key='/concourse-keys/tsa_host_key' \
  --tsa-authorized-keys='/concourse-keys/authorized_worker_keys' \
  --tsa-session-signing-key='/concourse-keys/session_signing_key' \
  --add-local-user=$CONCOURSE_ADMIN_USER:$CONCOURSE_ADMIN_PW \
  --main-team-local-user=$CONCOURSE_ADMIN_USER \
  --github-client-id=$GITHUB_CLIENT_ID \
  --github-client-secret=$GITHUB_CLIENT_SECRET \
  --main-team-github-team=$ORG_NAME:$TEAM_NAME \
  --postgres-user=$PSQL_CONCOURSE_USER \
  --postgres-password=$PSQL_CONCOURSE_PW \
  --postgres-host=$(sudo docker inspect -f "{{ .NetworkSettings.Networks.concourse_net.IPAddress }}" concourse-db) \
  --postgres-port=5432 \
  --bind-ip=$CONCOUSE_WEB_IP \
  --external-url='https://ci.mischaufen.de' && \
echo -e "User:\t\t$CONCOURSE_ADMIN_USER"; \
echo -e "Password:\t$CONCOURSE_ADMIN_PW"
```


### Install worker

```shell
CONCOURSE_WORKER_IP=172.23.1.12 && \
sudo docker run \
  --name concourse-worker \
  --volume concourse-keys:/concourse-keys \
  --network concourse_net \
  --hostname concourse-worker \
  --ip $CONCOURSE_WORKER_IP \
  --privileged \
  --restart unless-stopped \
  --detach \
  concourse/concourse:4.2.1 worker \
  --tsa-host=$(sudo docker inspect -f "{{ .NetworkSettings.Networks.concourse_net.IPAddress }}" concourse-web):2222 \
  --garden-dns-server=1.1.1.1 \
  --garden-dns-proxy-enable
```

## fly

### Install fly

```shell
sudo wget -O /root/fly_updater.sh https://gist.githubusercontent.com/shokinn/9eb8b9e39e8a73e4ad085cd9c75a3b4f/raw/3c3b29cc08c927bdd60253050b248d9e9f33d67d/fly_updater.sh && \
sudo chmod u+x /root/fly_updater.sh && \
sudo /root/fly_updater.sh
```

### Configure

First, let's check that we can access the Concourse service with the `fly` command line client.

We have to log in using the administrative username and password that we configured in the `/etc/concourse/web_environment` file using the `login` subcommand. A single `fly` binary can be used to contact and manage multiple Concourse servers, so the command uses a concept called "targets" as an alias for different servers. We will call our target "local" to log into the local Concourse server:  
```shell
fly -t local login -c http://$(sudo docker inspect -f "{{ .NetworkSettings.Networks.concourse_net.IPAddress }}" concourse-web):8080
```

You will be prompted for going to `http://172.23.1.11:8080/sky/login?redirect_uri=http://127.0.0.1:33277/auth/callback`. Do the following instead:

Add a port forwarding (on your local PC) from `8080` to `172.23.1.11:8080` and go to:  
<http://127.0.0.1:8080/sky/login?redirect_uri=http://127.0.0.1:8080/sky/token>

Now copy/paste the token to your promt.

This indicates that we were able to log in successfully. While we are here, let's verify that the worker process was able to successfully register to the TSA component by typing:  
```shell
fly -t local workers
```

Output:  
```
name              containers  platform  tags  team  state    version
concourse-server  0           linux     none  none  running  2.1
```

The `fly` command is used to configure pipelines and manage the Concourse CI service. The `fly help` command provides information about additional commands.


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
setuptools && \
sudo -H pip3 install --upgrade \
cryptography && \
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

#### Portainer

Add server directive for Portainer:  
```shell
portainer_ip=$(sudo docker inspect -f "{{ .NetworkSettings.Networks.concourse_net.IPAddress }}" portainer); \
cat << EOF | sed 's/\\t/\t/g' | sudo tee /etc/nginx/sites-available/10-portainer.ci.mischaufen.de.conf && \
sudo ln -s ../sites-available/10-portainer.ci.mischaufen.de.conf /etc/nginx/sites-enabled/10-portainer.ci.mischaufen.de.conf
server {
\tlisten\t\t443 ssl http2;
\tlisten\t\t[::]:443 ssl http2;
\tserver_name\tportainer.ci.mischaufen.de;

\taccess_log\t/var/log/nginx/portainer.ci.mischaufen.de_access.log combined gzip=9;
\terror_log\t/var/log/nginx/portainer.ci.mischaufen.de_error.log warn;

\tlocation / {
\t\tset \$upstream_endpoint http://$portainer_ip:9000;
\t\tproxy_http_version\t1.1;
\t\tproxy_set_header\tConnection "";
\t\tproxy_set_header\tHost \$host;
\t\tproxy_set_header\tX-Forwarded-Host \$server_name;
\t\tadd_header\t\tX-Upstream \$upstream_addr;
\t\tproxy_pass\t\t\$upstream_endpoint;
\t}

\tlocation /api/websocket/ {
\t\tset \$upstream_endpoint http://$portainer_ip:9000;
\t\tproxy_buffering\t\toff;
\t\tproxy_set_header\tUpgrade \$http_upgrade;
\t\tproxy_set_header\tConnection "Upgrade";
\t\tproxy_set_header\tHost \$host;
\t\tproxy_set_header\tX-Forwarded-Server \$host;
\t\tproxy_set_header\tX-Forwarded-For \$proxy_add_x_forwarded_for;
\t\tproxy_set_header\tX-Forwarded-Host \$server_name;
\t\tadd_header\t\tX-Upstream \$upstream_addr;
\t\tproxy_http_version\t1.1;
\t\tproxy_pass\t\t\$upstream_endpoint;
\t\t# Need this for the console
\t\tproxy_redirect\t\thttp://$portainer_ip:9000 \$scheme://\$host/;
\t}

\tinclude\t\t/etc/nginx/ssl_params;

}
EOF
```

Install a Let's encrypt SSL Certificate:

!!! Tip
    Don't add a redirect to HTTPS.

```shell
sudo certbot --nginx -d portainer.ci.mischaufen.de && \
sudo sed -i '/ssl_certificate_key/a \ \ \ \ ssl_trusted_certificate /etc/letsencrypt/live/portainer.ci.mischaufen.de/chain.pem;' /etc/nginx/sites-available/10-portainer.ci.mischaufen.de.conf && \
sudo systemctl reload nginx.service
```

#### Concourse

Add server directive for Concourse:  
```shell
cat << EOF | sed 's/\\t/\t/g' | sudo tee /etc/nginx/sites-available/11-ci.mischaufen.de.conf && \
sudo ln -s ../sites-available/11-ci.mischaufen.de.conf /etc/nginx/sites-enabled/11-ci.mischaufen.de.conf
upstream concourse {
\tserver\t\t$(sudo docker inspect -f "{{ .NetworkSettings.Networks.concourse_net.IPAddress }}" concourse-web):8080;
}

server {
\tlisten\t\t443 ssl http2;
\tlisten\t\t[::]:443 ssl http2;
\tserver_name\tci.mischaufen.de;

\taccess_log\t/var/log/nginx/ci.mischaufen.de_access.log combined gzip=9;
\terror_log\t/var/log/nginx/ci.mischaufen.de_error.log warn;

\tlocation / {
\t\tinclude\t\t\tproxy_params;
\t\tproxy_http_version\t1.1;
\t\tproxy_read_timeout\t90;

\t\tproxy_set_header\tUpgrade \$http_upgrade;
\t\tproxy_set_header\tConnection "upgrade";

\t\tproxy_pass\t\thttp://concourse;
\t}

\tinclude\t\t/etc/nginx/ssl_params;

}
EOF
```

Check if config is ok:  
```shell
sudo nginx -t
```

Install a Let's encrypt SSL Certificate:

!!! Tip
	Don't add a redirect to HTTPS.

```shell
sudo certbot --nginx -d ci.mischaufen.de && \
sudo sed -i '/ssl_certificate_key/a \ \ \ \ ssl_trusted_certificate /etc/letsencrypt/live/ci.mischaufen.de/chain.pem;' /etc/nginx/sites-available/11-ci.mischaufen.de.conf && \
sudo systemctl reload nginx.service
```

### Add cronjob for renewing cetificates

`sudo crontab -e`:  
```
0 */12 * * * /usr/local/bin/certbot renew
```

## Security

### iptables

??? Tip "Explanation iptables rules"
	```shell
	# Allow loopback
	iptables -A OUTPUT -o lo -j ACCEPT
	iptables -A INPUT -i lo -j ACCEPT

	# Allow SSH incoming
	iptables -t filter -A INPUT -i eth0 -p tcp --dport 22 -m state --state NEW,ESTABLISHED,RELATED -j ACCEPT

	# Allow web interface SeedBox incoming
	iptables -t filter -A INPUT -i eth0 -p tcp -m multiport --dports 80,443 -m state --state NEW,ESTABLISHED,RELATED -j ACCEPT

	# Policy DROP INPUT on `eth0`
	iptables -t filter -A INPUT -i eth0 -j DROP;

	# Allow output on `eth0`
	iptables -t filter -A OUTPUT -o eth0 -j ACCEPT
	```

Set up needed iptables rules:  
```shell
sudo iptables -A OUTPUT -o lo -j ACCEPT; \
sudo iptables -A INPUT -i lo -j ACCEPT; \
sudo iptables -t filter -A INPUT -i eth0 -p tcp --dport 22 -m state --state NEW,ESTABLISHED,RELATED -j ACCEPT; \
sudo iptables -t filter -A INPUT -i eth0 -p tcp -m multiport --dports 80,443 -m state --state NEW,ESTABLISHED,RELATED -j ACCEPT; \
sudo iptables -t filter -A INPUT -i eth0 -m state --state ESTABLISHED,RELATED -j ACCEPT; \
sudo iptables -t filter -A INPUT -i eth0 -p icmp -j ACCEPT; \
sudo iptables -t filter -A INPUT -i eth0 -j DROP; \
sudo iptables -t filter -A OUTPUT -o eth0 -j ACCEPT
```
Set up needed ip6tables rules:  
```shell
sudo ip6tables -A OUTPUT -o lo -j ACCEPT; \
sudo ip6tables -A INPUT -i lo -j ACCEPT; \
sudo ip6tables -t filter -A INPUT -i eth0 -p tcp --dport 22 -m state --state NEW,ESTABLISHED,RELATED -j ACCEPT; \
sudo ip6tables -t filter -A INPUT -i eth0 -p tcp -m multiport --dports 80,443 -m state --state NEW,ESTABLISHED,RELATED -j ACCEPT; \
sudo ip6tables -t filter -A INPUT -i eth0 -m state --state ESTABLISHED,RELATED -j ACCEPT; \
sudo ip6tables -t filter -A INPUT -i eth0 -p ipv6-icmp -j ACCEPT; \
sudo ip6tables -t filter -A INPUT -i eth0 -j DROP; \
sudo ip6tables -t filter -A OUTPUT -o eth0 -j ACCEPT
```

Persist iptables rules:  
```shell
sudo apt install -y iptables-persistent && \
sudo netfilter-persistent save && \
sudo netfilter-persistent reload
```



