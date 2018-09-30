# Concourse

OS: Ubuntu 16.04 LTS

## Initial Server setup

### Update System

```shell
apt update && apt full-upgrade -y
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
cat << EOF >> /etc/vim/vimrc

" Set background to dark for better readability in SSH connections
set background=dark
EOF
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
unzip \
unrar \
screen \
tmux \
htop
```

### Fix locale

```shell
cat << EOF | sudo tee -a /etc/environment

# Fix locale
LC_ALL=en_US.UTF-8
LANGUAGE=en_US:en
EOF
```

## PostgreSQL

### Install

```shell
sudo apt install -y postgresql postgresql-contrib
```

### Configure

Once the database software is installed, we will create a dedicated PostgreSQL user named concourse to manage the `Concourse` assets within the database system. To create this user, we will use `sudo` to act as the `postgres` system user, which has admin access to the database system:  
```shell
sudo -u postgres createuser concourse
```

By default, Concourse attempts to connect to a database called `atc`. Concourse calls its main web and API hub the "ATC", which stands for "air traffic control". We can create this database and assign ownership to the `concourse` database user to provide appropriate access:  
```shell
sudo -u postgres createdb --owner=concourse atc
```

### Backup



## Concourse

### Install

Download and install Concourse and fly:  
```shell
sudo wget -O /root/concourse_updater.sh https://gist.githubusercontent.com/shokinn/9eb8b9e39e8a73e4ad085cd9c75a3b4f/raw/3c3b29cc08c927bdd60253050b248d9e9f33d67d/concourse_updater.sh && \
sudo chmod u+x /root/concourse_updater.sh && \
sudo /root/concourse_updater.sh; \
sudo wget -O /root/fly_updater.sh https://gist.githubusercontent.com/shokinn/9eb8b9e39e8a73e4ad085cd9c75a3b4f/raw/3c3b29cc08c927bdd60253050b248d9e9f33d67d/fly_updater.sh && \
sudo chmod u+x /root/fly_updater.sh && \
sudo /root/fly_updater.sh
```

### Configure

Next, we can start to put together the configuration and keys that Concourse will use to start up.

Before we begin, create a configuration directory where we can keep all of the relevant files:  

```shell
sudo mkdir /etc/concourse
```

#### Creating the Key Files

Concourse is composed of a few related components that all need to be able to communicate securely with one another.

The [ATC](https://concourse-ci.org/concepts.html#component-atc) is the main hub that handles web and API requests and coordinates pipelines. [Workers](https://concourse-ci.org/concepts.html#architecture-worker) manage containers to run the CI/CD tasks defined in the pipeline. The [TSA](https://concourse-ci.org/concepts.html#component-tsa) is a custom SSH server that securely registers workers with the ATC.

Even though we will be running all of these components on a single server, the worker and TSA expect to communicate securely. To satisfy this expectation, we will create three sets of keys:

* a key pair for the TSA component
* a key pair for the worker
* a session signing key pair used to sign tokens for user sessions and TSA to ATC communication

```shell
sudo ssh-keygen -t rsa -q -N '' -f /etc/concourse/tsa_host_key; \
sudo ssh-keygen -t rsa -q -N '' -f /etc/concourse/worker_key; \
sudo ssh-keygen -t rsa -q -N '' -f /etc/concourse/session_signing_key
```

If we check in the concourse directory, we can see that three public and three private keys are now available:  
```shell
ls -l /etc/concourse
```

Output:  
```
total 24
-rw------- 1 root root 1679 May 11 17:19 session_signing_key
-rw-r--r-- 1 root root  394 May 11 17:19 session_signing_key.pub
-rw------- 1 root root 1679 May 11 17:19 tsa_host_key
-rw-r--r-- 1 root root  394 May 11 17:19 tsa_host_key.pub
-rw------- 1 root root 1675 May 11 17:19 worker_key
-rw-r--r-- 1 root root  394 May 11 17:19 worker_key.pub

```

The TSA will decide which workers are authorized to connect to the system by checking an authorized key file. We need to pre-populate the authorized keys file with the worker's public key that we generated so that it can connect successfully.

Since this is our only worker, we can just copy the file over:  
```shell
sudo cp /etc/concourse/worker_key.pub /etc/concourse/authorized_worker_keys
```

Now that we have the key files and an initial file for authorized workers, we can create the files that will define our Concourse configuration.

#### Creating the Environment Configuration Files

The Concourse binary does not read from a configuration file natively. However, it can take configuration values from environment variables passed in when the process starts.

In a moment, we will be creating `systemd` unit files to define and manage our Concourse services. The unit files can read environment variables from a file and pass them to the process as it starts. We will create a file that defines the variables for the Concourse `web` process, which start the ATC and TSA components, and another file for the Concourse `worker` process.

Create a file for the `web` process.

Inside, we will define the environment variables needed by the ATC and TSA components. Each variable begins with `CONCOURSE_`.
To start, we will define some static values that we don't need to modify. These variables will define the location of the private TSA and session keys, the file defining the authorized workers, and the PostgreSQL socket location:  
```shell
cat << EOF | sudo tee /etc/concourse/web_environment
# These values can be used as-is
CONCOURSE_SESSION_SIGNING_KEY=/etc/concourse/session_signing_key
CONCOURSE_TSA_HOST_KEY=/etc/concourse/tsa_host_key
CONCOURSE_TSA_AUTHORIZED_KEYS=/etc/concourse/authorized_worker_keys
CONCOURSE_POSTGRES_SOCKET=/var/run/postgresql
EOF
```

Next, we'll set some variables that will need to be changed to match your environment. The `CONCOURSE_EXTERNAL_URL` defines the IP address and port that the service will bind to. Set this to your server's public IP address and port 8080.

We will also set a username and password for the `main` team, which functions as the Concourse administrative group. You can select any username and password you'd like here. You can change the admin credentials at any time by modifying these values and restarting the service:  
```shell
while true; do \
unset pw; \
unset pw_confirm; \
read -s -p "Enter admin user password: " pw; echo "" && \
read -s -p "Confirm admin user password: " pw_confirm; echo "" && \
if [[ "$pw" == "$pw_confirm" ]]; then \
break; \
else \
echo "Your passwords don't match. Try again!"; \
echo ""; \
fi; \
done; \
read -p "Enter the public URL for concours (without http(s)://): " ext_url; \
cat << EOF | sudo tee -a /etc/concourse/web_environment; \
unset pw; \
unset pw_confirm; \
unset ext_url

# Change these values to match your environment
CONCOURSE_BASIC_AUTH_USERNAME=admin
CONCOURSE_BASIC_AUTH_PASSWORD=$pw
CONCOURSE_EXTERNAL_URL=https://$ext_url
CONCOURSE_BIND_IP=127.0.0.1
EOF
```

Next, create an environment file for the worker process.

Inside, we will define the locations of the worker's private key, the TSA's public key, and the directory where the worker will store its files. We will also set the address where the TSA can be reached, which will be the localhost in our case. You can use the values below without modification:  
```shell
cat << EOF | sudo tee /etc/concourse/worker_environment
# These values can be used as-is
CONCOURSE_WORK_DIR=/var/lib/concourse
CONCOURSE_TSA_WORKER_PRIVATE_KEY=/etc/concourse/worker_key
CONCOURSE_TSA_PUBLIC_KEY=/etc/concourse/tsa_host_key.pub
CONCOURSE_TSA_HOST=127.0.0.1:2222
EOF
```

#### Creating a Dedicated System User and Adjusting Permissions

Before we move on, we should create a dedicated Linux user to run the Concourse `web` process. This will allow us to start the web-facing service with limited privileges.

Because of the way that PostgreSQL handles authentication by default, it is important that the username match the PostgreSQL username we created earlier. Create a system user and group called `concourse` by typing:  
```shell
sudo adduser --system --group concourse
```

We can give the new user ownership over the `/etc/concourse` directory and its contents by typing:  
```shell
sudo chown -R concourse:concourse /etc/concourse
```

The environment files contain some sensitive data like the administrative username and password for the CI server. Adjust the permissions of the environment files so that regular users cannot read or modify the values in those files:  
```shell
sudo chmod 600 /etc/concourse/*_environment
```

Our configuration assets are now owned by the `concourse` system user with limited privileges for other users.

#### Create Systemd Unit Files for the Web and Worker Processes

We are now ready to define the Concourse CI unit files that will start and manage the application processes. We will create one file for the `web` process that handles the TSA and ATC components and one file for the `worker` process that handles containers for pipeline tasks.

##### Creating the Concourse Web Unit File

Start by creating a `concourse-web.service` file within the `/etc/systemd/system` file with the following content:  
```shell
cat << EOF | sudo tee /etc/systemd/system/concourse-web.service
[Unit]
Description=Concourse CI web process (ATC and TSA)
After=postgresql.service

[Service]
User=concourse
Restart=on-failure
EnvironmentFile=/etc/concourse/web_environment
ExecStart=/usr/local/bin/concourse web

[Install]
WantedBy=multi-user.target
EOF
```

The first section of the file sets the unit description for the `web` process and indicates that this unit should be started after the PostgreSQL unit when deciding on ordering.

The `[Service]` section defines the way that the service will be run. We will run the service as the `concourse` user we configured earlier and we tell systemd to automatically restart the service if it fails, which can be useful if the process dies from memory constraints or similar issues. We load the `web_environment` file we defined earlier to establish the environment and we start the actual process by calling `concourse web`.

The `[Install]` section tells systemd how to tie the unit to the system start order if we configure the service to start at boot.

##### Creating the Concourse Worker Unit File

Next, create a similar file to define the worker process with the following content:  
```shell
cat << EOF | sudo tee /etc/systemd/system/concourse-worker.service
[Unit]
Description=Concourse CI worker process
After=concourse-web.service

[Service]
User=root
Restart=on-failure
EnvironmentFile=/etc/concourse/worker_environment
ExecStart=/usr/local/bin/concourse worker

[Install]
WantedBy=multi-user.target
EOF
```

This unit functions similarly to the `concourse-web` unit. This time, we tell system to start the `worker` process after the Concourse `web` process has been started. The `worker` process is run as the `root` user instead of `concourse` because it requires administrative privileges for container management. We load the `worker_environment` file and use the `concourse worker` command to start the process.

#### Start and enable the Services

Reload the deamon files, start the services:  
```shell
sudo systemctl daemon-reload; \
sudo systemctl start concourse-web concourse-worker; \
sudo systemctl status concourse-web concourse-worker
```

Check that both services read "active (running)" and that the log lines do not contain any obvious errors. Pay special attention to the `web` service to make sure that the log lines do not indicate problems connecting to the database.

If the services started successfully, enable them so that they will start each time the server boots:  
```shell
sudo systemctl enable concourse-web concourse-worker
```

### Check Access On the Command Line

Now that the Concourse services is running, we should check that we have access.

#### Checking Access On the Command Line

First, let's check that we can access the Concourse service with the `fly` command line client.

We have to log in using the administrative username and password that we configured in the `/etc/concourse/web_environment` file using the `login` subcommand. A single `fly` binary can be used to contact and manage multiple Concourse servers, so the command uses a concept called "targets" as an alias for different servers. We will call our target "local" to log into the local Concourse server:  
```shell
fly -t local login -c http://127.0.0.1:8080
```

You will be prompted for the username and password for the `main` team, which we set in the `web_environments` file. After entering your credentials, "target saved" should be displayed:  
```
logging in to team 'main'

username: admin
password: 

target saved
```

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

#### Concourse

Add server directive for Concourse:  
```shell
cat << EOF | sed 's/\\t/\t/g' | sudo tee /etc/nginx/sites-available/10-ci.pphg.tech.conf && \
sudo ln -s ../sites-available/10-ci.pphg.tech.conf /etc/nginx/sites-enabled/10-ci.pphg.tech.conf
upstream concourse {
\tserver\t\t127.0.0.1:8080;
}

server {
\tlisten\t\t443 ssl http2;
\tlisten\t\t[::]:443 ssl http2;
\tserver_name\tci.pphg.tech;

\taccess_log\t/var/log/nginx/ci.pphg.tech_access.log combined gzip=9;
\terror_log\t/var/log/nginx/ci.pphg.tech_error.log warn;

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
sudo certbot --nginx -d ci.pphg.tech && \
sudo sed -i '/ssl_certificate_key/a \ \ \ \ ssl_trusted_certificate /etc/letsencrypt/live/ci.pphg.tech/chain.pem;' /etc/nginx/sites-available/10-ci.pphg.tech.conf && \
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
sudo ip6tables -t filter -A INPUT -i eth0 -j DROP; \
sudo ip6tables -t filter -A OUTPUT -o eth0 -j ACCEPT
```

Persist iptables rules:  
```shell
sudo apt install -y iptables-persistent && \
sudo netfilter-persistent save && \
sudo netfilter-persistent reload
```



