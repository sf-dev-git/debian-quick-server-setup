# Debian quick Server setup

## Description

This repository serves as a simplified solution for setting up a Debian server with essential security configurations, primarily for personal use. For further explanation, please refer to the provided [link](https://github.com/imthenachoman/How-To-Secure-A-Linux-Server?tab=readme-ov-file). Note: Due to the nature of the project, typographical/grammatic errors may be present.

## Change root password

```
apt-get update
apt-get upgrade
passwd
```

## Create user

```
adduser %USER%
```

## Add user to sudo

```
usermod -aG sudo %USER%
```

## Create sshuser group

```
addgroup sshusers
```

## Add new user to sshuser group

```
usermod -aG sshusers %USER%
```

## Configure SSH key login with passphrase

- `mkdir /home/%USER%/.ssh/`
- copy paste pubkey `nano /home/%USER%/.ssh/authorized_keys` 
- edit sshd_config file 
	- `AuthorizedKeysFile      /home/%USER%/.ssh/authorized_keys`
	- `AllowGroups sshusers`
	- `ClientAliveCountMax 0`
	- `ClientAliveInterval 300`
	- `LoginGraceTime 30`
	- `MaxAuthTries 2`
	- `MaxSessions 2`
	- `PasswordAuthentication no`
	- `Port 1-65,535`

if ssh key login error occurs 
- `chmod 700 ~/.ssh`
- `chmod 600 ~/.ssh/authorized_keys`
- `chown $USER:$USER ~/.ssh -R
- `sudo service ssh restart`

if ur still able to login with password despite the changes in `sshd_conf`
- edit `sudo nano /etc/ssh/sshd_config.d/50-cloud-init.conf` set `PasswordAuthentication no`

## Remove Short Diffie-Hellman Keys

- `sudo cp --archive /etc/ssh/moduli /etc/ssh/moduli-COPY-$(date +"%Y%m%d%H%M%S")`
- `sudo awk '$5 >= 3071' /etc/ssh/moduli | sudo tee /etc/ssh/moduli.tmp`
- `sudo mv /etc/ssh/moduli.tmp /etc/ssh/moduli`

## Limit Who Can Use su

- `sudo groupadd suusers`
- `sudo usermod -aG suusers user1`
- `sudo dpkg-statoverride --update --add root suusers 4750 /bin/su`

##  Automatic Security Updates

- `sudo apt install unattended-upgrades`
- Comment out everything other than the security updates ` sudo nano /etc/apt/apt.conf.d/50unattended-upgrades`
- edit `sudo nano /etc/apt/apt.conf.d/20auto-upgrades`
```
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Download-Upgradeable-Packages "1";
APT::Periodic::AutocleanInterval "7";
APT::Periodic::Unattended-Upgrade "1";
```

## Firewalll

- `sudo apt install ufw`
- `sudo ufw allow %SSH-PORT%`
- `sudo ufw enable`
- `sudo ufw status`

## Fail2ban

- `sudo apt install fail2ban`
- `sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local`
```
cat << EOF | sudo tee /etc/fail2ban/jail.d/ssh.local
[sshd]
enabled = true
banaction = ufw
port = ssh
filter = sshd
logpath = %(sshd_log)s
maxretry = 5
EOF
```

- on debian fail2ban might run into error for that install rsyslog `sudo apt-get install rsyslog`
- `sudo fail2ban-client start`
- `sudo fail2ban-client reload`
- `sudo fail2ban-client add sshd`
- `sudo fail2ban-client status`
- `sudo fail2ban-client status sshd`
- to unban an ip `fail2ban-client set [jail] unbanip [IP]`
