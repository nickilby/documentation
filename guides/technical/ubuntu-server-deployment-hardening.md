# Ubuntu Server Deployment Hardening Guide

**Version:** 1.0  
**Last Updated:** 2025  
**Next Review Date:** 2026-01-01  
**Owner:** IT Security Team

## Purpose

This guide provides best practices and procedures for securely deploying Ubuntu servers in production environments, including network architecture, bastion host configuration, and secure deployment patterns.

## Scope

This guide applies to all Ubuntu server deployments within the organization, including cloud, on-premises, and hybrid environments. It covers pre-deployment planning, network architecture, bastion host setup, and secure deployment practices.

## Prerequisites

- Understanding of network architecture and security
- Access to network infrastructure
- Administrative access to servers
- Familiarity with SSH and network protocols
- Completion of [Ubuntu OS Hardening Guide](./ubuntu-os-hardening.md)

## Related Policies

- [Security Policy](../policies/security-policy.md)
- [Network Security Policy](../policies/network-security-policy.md)
- [Remote Access Policy](../policies/remote-access-policy.md)
- [Access Control Policy](../policies/access-control-policy.md)

---

## 1. Pre-Deployment Planning and Security Requirements

### 1.1 Security Requirements Assessment

Before deployment, document security requirements:

- **Data Classification:** Identify data sensitivity levels
- **Compliance Requirements:** GDPR, HIPAA, PCI-DSS, etc.
- **Access Requirements:** Who needs access and from where
- **Network Requirements:** Network segmentation needs
- **Monitoring Requirements:** Logging and alerting needs
- **Backup Requirements:** Backup and recovery needs

### 1.2 Risk Assessment

Conduct risk assessment for each deployment:

```bash
# Document risk assessment
cat > deployment-risk-assessment.txt << EOF
Server: [server-name]
Purpose: [server-purpose]
Data Classification: [classification-level]
Network Zone: [zone-name]
Access Requirements: [access-details]
Risk Level: [low/medium/high]
Mitigation: [mitigation-strategies]
EOF
```

### 1.3 Deployment Checklist

Create deployment checklist:

- [ ] Security requirements documented
- [ ] Network architecture designed
- [ ] Firewall rules defined
- [ ] Access controls configured
- [ ] Monitoring configured
- [ ] Backup strategy defined
- [ ] Documentation completed

---

## 2. Network Architecture and Segmentation

### 2.1 Network Zone Design

Design network architecture with proper segmentation:

```
Internet
  |
  | (Firewall)
  |
DMZ Zone (Public-facing services)
  | - Web servers
  | - Email gateways
  | - DNS servers
  |
  | (Internal Firewall)
  |
Internal Network Zone
  | - Application servers
  | - Database servers
  | - File servers
  |
  | (Management Firewall)
  |
Management Zone
  | - Bastion hosts
  | - Monitoring systems
  | - Management tools
```

### 2.2 Network Segmentation Best Practices

**DMZ Zone:**
- Public-facing services only
- No direct access to internal networks
- Strict firewall rules
- Regular security updates

**Internal Network Zone:**
- Application and data servers
- Restricted external access
- Internal communication only
- Network monitoring

**Management Zone:**
- Administrative access
- Bastion hosts
- Monitoring and logging
- Highly restricted access

### 2.3 VLAN Configuration

Configure VLANs for network segmentation:

```bash
# Example network interface configuration
sudo nano /etc/netplan/01-netcfg.yaml
```

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: no
      addresses:
        - 192.168.1.10/24
      gateway4: 192.168.1.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 8.8.4.4
  vlans:
    vlan100:
      id: 100
      link: eth0
      addresses:
        - 10.0.100.10/24
```

Apply configuration:

```bash
sudo netplan apply
```

---

## 3. Bastion Host/Jump Server Setup

### 3.1 Bastion Host Architecture

Bastion hosts provide secure access to internal servers:

```
Internet
  |
  | (SSH on port 2222)
  |
Bastion Host (Public IP)
  | - Single point of entry
  | - Strong authentication
  | - Logging and monitoring
  |
  | (SSH to internal network)
  |
Internal Servers (Private IPs)
  | - No direct internet access
  | - Accessible only via bastion
```

### 3.2 Deploy Bastion Host

**Step 1: Initial Setup**

```bash
# Follow Ubuntu OS Hardening Guide first
# Then configure as bastion host
```

**Step 2: Configure Network**

```bash
# Configure network interface
sudo nano /etc/netplan/01-netcfg.yaml
```

Ensure bastion has:
- Public IP address (or DMZ IP)
- Access to internal network
- No unnecessary network services

**Step 3: Harden SSH Configuration**

Edit `/etc/ssh/sshd_config`:

```bash
sudo nano /etc/ssh/sshd_config
```

Configure for bastion:

```
# Listen on all interfaces
ListenAddress 0.0.0.0

# Custom port
Port 2222

# Disable root login
PermitRootLogin no

# Key-based authentication only
PasswordAuthentication no
PubkeyAuthentication yes

# Restrict users
AllowUsers bastion-admin

# Port forwarding (for tunneling)
AllowTcpForwarding yes
GatewayPorts no

# X11 forwarding disabled
X11Forwarding no

# Connection limits
MaxAuthTries 3
MaxSessions 10
MaxStartups 10:30:100

# Timeouts
ClientAliveInterval 300
ClientAliveCountMax 2

# Logging
LogLevel VERBOSE
SyslogFacility AUTH
```

### 3.3 Configure SSH Keys for Bastion Access

**On Client Machine:**

```bash
# Generate dedicated key for bastion
ssh-keygen -t ed25519 -f ~/.ssh/bastion_key -C "bastion-access"

# Copy to bastion
ssh-copy-id -i ~/.ssh/bastion_key.pub -p 2222 bastion-admin@bastion-ip
```

**On Bastion Host:**

```bash
# Secure authorized_keys
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys

# Configure SSH client for internal servers
nano ~/.ssh/config
```

### 3.4 Configure SSH Client for Internal Access

On bastion host, configure SSH client:

```bash
nano ~/.ssh/config
```

```ssh-config
# Internal server access via bastion
Host internal-*
    ProxyJump bastion-admin@bastion-ip:2222
    User admin
    IdentityFile ~/.ssh/internal_key
    StrictHostKeyChecking yes
    UserKnownHostsFile ~/.ssh/known_hosts

# Example internal server
Host internal-web1
    HostName 10.0.1.10
    Port 22
```

### 3.5 Configure Firewall for Bastion

```bash
# Allow SSH from specific IPs only (recommended)
sudo ufw allow from [trusted-ip] to any port 2222 proto tcp

# Or allow from specific network
sudo ufw allow from 192.168.1.0/24 to any port 2222 proto tcp

# Deny all other SSH access
sudo ufw deny 22/tcp
sudo ufw deny 2222/tcp

# Allow outbound connections to internal network
sudo ufw allow out to 10.0.0.0/8 port 22 proto tcp
```

### 3.6 Enable Enhanced Logging on Bastion

```bash
# Configure rsyslog for bastion-specific logging
sudo nano /etc/rsyslog.d/50-bastion.conf
```

```
# Log all SSH connections
:msg, contains, "sshd" /var/log/bastion-ssh.log
& stop
```

```bash
# Restart rsyslog
sudo systemctl restart rsyslog
```

---

## 4. SSH Tunneling and Port Forwarding Best Practices

### 4.1 Local Port Forwarding

Access internal services through bastion:

```bash
# Forward local port 8080 to internal web server
ssh -L 8080:10.0.1.10:80 -p 2222 bastion-admin@bastion-ip

# Access via http://localhost:8080
```

### 4.2 Remote Port Forwarding

Expose local service to internal network:

```bash
# Forward remote port 3306 to local MySQL
ssh -R 3306:localhost:3306 -p 2222 bastion-admin@bastion-ip
```

### 4.3 Dynamic Port Forwarding (SOCKS Proxy)

Create SOCKS proxy through bastion:

```bash
# Create SOCKS proxy on local port 1080
ssh -D 1080 -p 2222 bastion-admin@bastion-ip

# Configure browser to use SOCKS proxy
# localhost:1080
```

### 4.4 SSH Config for Tunneling

Configure persistent tunnels:

```bash
nano ~/.ssh/config
```

```ssh-config
Host bastion-tunnel
    HostName bastion-ip
    User bastion-admin
    Port 2222
    LocalForward 8080 10.0.1.10:80
    LocalForward 3306 10.0.1.20:3306
    ServerAliveInterval 60
    ServerAliveCountMax 3
```

### 4.5 Security Considerations for Tunneling

- **Limit Port Forwarding:** Configure `AllowTcpForwarding` in sshd_config
- **Monitor Tunnels:** Log all port forwarding activities
- **Time Limits:** Set connection timeouts
- **Access Control:** Restrict which users can create tunnels

---

## 5. Network Security Configuration

### 5.1 Firewall Rules for Multi-Tier Deployment

**DMZ Firewall Rules:**

```bash
# Allow HTTP/HTTPS from internet
sudo ufw allow from any to any port 80 proto tcp
sudo ufw allow from any to any port 443 proto tcp

# Allow SSH from bastion only
sudo ufw allow from 10.0.0.10 to any port 22 proto tcp

# Deny all other inbound
sudo ufw default deny incoming
```

**Internal Network Firewall Rules:**

```bash
# Allow SSH from bastion
sudo ufw allow from 10.0.0.10 to any port 22 proto tcp

# Allow application communication
sudo ufw allow from 10.0.1.0/24 to any port 8080 proto tcp

# Allow database access from app servers only
sudo ufw allow from 10.0.1.0/24 to any port 3306 proto tcp

# Deny all other inbound
sudo ufw default deny incoming
```

### 5.2 VPN Integration

If using VPN for remote access:

```bash
# Install OpenVPN or WireGuard
sudo apt install openvpn -y

# Configure VPN server
sudo nano /etc/openvpn/server/server.conf
```

Key VPN security settings:

```
# Strong encryption
cipher AES-256-GCM
auth SHA256

# TLS settings
tls-version-min 1.2

# Network configuration
server 10.8.0.0 255.255.255.0

# Push routes for internal network
push "route 10.0.0.0 255.255.0.0"
```

### 5.3 Network Access Control (NAC)

Implement network access control:

```bash
# Install and configure fail2ban for network-level blocking
sudo apt install fail2ban -y

# Configure network-level rules
sudo nano /etc/fail2ban/jail.local
```

### 5.4 Intrusion Detection at Network Level

```bash
# Install Suricata or Snort
sudo apt install suricata -y

# Configure for network monitoring
sudo nano /etc/suricata/suricata.yaml
```

---

## 6. Multi-Tier Deployment Patterns

### 6.1 Three-Tier Architecture

```
Internet
  |
  | (Load Balancer)
  |
Web Tier (DMZ)
  | - Web servers
  | - Load balancers
  |
  | (Application Firewall)
  |
Application Tier (Internal)
  | - Application servers
  | - API servers
  |
  | (Database Firewall)
  |
Data Tier (Internal)
  | - Database servers
  | - File servers
```

### 6.2 Secure Communication Between Tiers

**TLS/SSL Configuration:**

```bash
# Install certificates
sudo apt install certbot -y

# Generate certificates
sudo certbot certonly --standalone -d example.com

# Configure application to use certificates
sudo nano /etc/nginx/nginx.conf
```

**Internal Certificate Authority:**

```bash
# Create internal CA
openssl genrsa -out ca-key.pem 4096
openssl req -new -x509 -days 365 -key ca-key.pem -out ca-cert.pem

# Generate server certificates
openssl genrsa -out server-key.pem 4096
openssl req -new -key server-key.pem -out server.csr
openssl x509 -req -days 365 -in server.csr -CA ca-cert.pem -CAkey ca-key.pem -out server-cert.pem
```

### 6.3 Service-to-Service Authentication

**Use Service Accounts:**

```bash
# Create service account
sudo useradd -r -s /bin/false service-account

# Generate service key
sudo -u service-account ssh-keygen -t ed25519 -f /home/service-account/.ssh/id_ed25519

# Distribute public key to target servers
```

**Use API Keys or Tokens:**

```bash
# Generate secure API token
openssl rand -hex 32

# Store securely
echo "API_TOKEN=$(openssl rand -hex 32)" | sudo tee -a /etc/environment
```

---

## 7. Secure Communication Between Servers

### 7.1 SSH Key Distribution

**Automated Key Distribution:**

```bash
# Use Ansible or similar tool
ansible-playbook -i inventory deploy-ssh-keys.yml
```

**Manual Distribution:**

```bash
# On source server
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519

# Copy to target server
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@target-server
```

### 7.2 VPN Between Servers

Configure site-to-site VPN:

```bash
# Install WireGuard
sudo apt install wireguard -y

# Generate keys
wg genkey | tee privatekey | wg pubkey > publickey

# Configure interface
sudo nano /etc/wireguard/wg0.conf
```

### 7.3 Encrypted Backup Communication

```bash
# Use rsync over SSH
rsync -avz -e "ssh -p 2222" /backup/ user@backup-server:/backups/

# Use encrypted backup tools
sudo apt install duplicity -y
duplicity /source scp://user@backup-server//backups
```

---

## 8. Monitoring and Logging Infrastructure

### 8.1 Centralized Logging

**Install and Configure rsyslog Server:**

```bash
# On log server
sudo apt install rsyslog -y

# Configure to receive logs
sudo nano /etc/rsyslog.conf
```

Enable:

```
module(load="imtcp")
input(type="imtcp" port="514")
```

**Configure Clients:**

```bash
# On each server
sudo nano /etc/rsyslog.conf
```

Add:

```
*.* @@log-server-ip:514
```

### 8.2 Monitoring Setup

**Install Monitoring Agent:**

```bash
# Install Prometheus node exporter
wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz
tar xvfz node_exporter-*.tar.gz
sudo mv node_exporter-*.linux-amd64/node_exporter /usr/local/bin/
```

**Configure Monitoring:**

```bash
# Create systemd service
sudo nano /etc/systemd/system/node_exporter.service
```

```ini
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=nobody
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl enable node_exporter
sudo systemctl start node_exporter
```

### 8.3 Security Event Monitoring

**Install OSSEC or Wazuh:**

```bash
# Install Wazuh agent
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo apt-key add -
echo "deb https://packages.wazuh.com/4.x/apt/ stable main" | sudo tee /etc/apt/sources.list.d/wazuh.list
sudo apt update
sudo apt install wazuh-agent -y
```

**Configure Agent:**

```bash
sudo nano /var/ossec/etc/ossec.conf
```

Set manager IP and configure monitoring.

---

## 9. Backup and Disaster Recovery Considerations

### 9.1 Backup Strategy

**Automated Backups:**

```bash
# Create backup script
sudo nano /usr/local/bin/backup-server.sh
```

```bash
#!/bin/bash
BACKUP_DIR="/backups"
SOURCE_DIR="/data"
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="$BACKUP_DIR/backup_$DATE.tar.gz"

# Create backup
tar -czf $BACKUP_FILE $SOURCE_DIR

# Encrypt backup
gpg --encrypt --recipient backup@example.com $BACKUP_FILE

# Transfer to remote server
scp -p 2222 $BACKUP_FILE.encrypted user@backup-server:/backups/

# Cleanup old backups (keep 30 days)
find $BACKUP_DIR -name "backup_*.tar.gz.encrypted" -mtime +30 -delete
```

**Schedule Backups:**

```bash
# Add to crontab
sudo crontab -e
```

```
# Daily backup at 2 AM
0 2 * * * /usr/local/bin/backup-server.sh
```

### 9.2 Disaster Recovery Planning

**Document Recovery Procedures:**

- Recovery Time Objective (RTO)
- Recovery Point Objective (RPO)
- Backup verification procedures
- Recovery testing schedule

**Create Recovery Scripts:**

```bash
# Document recovery procedures
cat > disaster-recovery-procedures.md << EOF
# Disaster Recovery Procedures

## Server Recovery Steps
1. Provision new server
2. Apply OS hardening
3. Restore from backup
4. Verify services
5. Update DNS/load balancer

## Backup Verification
- Weekly backup restoration test
- Monthly full disaster recovery drill
EOF
```

---

## 10. Compliance and Audit Requirements

### 10.1 Audit Logging

**Enable Comprehensive Auditing:**

```bash
# Install auditd
sudo apt install auditd audispd-plugins -y

# Configure audit rules
sudo nano /etc/audit/rules.d/audit.rules
```

Add comprehensive rules:

```
# Monitor file access
-w /etc/passwd -p wa -k passwd_changes
-w /etc/group -p wa -k group_changes
-w /etc/shadow -p wa -k shadow_changes
-w /etc/sudoers -p wa -k sudoers_changes

# Monitor network configuration
-w /etc/network/interfaces -p wa -k network_changes
-w /etc/netplan -p wa -k netplan_changes

# Monitor system calls
-a always,exit -F arch=b64 -S mount -S umount2 -k mount
-a always,exit -F arch=b64 -S adjtimex -S settimeofday -k time_change
```

### 10.2 Compliance Documentation

**Maintain Compliance Records:**

```bash
# Document compliance measures
cat > compliance-checklist.md << EOF
# Compliance Checklist

## Security Controls
- [ ] Firewall configured
- [ ] Encryption enabled
- [ ] Access controls implemented
- [ ] Monitoring configured
- [ ] Backups configured

## Audit Requirements
- [ ] Audit logging enabled
- [ ] Log retention configured
- [ ] Regular audits scheduled
EOF
```

### 10.3 Regular Security Audits

**Schedule Audits:**

```bash
# Create audit script
sudo nano /usr/local/bin/security-audit.sh
```

```bash
#!/bin/bash
echo "=== Security Audit Report ===" > /var/log/security-audit.log
echo "Date: $(date)" >> /var/log/security-audit.log
echo "" >> /var/log/security-audit.log

# Check for failed logins
echo "Failed Login Attempts:" >> /var/log/security-audit.log
grep "Failed password" /var/log/auth.log | tail -20 >> /var/log/security-audit.log

# Check for sudo usage
echo "" >> /var/log/security-audit.log
echo "Sudo Usage:" >> /var/log/security-audit.log
grep "sudo" /var/log/auth.log | tail -20 >> /var/log/security-audit.log

# Check firewall status
echo "" >> /var/log/security-audit.log
echo "Firewall Status:" >> /var/log/security-audit.log
sudo ufw status verbose >> /var/log/security-audit.log
```

---

## 11. Deployment Automation Security

### 11.1 Secure Configuration Management

**Use Ansible with Vault:**

```bash
# Encrypt sensitive data
ansible-vault create secrets.yml

# Use in playbooks
ansible-playbook deploy.yml --ask-vault-pass
```

**Secure Credential Storage:**

```bash
# Use environment variables
export DB_PASSWORD=$(cat /secure/password-file)

# Use secret management tools
# HashiCorp Vault, AWS Secrets Manager, etc.
```

### 11.2 Infrastructure as Code Security

**Secure Terraform/CloudFormation:**

```bash
# Never commit secrets
echo "*.tfvars" >> .gitignore
echo "secrets/" >> .gitignore

# Use secret management
# AWS Secrets Manager, Azure Key Vault, etc.
```

### 11.3 CI/CD Pipeline Security

**Secure Pipeline Configuration:**

- Use secrets management in CI/CD
- Scan code for secrets before deployment
- Verify artifact integrity
- Use signed containers/images

**Example GitLab CI:**

```yaml
deploy:
  script:
    - echo $CI_JOB_TOKEN | docker login -u gitlab-ci-token --password-stdin $CI_REGISTRY
    - docker pull $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
    - ansible-playbook deploy.yml --vault-password-file .vault-pass
  only:
    - main
```

---

## 12. Deployment Checklist

Use this checklist for each server deployment:

### Pre-Deployment
- [ ] Security requirements documented
- [ ] Risk assessment completed
- [ ] Network architecture designed
- [ ] Firewall rules defined
- [ ] Access controls planned

### Deployment
- [ ] OS hardening completed (see Ubuntu OS Hardening Guide)
- [ ] Network configuration applied
- [ ] Firewall configured and enabled
- [ ] SSH access via bastion configured
- [ ] Monitoring configured
- [ ] Logging configured
- [ ] Backups configured

### Post-Deployment
- [ ] Security testing completed
- [ ] Access verified
- [ ] Monitoring verified
- [ ] Backup tested
- [ ] Documentation updated
- [ ] Compliance verified

---

## 13. Best Practices Summary

### Network Security
- Use network segmentation (DMZ, internal, management zones)
- Implement bastion hosts for secure access
- Configure strict firewall rules
- Monitor network traffic

### Access Control
- Use key-based authentication
- Implement least privilege access
- Use bastion hosts for all remote access
- Monitor and log all access

### Communication Security
- Encrypt all sensitive communications
- Use TLS/SSL for all services
- Implement service-to-service authentication
- Secure backup communications

### Monitoring and Logging
- Centralize logging
- Monitor security events
- Set up alerts for suspicious activity
- Regular log review

### Backup and Recovery
- Automated backups
- Encrypted backups
- Off-site backup storage
- Regular backup testing
- Documented recovery procedures

---

## Troubleshooting

### Common Issues

**Cannot Access Server via Bastion:**
- Verify bastion SSH configuration
- Check firewall rules on bastion
- Verify SSH keys are correct
- Check network connectivity

**Port Forwarding Not Working:**
- Verify `AllowTcpForwarding` is enabled in sshd_config
- Check firewall rules
- Verify network connectivity
- Check for port conflicts

**Backup Failures:**
- Verify backup script permissions
- Check disk space
- Verify network connectivity to backup server
- Check encryption key availability

---

## References

- [Ubuntu OS Hardening Guide](./ubuntu-os-hardening.md)
- [Network Security Policy](../policies/network-security-policy.md)
- [Remote Access Policy](../policies/remote-access-policy.md)
- [SSH Tunneling Guide](https://www.ssh.com/academy/ssh/tunneling)
- [Bastion Host Best Practices](https://aws.amazon.com/solutions/implementations/linux-bastion/)

---

## Document Control

**Version History:**
- 1.0 (2025) - Initial release

**Review Schedule:**
This document should be reviewed annually or when significant deployment changes occur.

