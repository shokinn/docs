# Quassel-IRC

OS: Ubuntu 18.04 LTS

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

## quassel core

### Pre requirements

Add the PPA for quassel to get the latest stable version:  
```shell
sudo add-apt-repository ppa:mamarley/quassel && \
sudo apt update
```

Install cerbot (needed for ssl):  

```shell
sudo apt install -y certbot
```

### Install quassel core

```shell
sudo apt install -y quassel-core && \
systemctl status quasselcore.service
```

### Setup Let's encrypt SSL

```shell
sudo certbot certonly --standalone -d quassel.mischaufen.de && \
sudo mv /var/lib/quassel/quasselCert.pem /var/lib/quassel/quasselCert.pem.old && \
sudo cat /etc/letsencrypt/live/quassel.mischaufen.de/{fullchain,privkey}.pem | sudo tee -a /var/lib/quassel/quasselCert.pem && \
sudo chown quasselcore:quassel /var/lib/quassel/quasselCert.pem && \
sudo chmod 600 /var/lib/quassel/quasselCert.pem && \
sudo systemctl restart quasselcore
```

### Setup automatic certificate renew

Add a deploy hook script to `/srv/certbot`:  
```shell
deamon='quasselcore.service'; \
target_domain='quassel.mischaufen.de'; \
sudo mkdir -p /srv/cerbot && \
sudo chmod 655 /srv/cerbot && \
cat << EOF | sed 's/\\t/\t/g' | sudo tee /srv/cerbot/deploy_hook.sh && \
sudo chmod 744 /srv/cerbot/deploy_hook.sh
#!/bin/bash

set -e

for domain in \$RENEWED_DOMAINS; do
\tcase \$domain in
\t$target_domain)
\t\tdaemon_cert_root='/var/lib/quassel'

\t\t# Make sure the certificate and private key files are
\t\t# never world readable, even just for an instant while
\t\t# we're copying them into daemon_cert_root.
\t\tumask 177

\t\trm \$daemon_cert_root/quasselCert.pem
\t\tcat \$RENEWED_LINEAGE/{fullchain,privkey}.pem >> "\$daemon_cert_root/quasselCert.pem"

\t\t# Apply the proper file ownership and permissions for
\t\t# the $deamon daemon to read its certificate and key.
\t\tchown quasselcore:quassel "\$daemon_cert_root/quasselCert.pem"
\t\tchmod 400 \$daemon_cert_root/quasselCert.pem

\t\tsystemctl restart $deamon >/dev/null
\t\t;;
\tesac
done
EOF
```

Add cronjob for renewing cetificates.

```shell
sudo crontab -e
```

Add at the end of the file:  
```
0 */12 * * * /usr/local/bin/certbot renew --deploy-hook /srv/cerbot/deploy_hook.sh
```

### Add user management scripts

#### Add user script

`quassel_add.sh`:  
```shell
cat << EOF | sudo tee /usr/local/sbin/quassel_add && \
sudo chmod 500 /usr/local/sbin/quassel_add
#!/bin/sh

sudo -u quasselcore quasselcore --configdir=/var/lib/quassel --add-user
EOF
```

`quassel_change_pw.sh`:  
```shell
cat << EOF | sudo tee /usr/local/sbin/quassel_change_pw && \
sudo chmod 500 /usr/local/sbin/quassel_change_pw
#!/bin/sh

die () {
    echo >&2 "\$@"
    exit 1
}

[ "\$#" -eq 1 ] || die "Username required!\n\nRun:\t'\$0 <username>'\ne.g.:\t'\$0 admin'"

systemctl stop quasselcore.service
sudo -u quasselcore quasselcore --configdir=/var/lib/quassel --change-userpass=\$1
systemctl start quasselcore.service
EOF
```

`quassel_del.sh`:  
```shell
if ! [ -x "$(command -v sqlite3)" ]; then sudo apt install -y sqlite3; fi && \
cat << EOF | sudo tee /usr/local/sbin/quassel_del && \
sudo chmod 500 /usr/local/sbin/quassel_del
#!/bin/sh
#
# Delete Quasselcore users from your SQLite database
#

exeq()
{
    # Execute SQL Query
    result=\$(sqlite3 "\${QUASSELDB}" "\${1}")
    echo "\${result}"
}

usage()
{
    echo "Usage: \${SCRIPT} username [database]"
}

print_users()
{
    sqlite3 "\${QUASSELDB}" "SELECT quasseluser.userid, quasseluser.username FROM quasseluser ORDER BY quasseluser.userid;"
}

# Main body

SCRIPT="\${0}"
QUASSELDB=""
USER=""

if [ -z "\${2}" ] ; then
    # No file supplied.
    QUASSELDB="/var/lib/quassel/quassel-storage.sqlite"
else
    QUASSELDB="\${2}"
fi

if [ -z "\${1}" ] ; then
    echo "No user supplied."
    echo "Pick one: "
    print_users
    usage
    exit 1
else
    USER="${1}"
fi

if [ -e "\${QUASSELDB}" ] ; then
    echo "SELECTED DB: \${QUASSELDB}"
else
    echo "SELECTED DB '\${QUASSELDB}' does not exist."
    usage
    exit 2
fi

if [ -z \$(exeq "SELECT quasseluser.username FROM quasseluser WHERE username = '\${USER}';") ] ; then
    echo "SELECTED USER '\${USER}' does not exist."
    print_users
    usage
    exit 3
else
    echo "SELECTED USER: \${USER}"
fi

# Sadly SQLITE does not allow DELETE statements that JOIN tables.
# All queries are written with a subquery.
# Contact me if you know a better way.

backlogq="DELETE
FROM backlog
WHERE backlog.bufferid in (
    SELECT bufferid
    FROM buffer, quasseluser
    WHERE buffer.userid = quasseluser.userid
    AND quasseluser.username = '\${USER}'
);"

bufferq="DELETE
FROM buffer
WHERE buffer.userid in (
    SELECT userid
    FROM quasseluser
    WHERE quasseluser.username = '\${USER}'
);"

ircserverq="DELETE
FROM ircserver
WHERE ircserver.userid in (
    SELECT userid
    FROM quasseluser
    WHERE quasseluser.username = '\${USER}'
);"

identity_nickq="DELETE
FROM identity_nick
WHERE identity_nick.identityid in (
    SELECT identityid
    FROM quasseluser, identity
    WHERE quasseluser.userid = identity.userid
    AND quasseluser.username = '\${USER}'
);"

identityq="DELETE
FROM identity
WHERE identity.userid in (
    SELECT userid
    FROM quasseluser
    WHERE quasseluser.username = '\${USER}'
);"

networkq="DELETE
FROM network
WHERE network.userid in (
    SELECT userid
    FROM quasseluser
    WHERE quasseluser.username = '\${USER}'
);"

usersettingq="DELETE
FROM user_setting
WHERE user_setting.userid in (
    SELECT userid
    FROM quasseluser
    WHERE quasseluser.username = '\${USER}'
);"

quasseluserq="DELETE
FROM quasseluser
WHERE quasseluser.username = '\${USER}'
;"


exeq "\${backlogq}"
exeq "\${bufferq}"
exeq "\${ircserverq}"
exeq "\${identity_nickq}"
exeq "\${identityq}"
exeq "\${networkq}"
exeq "\${usersettingq}"
exeq "\${quasseluserq}"
EOF
```

## Backup

### Pre requirements

Install rclone:  
```shell
curl https://rclone.org/install.sh | sudo bash
```

Configure rclone:  
```shell
sudo -H rclone config
```

Config log:  
```
2018/08/28 14:23:13 NOTICE: Config file "/root/.config/rclone/rclone.conf" not found - using defaults
No remotes found - make a new one
n) New remote
s) Set configuration password
q) Quit config
n/s/q> n
name> gteamdrive_quassel
Type of storage to configure.
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
13 / Local Disk
   \ "local"
14 / Mega
   \ "mega"
15 / Microsoft Azure Blob Storage
   \ "azureblob"
16 / Microsoft OneDrive
   \ "onedrive"
17 / OpenDrive
   \ "opendrive"
18 / Openstack Swift (Rackspace Cloud Files, Memset Memstore, OVH)
   \ "swift"
19 / Pcloud
   \ "pcloud"
20 / QingCloud Object Storage
   \ "qingstor"
21 / SSH/SFTP Connection
   \ "sftp"
22 / Webdav
   \ "webdav"
23 / Yandex Disk
   \ "yandex"
24 / http Connection
   \ "http"
Storage> 11
Google Application Client Id - leave blank normally.
client_id> ***
Google Application Client Secret - leave blank normally.
client_secret> ***
Scope that rclone should use when requesting access from drive.
Choose a number from below, or type in your own value
 1 / Full access all files, excluding Application Data Folder.
   \ "drive"
 2 / Read-only access to file metadata and file contents.
   \ "drive.readonly"
   / Access to files created by rclone only.
 3 | These are visible in the drive website.
   | File authorization is revoked when the user deauthorizes the app.
   \ "drive.file"
   / Allows read and write access to the Application Data folder.
 4 | This is not visible in the drive website.
   \ "drive.appfolder"
   / Allows read-only access to file metadata but
 5 | does not allow any access to read or download file content.
   \ "drive.metadata.readonly"
scope> 1
ID of the root folder - leave blank normally.  Fill in to access "Computers" folders. (see docs).
root_folder_id>
Service Account Credentials JSON file path  - leave blank normally.
Needed only if you want use SA instead of interactive login.
service_account_file>
Remote config
Use auto config?
 * Say Y if not sure
 * Say N if you are working on a remote or headless machine or Y didn't work
y) Yes
n) No
y/n> n
If your browser doesn't open automatically go to the following link: https://accounts.google.com/o/oauth2/auth?access_type=offline&client_id=***
Log in and authorize rclone for access
Enter verification code> ***
Configure this as a team drive?
y) Yes
n) No
y/n> y
Fetching team drive list...
Choose a number from below, or type in your own value
 1 / ***
   \ "***"
 2 / ***
   \ "***"
 3 / quassel
   \ "***"
Enter a Team Drive ID> 3
--------------------
[gteamdrive_quassel]
type = drive
client_id = ***
client_secret = ***
scope = drive
root_folder_id =
service_account_file =
token = ***
team_drive = <drive id from quassel>
--------------------
y) Yes this is OK
e) Edit this remote
d) Delete this remote
y/e/d> y
Current remotes:

Name                 Type
====                 ====
gteamdrive_quassel   drive

e) Edit existing remote
n) New remote
d) Delete remote
r) Rename remote
c) Copy remote
s) Set configuration password
q) Quit config
e/n/d/r/c/s/q> n
name> quassel_crypt
Type of storage to configure.
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
13 / Local Disk
   \ "local"
14 / Mega
   \ "mega"
15 / Microsoft Azure Blob Storage
   \ "azureblob"
16 / Microsoft OneDrive
   \ "onedrive"
17 / OpenDrive
   \ "opendrive"
18 / Openstack Swift (Rackspace Cloud Files, Memset Memstore, OVH)
   \ "swift"
19 / Pcloud
   \ "pcloud"
20 / QingCloud Object Storage
   \ "qingstor"
21 / SSH/SFTP Connection
   \ "sftp"
22 / Webdav
   \ "webdav"
23 / Yandex Disk
   \ "yandex"
24 / http Connection
   \ "http"
Storage> 8
Remote to encrypt/decrypt.
Normally should contain a ':' and a path, eg "myremote:path/to/dir",
"myremote:bucket" or maybe "myremote:" (not recommended).
remote> gteamdrive_quassel:/
How to encrypt the filenames.
Choose a number from below, or type in your own value
 1 / Don't encrypt the file names.  Adds a ".bin" extension only.
   \ "off"
 2 / Encrypt the filenames see the docs for the details.
   \ "standard"
 3 / Very simple filename obfuscation.
   \ "obfuscate"
filename_encryption> 2
Option to either encrypt directory names or leave them intact.
Choose a number from below, or type in your own value
 1 / Encrypt directory names.
   \ "true"
 2 / Don't encrypt directory names, leave them intact.
   \ "false"
directory_name_encryption> 1
Password or pass phrase for encryption.
y) Yes type in my own password
g) Generate random password
y/g> g
Password strength in bits.
64 is just about memorable
128 is secure
1024 is the maximum
Bits> 1024
Your password is: ***
Use this password?
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
Use this password?
y) Yes
n) No
y/n> y
Remote config
--------------------
[quassel_crypt]
type = crypt
remote = gteamdrive_quassel:/
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
gteamdrive_quassel   drive
quassel_crypt        crypt

e) Edit existing remote
n) New remote
d) Delete remote
r) Rename remote
c) Copy remote
s) Set configuration password
q) Quit config
e/n/d/r/c/s/q> q
```

### Backup script

```shell
cat << EOF | sudo tee /usr/local/sbin/quassel_backup && \
sudo chmod 500 /usr/local/sbin/quassel_backup
#!/bin/bash

user='quasselcore'
database='/var/lib/quassel/quassel-storage.sqlite'
backup_target='/tmp'
backup_name='quassel'
date=\$(date +"%Y-%m-%d_%H:%m")

# Backup database
sudo -u \${user} sqlite3 \${database} ".backup \${backup_target}/\${backup_name}_\${date}.sqlite"

# Upload database to google
rclone copy \${backup_target}/\${backup_name}_\${date}.sqlite quassel_crypt:

# remove local backup file
rm \${backup_target}/\${backup_name}_\${date}.sqlite
EOF
```

### Cronjob

```shell
sudo crontab -e
```

Add at the end of the file:  
```
@hourly /usr/local/sbin/quassel_backup
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

    # Allow quassel incoming
    iptables -t filter -A INPUT -i eth0 -p tcp --dport 4242 -m state --state NEW,ESTABLISHED,RELATED -j ACCEPT

    # Allow ESTABLISHED and RELATED connection (important for outgoing connections!)
    iptables -t filter -A INPUT -i eth0 -m state --state ESTABLISHED,RELATED -j ACCEPT

    # Policy DROP INPUT on
    iptables -P INPUT DROP;

    # Policy ACCEPT OUTPUT
    iptables -P OUTPUT ACCEPT
    ```

Set up needed iptables rules:  
```shell
sudo iptables -A OUTPUT -o lo -j ACCEPT; \
sudo iptables -A INPUT -i lo -j ACCEPT; \
sudo iptables -t filter -A INPUT -i eth0 -p tcp --dport 22 -m state --state NEW,ESTABLISHED,RELATED -j ACCEPT; \
sudo iptables -t filter -A INPUT -i eth0 -p tcp --dport 4242 -m state --state NEW,ESTABLISHED,RELATED -j ACCEPT; \
sudo iptables -t filter -A INPUT -i eth0 -m state --state ESTABLISHED,RELATED -j ACCEPT; \
sudo iptables -t filter -A INPUT -i eth0 -p icmp -j ACCEPT; \
sudo iptables -P INPUT DROP; \
sudo iptables -P OUTPUT ACCEPT
```
Set up needed ip6tables rules:  
```shell
sudo ip6tables -A OUTPUT -o lo -j ACCEPT; \
sudo ip6tables -A INPUT -i lo -j ACCEPT; \
sudo ip6tables -t filter -A INPUT -i eth0 -p tcp --dport 22 -m state --state NEW,ESTABLISHED,RELATED -j ACCEPT; \
sudo ip6tables -t filter -A INPUT -i eth0 -p tcp --dport 4242 -m state --state NEW,ESTABLISHED,RELATED -j ACCEPT; \
sudo ip6tables -t filter -A INPUT -i eth0 -m state --state ESTABLISHED,RELATED -j ACCEPT; \
sudo ip6tables -t filter -A INPUT -i eth0 -p ipv6-icmp -j ACCEPT; \
sudo ip6tables -P INPUT DROP; \
sudo ip6tables -P OUTPUT ACCEPT
```

Persist iptables rules:  
```shell
sudo apt install -y iptables-persistent && \
sudo netfilter-persistent save && \
sudo netfilter-persistent reload
```