# orangefs Installation Guide

orangefs is a free and open-source parallel file system. OrangeFS provides scale-out parallel file system

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
  - Storage: 100GB+ per server
  - Network: Native protocol
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 3334 (default orangefs port)
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

# Install orangefs
sudo dnf install -y orangefs

# Enable and start service
sudo systemctl enable --now orangefs

# Configure firewall
sudo firewall-cmd --permanent --add-port=3334/tcp
sudo firewall-cmd --reload

# Verify installation
orangefs --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install orangefs
sudo apt install -y orangefs

# Enable and start service
sudo systemctl enable --now orangefs

# Configure firewall
sudo ufw allow 3334

# Verify installation
orangefs --version
```

### Arch Linux

```bash
# Install orangefs
sudo pacman -S orangefs

# Enable and start service
sudo systemctl enable --now orangefs

# Verify installation
orangefs --version
```

### Alpine Linux

```bash
# Install orangefs
apk add --no-cache orangefs

# Enable and start service
rc-update add orangefs default
rc-service orangefs start

# Verify installation
orangefs --version
```

### openSUSE/SLES

```bash
# Install orangefs
sudo zypper install -y orangefs

# Enable and start service
sudo systemctl enable --now orangefs

# Configure firewall
sudo firewall-cmd --permanent --add-port=3334/tcp
sudo firewall-cmd --reload

# Verify installation
orangefs --version
```

### macOS

```bash
# Using Homebrew
brew install orangefs

# Start service
brew services start orangefs

# Verify installation
orangefs --version
```

### FreeBSD

```bash
# Using pkg
pkg install orangefs

# Enable in rc.conf
echo 'orangefs_enable="YES"' >> /etc/rc.conf

# Start service
service orangefs start

# Verify installation
orangefs --version
```

### Windows

```bash
# Using Chocolatey
choco install orangefs

# Or using Scoop
scoop install orangefs

# Verify installation
orangefs --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/orangefs

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
orangefs --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable orangefs

# Start service
sudo systemctl start orangefs

# Stop service
sudo systemctl stop orangefs

# Restart service
sudo systemctl restart orangefs

# Check status
sudo systemctl status orangefs

# View logs
sudo journalctl -u orangefs -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add orangefs default

# Start service
rc-service orangefs start

# Stop service
rc-service orangefs stop

# Restart service
rc-service orangefs restart

# Check status
rc-service orangefs status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'orangefs_enable="YES"' >> /etc/rc.conf

# Start service
service orangefs start

# Stop service
service orangefs stop

# Restart service
service orangefs restart

# Check status
service orangefs status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start orangefs
brew services stop orangefs
brew services restart orangefs

# Check status
brew services list | grep orangefs
```

### Windows Service Manager

```powershell
# Start service
net start orangefs

# Stop service
net stop orangefs

# Using PowerShell
Start-Service orangefs
Stop-Service orangefs
Restart-Service orangefs

# Check status
Get-Service orangefs
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream orangefs_backend {
    server 127.0.0.1:3334;
}

server {
    listen 80;
    server_name orangefs.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name orangefs.example.com;

    ssl_certificate /etc/ssl/certs/orangefs.example.com.crt;
    ssl_certificate_key /etc/ssl/private/orangefs.example.com.key;

    location / {
        proxy_pass http://orangefs_backend;
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
    ServerName orangefs.example.com
    Redirect permanent / https://orangefs.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName orangefs.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/orangefs.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/orangefs.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:3334/
    ProxyPassReverse / http://127.0.0.1:3334/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend orangefs_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/orangefs.pem
    redirect scheme https if !{ ssl_fc }
    default_backend orangefs_backend

backend orangefs_backend
    balance roundrobin
    server orangefs1 127.0.0.1:3334 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R orangefs:orangefs /etc/orangefs
sudo chmod 750 /etc/orangefs

# Configure firewall
sudo firewall-cmd --permanent --add-port=3334/tcp
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
sudo systemctl status orangefs

# View logs
sudo journalctl -u orangefs -f

# Monitor resource usage
top -p $(pgrep orangefs)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/orangefs"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/orangefs-backup-$DATE.tar.gz" /etc/orangefs /var/lib/orangefs

echo "Backup completed: $BACKUP_DIR/orangefs-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop orangefs

# Restore from backup
tar -xzf /backup/orangefs/orangefs-backup-*.tar.gz -C /

# Start service
sudo systemctl start orangefs
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u orangefs -n 100
sudo tail -f /var/log/orangefs/orangefs.log

# Check configuration
orangefs --version

# Check permissions
ls -la /etc/orangefs
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 3334

# Test connectivity
telnet localhost 3334

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep orangefs)

# Check disk I/O
iotop -p $(pgrep orangefs)

# Check connections
ss -an | grep 3334
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  orangefs:
    image: orangefs:latest
    ports:
      - "3334:3334"
    volumes:
      - ./config:/etc/orangefs
      - ./data:/var/lib/orangefs
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update orangefs

# Debian/Ubuntu
sudo apt update && sudo apt upgrade orangefs

# Arch Linux
sudo pacman -Syu orangefs

# Alpine Linux
apk update && apk upgrade orangefs

# openSUSE
sudo zypper update orangefs

# FreeBSD
pkg update && pkg upgrade orangefs

# Always backup before updates
tar -czf /backup/orangefs-pre-update-$(date +%Y%m%d).tar.gz /etc/orangefs

# Restart after updates
sudo systemctl restart orangefs
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/orangefs

# Clean old logs
find /var/log/orangefs -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/orangefs
```

## Additional Resources

- Official Documentation: https://docs.orangefs.org/
- GitHub Repository: https://github.com/orangefs/orangefs
- Community Forum: https://forum.orangefs.org/
- Best Practices Guide: https://docs.orangefs.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
