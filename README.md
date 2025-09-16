# moodle Installation Guide

moodle is a free and open-source learning platform. Moodle provides open source learning management system

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Supported Operating Systems](#supported-operating-systems)
3. [Installation](#installation)
4. [Configuration](#configuration)
5. [Service Management](#service-management)
6. [Troubleshooting](#troubleshooting)
7. [Security Considerations](#security-considerations)
8. [Performance Tuning](#performance-tuning)
9. [Backup and Restore](#backup-and-restore)
10. [System Requirements](#system-requirements)
11. [Support](#support)
12. [Contributing](#contributing)
13. [License](#license)
14. [Acknowledgments](#acknowledgments)
15. [Version History](#version-history)
16. [Appendices](#appendices)

## 1. Prerequisites

- **Hardware Requirements**:
  - CPU: 2+ cores
  - RAM: 2GB minimum
  - Storage: 20GB for courses
  - Network: HTTP/HTTPS
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 80 (default moodle port)
  - None
- **Dependencies**:
  - See official documentation for specific requirements
- **System Access**: root or sudo privileges required


## 2. Supported Operating Systems

This guide supports installation on:
- RHEL 8/9 and derivatives (CentOS Stream, Rocky Linux, AlmaLinux)
- Debian 11/12
- Ubuntu 20.04/22.04/24.04 LTS
- Arch Linux (rolling release)
- Alpine Linux 3.18+
- openSUSE Leap 15.5+ / Tumbleweed
- SUSE Linux Enterprise Server (SLES) 15+
- macOS 12+ (Monterey and later) 
- FreeBSD 13+
- Windows 10/11/Server 2019+ (where applicable)

## 3. Installation

### RHEL/CentOS/Rocky Linux/AlmaLinux

```bash
# Install EPEL repository if needed
sudo dnf install -y epel-release

# Install moodle
sudo dnf install -y moodle

# Enable and start service
sudo systemctl enable --now moodle

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload

# Verify installation
moodle --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install moodle
sudo apt install -y moodle

# Enable and start service
sudo systemctl enable --now moodle

# Configure firewall
sudo ufw allow 80

# Verify installation
moodle --version
```

### Arch Linux

```bash
# Install moodle
sudo pacman -S moodle

# Enable and start service
sudo systemctl enable --now moodle

# Verify installation
moodle --version
```

### Alpine Linux

```bash
# Install moodle
apk add --no-cache moodle

# Enable and start service
rc-update add moodle default
rc-service moodle start

# Verify installation
moodle --version
```

### openSUSE/SLES

```bash
# Install moodle
sudo zypper install -y moodle

# Enable and start service
sudo systemctl enable --now moodle

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload

# Verify installation
moodle --version
```

### macOS

```bash
# Using Homebrew
brew install moodle

# Start service
brew services start moodle

# Verify installation
moodle --version
```

### FreeBSD

```bash
# Using pkg
pkg install moodle

# Enable in rc.conf
echo 'moodle_enable="YES"' >> /etc/rc.conf

# Start service
service moodle start

# Verify installation
moodle --version
```

### Windows

```bash
# Using Chocolatey
choco install moodle

# Or using Scoop
scoop install moodle

# Verify installation
moodle --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/moodle

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
moodle --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable moodle

# Start service
sudo systemctl start moodle

# Stop service
sudo systemctl stop moodle

# Restart service
sudo systemctl restart moodle

# Check status
sudo systemctl status moodle

# View logs
sudo journalctl -u moodle -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add moodle default

# Start service
rc-service moodle start

# Stop service
rc-service moodle stop

# Restart service
rc-service moodle restart

# Check status
rc-service moodle status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'moodle_enable="YES"' >> /etc/rc.conf

# Start service
service moodle start

# Stop service
service moodle stop

# Restart service
service moodle restart

# Check status
service moodle status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start moodle
brew services stop moodle
brew services restart moodle

# Check status
brew services list | grep moodle
```

### Windows Service Manager

```powershell
# Start service
net start moodle

# Stop service
net stop moodle

# Using PowerShell
Start-Service moodle
Stop-Service moodle
Restart-Service moodle

# Check status
Get-Service moodle
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream moodle_backend {
    server 127.0.0.1:80;
}

server {
    listen 80;
    server_name moodle.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name moodle.example.com;

    ssl_certificate /etc/ssl/certs/moodle.example.com.crt;
    ssl_certificate_key /etc/ssl/private/moodle.example.com.key;

    location / {
        proxy_pass http://moodle_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Apache Configuration

```apache
<VirtualHost *:80>
    ServerName moodle.example.com
    Redirect permanent / https://moodle.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName moodle.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/moodle.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/moodle.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:80/
    ProxyPassReverse / http://127.0.0.1:80/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend moodle_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/moodle.pem
    redirect scheme https if !{ ssl_fc }
    default_backend moodle_backend

backend moodle_backend
    balance roundrobin
    server moodle1 127.0.0.1:80 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R moodle:moodle /etc/moodle
sudo chmod 750 /etc/moodle

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on
```

## Database Setup

See official documentation for database configuration requirements.

## Performance Optimization

### System Tuning

```bash
# Basic system tuning
echo 'net.core.somaxconn = 65535' | sudo tee -a /etc/sysctl.conf
echo 'net.ipv4.tcp_max_syn_backlog = 65535' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

## Monitoring

### Basic Monitoring

```bash
# Check service status
sudo systemctl status moodle

# View logs
sudo journalctl -u moodle -f

# Monitor resource usage
top -p $(pgrep moodle)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/moodle"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/moodle-backup-$DATE.tar.gz" /etc/moodle /var/lib/moodle

echo "Backup completed: $BACKUP_DIR/moodle-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop moodle

# Restore from backup
tar -xzf /backup/moodle/moodle-backup-*.tar.gz -C /

# Start service
sudo systemctl start moodle
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u moodle -n 100
sudo tail -f /var/log/moodle/moodle.log

# Check configuration
moodle --version

# Check permissions
ls -la /etc/moodle
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 80

# Test connectivity
telnet localhost 80

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep moodle)

# Check disk I/O
iotop -p $(pgrep moodle)

# Check connections
ss -an | grep 80
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  moodle:
    image: moodle:latest
    ports:
      - "80:80"
    volumes:
      - ./config:/etc/moodle
      - ./data:/var/lib/moodle
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update moodle

# Debian/Ubuntu
sudo apt update && sudo apt upgrade moodle

# Arch Linux
sudo pacman -Syu moodle

# Alpine Linux
apk update && apk upgrade moodle

# openSUSE
sudo zypper update moodle

# FreeBSD
pkg update && pkg upgrade moodle

# Always backup before updates
tar -czf /backup/moodle-pre-update-$(date +%Y%m%d).tar.gz /etc/moodle

# Restart after updates
sudo systemctl restart moodle
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/moodle

# Clean old logs
find /var/log/moodle -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/moodle
```

## Additional Resources

- Official Documentation: https://docs.moodle.org/
- GitHub Repository: https://github.com/moodle/moodle
- Community Forum: https://forum.moodle.org/
- Best Practices Guide: https://docs.moodle.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
