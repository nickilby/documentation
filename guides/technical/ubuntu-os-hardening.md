# Ubuntu OS Hardening Guide

**Version:** 1.0  
**Last Updated:** 2025  
**Next Review Date:** 2026-01-01  
**Owner:** IT Security Team

## Purpose

This guide provides step-by-step procedures for hardening Ubuntu operating systems to meet organizational
security requirements. This guide covers OS-level security configurations, system hardening, and security
best practices.

## Scope

This guide applies to all Ubuntu server and desktop installations within the organization. It covers both
initial system hardening and ongoing security maintenance.

## Prerequisites

- Root or sudo access to the Ubuntu system
- Basic knowledge of Linux command line
- Access to system documentation and network configuration

## Related Policies

- [Security Policy](../policies/security-policy.md)
- [Password Policy](../policies/password-policy.md)
- [Network Security Policy](../policies/network-security-policy.md)
- [Access Control Policy](../policies/access-control-policy.md)

---

## 1. Initial System Setup and Updates

### 1.1 System Update

Before beginning any hardening procedures, ensure the system is fully updated:

```bash
# Update package lists
sudo apt update

# Upgrade all packages to latest versions
sudo apt upgrade -y

# Remove unnecessary packages
sudo apt autoremove -y

# Clean package cache
sudo apt autoclean
```

### 1.2 Enable Automatic Security Updates

Configure automatic installation of security updates:

```bash
# Install unattended-upgrades package
sudo apt install unattended-upgrades -y

# Enable automatic updates
sudo dpkg-reconfigure -plow unattended-upgrades
```

Configure automatic updates in `/etc/apt/apt.conf.d/50unattended-upgrades`:

```bash
# Edit configuration
sudo nano /etc/apt/apt.conf.d/50unattended-upgrades
```

Ensure the following settings are configured:

```ini
Unattended-Upgrade::Automatic-Reboot "false";
Unattended-Upgrade::Automatic-Reboot-Time "02:00";
Unattended-Upgrade::Remove-Unused-Kernel-Packages "true";
Unattended-Upgrade::Remove-Unused-Dependencies "true";
```

### 1.3 Configure Update Notifications

Enable email notifications for updates (if mail server is configured):

```bash
sudo nano /etc/apt/apt.conf.d/50unattended-upgrades
```

Add:

```ini
Unattended-Upgrade::Mail "admin@example.com";
Unattended-Upgrade::MailReport "only-on-error";
```

---

## 2. User Account Management and Sudo Configuration

### 2.1 Create Administrative User

Create a non-root administrative user:

```bash
# Create new user
sudo adduser adminuser

# Add user to sudo group
sudo usermod -aG sudo adminuser

# Verify sudo access
su - adminuser
sudo whoami
```

### 2.2 Secure Root Account

Disable direct root login via SSH (will be configured in SSH section):

```bash
# Lock root account password (prevents password login)
sudo passwd -l root

# Verify root account is locked
sudo passwd -S root
```

### 2.3 Configure Sudo Access

Edit sudo configuration:

```bash
sudo visudo
```

Add the following security settings:

```ini
# Require password for sudo (default, but ensure it's set)
Defaults        passwd_timeout=0

# Log all sudo commands
Defaults        logfile=/var/log/sudo.log
Defaults        log_input,log_output

# Prevent sudo caching
Defaults        timestamp_timeout=0

# Secure path
Defaults        secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"

# Prevent running commands in other users' home directories
Defaults        always_set_home
Defaults        !env_reset
```

### 2.4 Configure Password Policy

Install and configure password quality requirements:

```bash
# Install libpam-pwquality
sudo apt install libpam-pwquality -y
```

Edit `/etc/security/pwquality.conf`:

```bash
sudo nano /etc/security/pwquality.conf
```

Configure password requirements (aligned with [Password Policy](../policies/password-policy.md)):

```ini
minlen = 12
minclass = 3
dcredit = -1
ucredit = -1
lcredit = -1
ocredit = -1
maxrepeat = 2
maxsequence = 3
```

Configure password aging in `/etc/login.defs`:

```bash
sudo nano /etc/login.defs
```

Set:

```ini
PASS_MAX_DAYS   90
PASS_MIN_DAYS   1
PASS_WARN_AGE   14
PASS_MIN_LEN    12
```

### 2.5 Account Lockout Policy

Configure account lockout to prevent brute force attacks:

```bash
# Install fail2ban (covered in detail in section 8)
sudo apt install fail2ban -y
```

Configure PAM to lock accounts after failed attempts:

```bash
sudo nano /etc/pam.d/common-auth
```

Add before the existing `auth` lines:

```pam
auth required pam_tally2.so deny=5 unlock_time=1800 onerr=fail
```

---

## 3. SSH Hardening

### 3.1 Install and Update SSH

```bash
# Ensure SSH server is installed
sudo apt install openssh-server -y

# Verify SSH is running
sudo systemctl status ssh
```

### 3.2 Configure SSH Server

Edit SSH configuration:

```bash
sudo nano /etc/ssh/sshd_config
```

Apply the following security settings:

```ssh-config
# Disable root login
PermitRootLogin no

# Disable password authentication (use key-based only)
PasswordAuthentication no
PubkeyAuthentication yes

# Change default port (optional but recommended)
Port 2222

# Limit authentication attempts
MaxAuthTries 3
MaxSessions 2

# Disable empty passwords
PermitEmptyPasswords no

# Disable X11 forwarding (unless needed)
X11Forwarding no

# Disable TCP forwarding (unless needed)
AllowTcpForwarding no

# Set idle timeout
ClientAliveInterval 300
ClientAliveCountMax 2

# Disable DNS lookup (improves connection speed)
UseDNS no

# Restrict users who can login (adjust as needed)
AllowUsers adminuser

# Disable unused authentication methods
ChallengeResponseAuthentication no
UsePAM yes

# Strong encryption algorithms only
KexAlgorithms curve25519-sha256@libssh.org,diffie-hellman-group-exchange-sha256
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com,aes256-ctr,aes192-ctr,aes128-ctr
MACs hmac-sha2-256-etm@openssh.com,hmac-sha2-512-etm@openssh.com,hmac-sha2-256,hmac-sha2-512

# Logging
SyslogFacility AUTH
LogLevel VERBOSE
```

### 3.3 Generate SSH Keys

On the client machine, generate SSH key pair:

```bash
# Generate ED25519 key (recommended)
ssh-keygen -t ed25519 -C "your_email@example.com"

# Or generate RSA key (4096 bits minimum)
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

Copy public key to server:

```bash
# From client machine
ssh-copy-id -p 2222 adminuser@server-ip

# Or manually copy
cat ~/.ssh/id_ed25519.pub | ssh -p 2222 adminuser@server-ip "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

### 3.4 Secure SSH Key Permissions

On the server, set correct permissions:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
chmod 600 ~/.ssh/id_* 2>/dev/null
chmod 644 ~/.ssh/*.pub 2>/dev/null
```

### 3.5 Restart SSH Service

```bash
# Test configuration
sudo sshd -t

# If test passes, restart SSH
sudo systemctl restart sshd

# Verify SSH is running
sudo systemctl status sshd
```

**Important:** Before disconnecting, test SSH connection from another terminal to ensure you can still connect.

---

## 4. Firewall Configuration (UFW)

### 4.1 Install and Enable UFW

```bash
# Install UFW
sudo apt install ufw -y

# Enable UFW
sudo ufw enable

# Check status
sudo ufw status verbose
```

### 4.2 Configure Default Policies

```bash
# Deny all incoming by default
sudo ufw default deny incoming

# Allow all outgoing by default
sudo ufw default allow outgoing
```

### 4.3 Configure Firewall Rules

```bash
# Allow SSH (use your custom port if changed)
sudo ufw allow 2222/tcp comment 'SSH'

# Allow HTTP/HTTPS (if web server)
sudo ufw allow 80/tcp comment 'HTTP'
sudo ufw allow 443/tcp comment 'HTTPS'

# Allow specific IP addresses (example)
sudo ufw allow from 192.168.1.0/24 to any port 2222

# Deny specific IP addresses
sudo ufw deny from 10.0.0.50
```

### 4.4 Enable Logging

```bash
# Enable UFW logging
sudo ufw logging on

# Set log level (low, medium, high, full)
sudo ufw logging medium
```

### 4.5 Review and Test Rules

```bash
# List all rules
sudo ufw status numbered

# Delete a rule by number
sudo ufw delete [number]

# Test firewall (from another machine)
# Attempt to connect to blocked ports
```

---

## 5. System Service Hardening

### 5.1 Identify Running Services

```bash
# List all running services
sudo systemctl list-units --type=service --state=running

# List enabled services
sudo systemctl list-unit-files --type=service --state=enabled
```

### 5.2 Disable Unnecessary Services

Review and disable services that are not required:

```bash
# Example: Disable unnecessary services
sudo systemctl disable bluetooth
sudo systemctl disable avahi-daemon
sudo systemctl disable cups
sudo systemctl disable isc-dhcp-server
sudo systemctl disable isc-dhcp-server6

# Stop services immediately
sudo systemctl stop bluetooth
sudo systemctl stop avahi-daemon
```

### 5.3 Remove Unnecessary Packages

```bash
# List installed packages
dpkg -l

# Remove unnecessary packages
sudo apt remove --purge [package-name] -y

# Clean up
sudo apt autoremove -y
sudo apt autoclean
```

### 5.4 Secure Service Configuration

For services that must remain enabled, ensure secure configuration:

```bash
# Example: Secure NTP
sudo apt install ntp -y
sudo nano /etc/ntp.conf
# Configure trusted NTP servers only
```

---

## 6. Kernel Parameter Tuning for Security

### 6.1 Configure sysctl Parameters

Edit sysctl configuration:

```bash
sudo nano /etc/sysctl.conf
```

Add the following security parameters:

```ini
# IP Spoofing protection
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.rp_filter = 1

# Ignore ICMP redirects
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0
net.ipv6.conf.all.accept_redirects = 0
net.ipv6.conf.default.accept_redirects = 0

# Ignore send redirects
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.send_redirects = 0

# Disable source packet routing
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.conf.default.accept_source_route = 0
net.ipv6.conf.all.accept_source_route = 0
net.ipv6.conf.default.accept_source_route = 0

# Log Martians
net.ipv4.conf.all.log_martians = 1
net.ipv4.conf.default.log_martians = 1

# Ignore ICMP ping requests
net.ipv4.icmp_echo_ignore_all = 1

# Ignore directed pings
net.ipv4.icmp_echo_ignore_broadcasts = 1

# TCP SYN Cookie Protection
net.ipv4.tcp_syncookies = 1

# Disable IP forwarding
net.ipv4.ip_forward = 0
net.ipv6.conf.all.forwarding = 0

# Disable source routing
net.ipv4.conf.all.accept_source_route = 0
net.ipv6.conf.all.accept_source_route = 0

# Ignore send redirects
net.ipv4.conf.all.send_redirects = 0
net.ipv6.conf.all.send_redirects = 0

# Disable ICMP redirect acceptance
net.ipv4.conf.all.accept_redirects = 0
net.ipv6.conf.all.accept_redirects = 0

# Disable secure redirects
net.ipv4.conf.all.secure_redirects = 0

# Log suspicious packets
net.ipv4.conf.all.log_martians = 1

# Ignore ping broadcasts
net.ipv4.icmp_echo_ignore_broadcasts = 1

# Ignore bad ICMP errors
net.ipv4.icmp_ignore_bogus_error_responses = 1

# Reverse path filtering
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.rp_filter = 1

# Disable IPv6 if not used
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
```

### 6.2 Apply sysctl Changes

```bash
# Reload sysctl configuration
sudo sysctl -p

# Verify settings
sudo sysctl -a | grep -E "rp_filter|accept_redirects|send_redirects"
```

### 6.3 Additional Kernel Hardening

Install and configure kernel hardening tools:

```bash
# Install grsecurity-like hardening (if available)
sudo apt install linux-hardened -y
```

---

## 7. File System Security

### 7.1 Configure Mount Options

Edit `/etc/fstab`:

```bash
sudo nano /etc/fstab
```

Ensure secure mount options:

```fstab
# Example entry
UUID=xxxx-xxxx-xxxx / ext4 defaults,noexec,nosuid,nodev 0 1
/tmp /tmp tmpfs defaults,noexec,nosuid,nodev,size=2G 0 0
/var/tmp /var/tmp tmpfs defaults,noexec,nosuid,nodev,size=2G 0 0
```

**Note:** Be careful with `noexec` on root filesystem as it may break some applications.

### 7.2 Secure Directory Permissions

```bash
# Set restrictive permissions on sensitive directories
sudo chmod 700 /root
sudo chmod 700 /home/*

# Secure configuration files
sudo chmod 600 /etc/shadow
sudo chmod 644 /etc/passwd
sudo chmod 644 /etc/group
sudo chmod 600 /etc/ssh/sshd_config
sudo chmod 600 /etc/sudoers
```

### 7.3 Configure Sticky Bit on World-Writable Directories

```bash
# Set sticky bit on /tmp and /var/tmp
sudo chmod +t /tmp
sudo chmod +t /var/tmp
```

### 7.4 Disable Core Dumps

```bash
# Edit limits configuration
sudo nano /etc/security/limits.conf
```

Add:

```ini
* soft core 0
* hard core 0
```

Edit sysctl:

```bash
sudo nano /etc/sysctl.conf
```

Add:

```ini
fs.suid_dumpable = 0
kernel.core_uses_pid = 1
```

---

## 8. Logging and Audit Configuration

### 8.1 Configure rsyslog

Edit rsyslog configuration:

```bash
sudo nano /etc/rsyslog.conf
```

Ensure logging is configured:

```ini
# Log all authentication attempts
auth,authpriv.*                 /var/log/auth.log
*.*;auth,authpriv.none          -/var/log/syslog

# Log kernel messages
kern.*                          -/var/log/kern.log

# Log mail messages
mail.*                          -/var/log/mail.log

# Log cron jobs
cron.*                          /var/log/cron.log
```

### 8.2 Configure Log Rotation

Edit logrotate configuration:

```bash
sudo nano /etc/logrotate.conf
```

Ensure appropriate retention:

```ini
# Rotate logs weekly
weekly

# Keep 4 weeks of logs
rotate 4

# Compress old logs
compress

# Don't rotate empty logs
notifempty
```

### 8.3 Install and Configure fail2ban

```bash
# Install fail2ban
sudo apt install fail2ban -y

# Create local configuration
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local

# Edit configuration
sudo nano /etc/fail2ban/jail.local
```

Configure SSH protection:

```ini
[sshd]
enabled = true
port = 2222
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 3600
findtime = 600
```

Configure additional services as needed.

### 8.4 Enable Auditd (Advanced)

For enhanced auditing:

```bash
# Install auditd
sudo apt install auditd audispd-plugins -y

# Start and enable auditd
sudo systemctl enable auditd
sudo systemctl start auditd

# Configure audit rules
sudo nano /etc/audit/rules.d/audit.rules
```

Add rules:

```audit
# Monitor file access
-w /etc/passwd -p wa -k passwd_changes
-w /etc/group -p wa -k group_changes
-w /etc/shadow -p wa -k shadow_changes
-w /etc/sudoers -p wa -k sudoers_changes

# Monitor system calls
-a always,exit -F arch=b64 -S adjtimex -S settimeofday -k time_change
-a always,exit -F arch=b64 -S mount -S umount2 -k mount
```

---

## 9. Intrusion Detection Basics

### 9.1 Install AIDE (Advanced Intrusion Detection Environment)

```bash
# Install AIDE
sudo apt install aide -y

# Initialize AIDE database
sudo aideinit

# Create initial database
sudo mv /var/lib/aide/aide.db.new /var/lib/aide/aide.db

# Run initial check
sudo aide --check
```

### 9.2 Configure AIDE Checks

Edit AIDE configuration:

```bash
sudo nano /etc/aide/aide.conf
```

Configure what to monitor based on your needs.

### 9.3 Schedule Regular AIDE Checks

Add to crontab:

```bash
sudo crontab -e
```

Add:

```cron
# Run AIDE check daily at 2 AM
0 2 * * * /usr/bin/aide --check
```

### 9.4 Install and Configure rkhunter

```bash
# Install rkhunter
sudo apt install rkhunter -y

# Update database
sudo rkhunter --update

# Run check
sudo rkhunter --check

# Configure
sudo nano /etc/rkhunter.conf
```

---

## 10. Package Management Security

### 10.1 Configure APT Sources

Ensure only trusted repositories are used:

```bash
# Review sources
cat /etc/apt/sources.list
ls /etc/apt/sources.list.d/

# Remove untrusted repositories
sudo rm /etc/apt/sources.list.d/untrusted-repo.list
```

### 10.2 Verify Package Integrity

```bash
# Update package lists
sudo apt update

# Verify package signatures
sudo apt-get --allow-unauthenticated upgrade 2>&1 | grep -i "NO_PUBKEY"
```

### 10.3 Use GPG Keys for Repositories

```bash
# Add repository GPG key
wget -qO - https://example.com/repo/key.gpg | sudo apt-key add -

# Or use keyring file
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys [KEY_ID]
```

### 10.4 Audit Installed Packages

```bash
# List all installed packages
dpkg -l > installed-packages.txt

# Check for packages with known vulnerabilities
sudo apt list --upgradable

# Review package changelogs
apt changelog [package-name]
```

---

## 11. Additional Security Hardening

### 11.1 Disable USB Storage (if not needed)

```bash
# Blacklist USB storage module
echo "blacklist usb-storage" | sudo tee -a /etc/modprobe.d/blacklist.conf
```

### 11.2 Configure AppArmor

```bash
# Check AppArmor status
sudo apparmor_status

# Enable AppArmor
sudo systemctl enable apparmor
sudo systemctl start apparmor

# Put profiles in enforce mode
sudo aa-enforce /etc/apparmor.d/*
```

### 11.3 Install and Configure ClamAV

```bash
# Install ClamAV
sudo apt install clamav clamav-daemon -y

# Update virus definitions
sudo freshclam

# Run scan
sudo clamscan -r /home
```

### 11.4 Configure Time Synchronization

```bash
# Install and configure chrony
sudo apt install chrony -y

# Configure NTP servers
sudo nano /etc/chrony/chrony.conf

# Verify synchronization
chronyc tracking
```

---

## 12. Security Checklist

Use this checklist to verify hardening completion:

- [ ] System fully updated
- [ ] Automatic security updates enabled
- [ ] Non-root administrative user created
- [ ] Root account locked
- [ ] SSH hardened (key-based auth, root disabled)
- [ ] Firewall (UFW) configured and enabled
- [ ] Unnecessary services disabled
- [ ] Kernel parameters secured
- [ ] File system permissions secured
- [ ] Logging configured
- [ ] fail2ban installed and configured
- [ ] Password policy enforced
- [ ] Intrusion detection (AIDE/rkhunter) configured
- [ ] Package management secured
- [ ] Time synchronization configured

---

## 13. Maintenance and Ongoing Security

### 13.1 Regular Updates

```bash
# Weekly security updates
sudo apt update && sudo apt upgrade -y

# Review security advisories
sudo apt list --upgradable
```

### 13.2 Security Monitoring

- Review logs regularly: `/var/log/auth.log`, `/var/log/syslog`
- Monitor fail2ban status: `sudo fail2ban-client status`
- Check for failed login attempts
- Review sudo logs: `/var/log/sudo.log`

### 13.3 Vulnerability Scanning

```bash
# Install and run Lynis
sudo apt install lynis -y
sudo lynis audit system

# Review OpenVAS or Nessus scan results
```

### 13.4 Regular Audits

- Monthly review of user accounts
- Quarterly review of firewall rules
- Quarterly review of installed packages
- Annual security assessment

---

## Troubleshooting

### Common Issues

**SSH Connection Lost After Hardening:**

- Verify SSH configuration: `sudo sshd -t`
- Check firewall rules: `sudo ufw status`
- Test from another terminal before disconnecting

**Services Not Starting:**

- Check service status: `sudo systemctl status [service]`
- Review logs: `sudo journalctl -u [service]`
- Verify file permissions

**Package Installation Fails:**

- Check repository configuration
- Verify GPG keys
- Review network connectivity

---

## References

- [Ubuntu Security Documentation](https://ubuntu.com/security)
- [CIS Ubuntu Benchmark](https://www.cisecurity.org/benchmark/ubuntu_linux)
- [SSH Hardening Guide](https://stribika.github.io/2015/01/04/secure-secure-shell.html)

---

## Document Control

**Version History:**

- 1.0 (2025) - Initial release

**Review Schedule:**

This document should be reviewed annually or when significant security changes occur.
