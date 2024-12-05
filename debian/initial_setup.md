# Quick initial security setup of a Linux Debian 12 machine

This tutorial shows how to create and execute a Linux Debian 12 bash script to perform a basic security setup of the machine.

Specifically, the script will:

* Update and upgrade the operating system.
* Create a non-root user, let you choose username/pwd.
* Disable root login and password authentication. Next time you login, you must use the newly created username and the same SSH key used for logging in as root.
* Install unattended upgrades to automatically keep the machine up with the latest security updates.
* Add cron job to automatically trim system logs regularly.
* Install firewall, close all ports except SSH.
* Install fail2ban.

### Steps

After creating a new Debian machine and logging in as *root* using an SSH key, create an *initial.sh* file and make it executable:
```
cd && touch initial.sh && chmod +x initial.sh
```

Open *initial.sh* with your favourite text editor, insert the script contents below:
```
#!/bin/bash

echo '--- UPDATING THE OS'
apt update -y && apt upgrade -y

read -p '--- CREATING NON-ROOT USER, ENTER A USERNAME: ' NEWUSER
read -s -p '--- NOW ENTER A PWD: ' NEWUSERPWD
echo ''
adduser --disabled-password --gecos '' $NEWUSER
echo $NEWUSER:$NEWUSERPWD | sudo chpasswd
adduser $NEWUSER sudo

echo '--- MOVING SSH AUTHORIZED KEY TO NEW USER'
mkdir /home/$NEWUSER/.ssh
touch /home/$NEWUSER/.ssh/authorized_keys
cp .ssh/authorized_keys /home/$NEWUSER/.ssh
chown -R $NEWUSER:$NEWUSER /home/$NEWUSER/.ssh
chmod 0700 /home/$NEWUSER/.ssh
chmod 0600 /home/$NEWUSER/.ssh/authorized_keys
rm -rf /root/.ssh/*

echo '--- DISABLING PASSWORD AUTHENTICATION AND ROOT LOGIN'
sed -i 's/PermitRootLogin yes/PermitRootLogin no/g' /etc/ssh/sshd_config
sed -i 's/#PermitRootLogin no/PermitRootLogin no/g' /etc/ssh/sshd_config
sed -i 's/# PermitRootLogin no/PermitRootLogin no/g' /etc/ssh/sshd_config
sed -i 's/PasswordAuthentication yes/PasswordAuthentication no/g' /etc/ssh/sshd_config
sed -i 's/#PasswordAuthentication no/PasswordAuthentication no/g' /etc/ssh/sshd_config
sed -i 's/# PasswordAuthentication no/PasswordAuthentication no/g' /etc/ssh/sshd_config
echo 'ClientAliveInterval 120' >> /etc/ssh/sshd_config
echo 'ClientAliveCountMax 10' >> /etc/ssh/sshd_config
service ssh restart

echo '--- INSTALLING UNATTENDED UPGRADES'
apt install unattended-upgrades -y

echo '-- ADDING DAILY CRON JOB TO REDUCE SYSTEM LOG SIZE'
echo '#system' >> /etc/crontab
echo '0 0 * * *   root   journalctl --vacuum-size=200M >> /tmp/journalctl_daily_vacuum.log 2>&1' >> /etc/crontab

echo '--- INSTALLING FIREWALL, ENABLING SSH PORT ONLY'
apt install ufw -y
ufw default deny
ufw allow ssh
ufw --force enable
ufw status

echo '--- INSTALLING FAIL2BAN'
apt install fail2ban -y
echo -e '[sshd]\nbackend=systemd\nenabled=true' | tee /etc/fail2ban/jail.local
systemctl enable fail2ban --now

echo '--- INSTALLING UTILITIES'
apt install lsof zip unzip curl tree -y
```

Save and close the file then execute it:
```
./initial.sh
```
