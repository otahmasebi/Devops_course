# Install Linux
## install linux and configure partition tables


| # partition  | size |type| lvg | lvl |mount point|filesystem|
|-|-|-|-|-|-|-|
|partition1 | 1G|primary ESP | | |  /boot/efi | fat32 |
| partition2| 2G| primary| | | /boot | ext4 |
|partition3| 30G | LVM | ubuntu-vg | lv-root |/ | ext4|
|partition3 |30G | LVM | ubuntu-vg | lv-var | /var | ext4|

### update and install updates

```shell
sudo apt update
sudo apt dist-upgrade
```

### install lynis
```shell
sudo apt install lynis
```

audit system with lynis
```shell
lynis audit system
```

### hardening process

#### System and Software Updates

- Unattended Upgrades: Configure unattended-upgrades to automatically install security updates.
```bash
sudo apt-get install unattended-upgrades
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

#### User and Authentication Security

- Disable Root Login: Disable root SSH login if it's enabled.
```bash
sudo nano /etc/ssh/sshd_config
```
add these lines:
```bash
# Use SSH Protocol 2
Protocol 2

# Ignore .rhosts files for authentication
IgnoreRhosts yes

# Disable host-based authentication
HostbasedAuthentication no

# Disallow empty passwords
PermitEmptyPasswords no

# Disable X11 forwarding
X11Forwarding no

# Limit the number of authentication attempts
MaxAuthTries 5

# Set allowed ciphers for encryption
Ciphers aes128-ctr,aes192-ctr,aes256-ctr

# Configure client keepalive settings
ClientAliveInterval 900
ClientAliveCountMax 0

# Use PAM (Pluggable Authentication Modules)
UsePAM yes
# disable password login
PasswordAuthentication no
PubkeyAuthentication yes
# disable other form of authentication
ChallengeResponseAuthentication no
```
- restart `ssh` service
```bash
sudo service ssh restart
```
#### disable unused services
```bash
sudo apt-get --purge remove xinetd nis yp-tools tftpd atftpd tftpd-hpa telnetd rsh-server rsh-redone-server
```
#### file permissions
```bash
sudo chmod 600 /etc/ssh/sshd_config
```

#### Network Security

Allow incoming HTTPS traffic on port 22,80,443.

```bash
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
```
Allow loopback interface
```bash
sudo iptables -A INPUT -i lo -j ACCEPT
sudo iptables -A OUTPUT -o lo -j ACCEPT
```
Allow Established and Related connections
```bash
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
```
Allow outgoing trrafic
```bash
sudo iptables -A OUTPUT -m conntrack --ctstate NEW,ESTABLISHED,RELATED -j ACCEPT
```
Set policy Drop
```bash
sudo iptables -P INPUT DROP
sudo iptables -P FORWARD DROP
sudo iptables -P OUTPUT DROP
```
Make ip-tables persist
```bash
sudo apt install iptables-persistent
sudo iptables-save | sudo tee /etc/iptables/rules.v4
```
#### Security Tools and Configurations
- Use ClamAV for antivirus scanning.
```bash
sudo apt-get install clamav clamav-daemon
sudo freshclam
sudo systemctl start clamav-daemon
```

#### Logging and Auditing
Use `auditd` to monitor critical system files and suspicious activities.
```bash
sudo apt-get install auditd
sudo systemctl enable auditd
```
#### System and Kernel Hardening
```bash
sudo nano /etc/sysctl.conf
```
set these:
```bash
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.all.secure_redirects = 0
```
Apply settings:
```bash
sudo sysctl -p
```
#### disable usb usage
```bash
sudo nano /etc/modprobe.d/blacklist.conf
```
add `blacklist usb_storage` to file
```bash
nano /etc/rc.local
```
add these lines and save and exit
```bash
modprobe -r usb_storage
exit 0
```
#### install SELinux
```bash
sudo systemctl stop apparmor
sudo systemctl disable apparmor
sudo apt-get remove apparmor -y
sudo apt-get install policycoreutils selinux-utils selinux-basics -y
sudo selinux-activate
sudo selinux-config-enforcing
```

#### install fail2ban
```bash
sudo apt-get install fail2ban
```
add these in config file under `/etc/fail2ban/jail.local`
```bash
[DEFAULT]
bantime = 10m
findtime = 1h
maxretry = 5
[sshd]
enabled = true
port=22
```

#### install some packages
```bash
sudo apt-get -y install apt-listchanges
sudo apt-get install -y libpam-tmpdir
```
#### disable non used services
```
sudo nano /etc/modprobe.d/blacklist.conf
```
```
# Disable DCCP protocol
blacklist dccp

# Disable SCTP protocol
blacklist sctp

# Disable RDS protocol
blacklist rds

# Disable TIPC protocol
blacklist tipc
```

```
sudo update-initramfs -u
```