# rocketchat Installation Guide

rocketchat is a free and open-source open source team communication platform. Rocket.Chat offers real-time chat, video conferencing, and team collaboration, serving as an alternative to Slack or Microsoft Teams

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
  - CPU: 2+ cores minimum
  - RAM: 2GB minimum (4GB+ recommended)
  - Storage: 10GB for data
  - Network: HTTPS and WebSocket
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 3000 (default rocketchat port)
  - Port 3001 for MongoDB
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

# Install rocketchat
sudo dnf install -y rocketchat

# Enable and start service
sudo systemctl enable --now rocketchat

# Configure firewall
sudo firewall-cmd --permanent --add-port=3000/tcp
sudo firewall-cmd --reload

# Verify installation
rocketchat --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install rocketchat
sudo apt install -y rocketchat

# Enable and start service
sudo systemctl enable --now rocketchat

# Configure firewall
sudo ufw allow 3000

# Verify installation
rocketchat --version
```

### Arch Linux

```bash
# Install rocketchat
sudo pacman -S rocketchat

# Enable and start service
sudo systemctl enable --now rocketchat

# Verify installation
rocketchat --version
```

### Alpine Linux

```bash
# Install rocketchat
apk add --no-cache rocketchat

# Enable and start service
rc-update add rocketchat default
rc-service rocketchat start

# Verify installation
rocketchat --version
```

### openSUSE/SLES

```bash
# Install rocketchat
sudo zypper install -y rocketchat

# Enable and start service
sudo systemctl enable --now rocketchat

# Configure firewall
sudo firewall-cmd --permanent --add-port=3000/tcp
sudo firewall-cmd --reload

# Verify installation
rocketchat --version
```

### macOS

```bash
# Using Homebrew
brew install rocketchat

# Start service
brew services start rocketchat

# Verify installation
rocketchat --version
```

### FreeBSD

```bash
# Using pkg
pkg install rocketchat

# Enable in rc.conf
echo 'rocketchat_enable="YES"' >> /etc/rc.conf

# Start service
service rocketchat start

# Verify installation
rocketchat --version
```

### Windows

```bash
# Using Chocolatey
choco install rocketchat

# Or using Scoop
scoop install rocketchat

# Verify installation
rocketchat --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/rocketchat

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
rocketchat --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable rocketchat

# Start service
sudo systemctl start rocketchat

# Stop service
sudo systemctl stop rocketchat

# Restart service
sudo systemctl restart rocketchat

# Check status
sudo systemctl status rocketchat

# View logs
sudo journalctl -u rocketchat -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add rocketchat default

# Start service
rc-service rocketchat start

# Stop service
rc-service rocketchat stop

# Restart service
rc-service rocketchat restart

# Check status
rc-service rocketchat status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'rocketchat_enable="YES"' >> /etc/rc.conf

# Start service
service rocketchat start

# Stop service
service rocketchat stop

# Restart service
service rocketchat restart

# Check status
service rocketchat status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start rocketchat
brew services stop rocketchat
brew services restart rocketchat

# Check status
brew services list | grep rocketchat
```

### Windows Service Manager

```powershell
# Start service
net start rocketchat

# Stop service
net stop rocketchat

# Using PowerShell
Start-Service rocketchat
Stop-Service rocketchat
Restart-Service rocketchat

# Check status
Get-Service rocketchat
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream rocketchat_backend {
    server 127.0.0.1:3000;
}

server {
    listen 80;
    server_name rocketchat.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name rocketchat.example.com;

    ssl_certificate /etc/ssl/certs/rocketchat.example.com.crt;
    ssl_certificate_key /etc/ssl/private/rocketchat.example.com.key;

    location / {
        proxy_pass http://rocketchat_backend;
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
    ServerName rocketchat.example.com
    Redirect permanent / https://rocketchat.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName rocketchat.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/rocketchat.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/rocketchat.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:3000/
    ProxyPassReverse / http://127.0.0.1:3000/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend rocketchat_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/rocketchat.pem
    redirect scheme https if !{ ssl_fc }
    default_backend rocketchat_backend

backend rocketchat_backend
    balance roundrobin
    server rocketchat1 127.0.0.1:3000 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R rocketchat:rocketchat /etc/rocketchat
sudo chmod 750 /etc/rocketchat

# Configure firewall
sudo firewall-cmd --permanent --add-port=3000/tcp
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
sudo systemctl status rocketchat

# View logs
sudo journalctl -u rocketchat -f

# Monitor resource usage
top -p $(pgrep rocketchat)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/rocketchat"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/rocketchat-backup-$DATE.tar.gz" /etc/rocketchat /var/lib/rocketchat

echo "Backup completed: $BACKUP_DIR/rocketchat-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop rocketchat

# Restore from backup
tar -xzf /backup/rocketchat/rocketchat-backup-*.tar.gz -C /

# Start service
sudo systemctl start rocketchat
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u rocketchat -n 100
sudo tail -f /var/log/rocketchat/rocketchat.log

# Check configuration
rocketchat --version

# Check permissions
ls -la /etc/rocketchat
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 3000

# Test connectivity
telnet localhost 3000

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep rocketchat)

# Check disk I/O
iotop -p $(pgrep rocketchat)

# Check connections
ss -an | grep 3000
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  rocketchat:
    image: rocketchat:latest
    ports:
      - "3000:3000"
    volumes:
      - ./config:/etc/rocketchat
      - ./data:/var/lib/rocketchat
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update rocketchat

# Debian/Ubuntu
sudo apt update && sudo apt upgrade rocketchat

# Arch Linux
sudo pacman -Syu rocketchat

# Alpine Linux
apk update && apk upgrade rocketchat

# openSUSE
sudo zypper update rocketchat

# FreeBSD
pkg update && pkg upgrade rocketchat

# Always backup before updates
tar -czf /backup/rocketchat-pre-update-$(date +%Y%m%d).tar.gz /etc/rocketchat

# Restart after updates
sudo systemctl restart rocketchat
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/rocketchat

# Clean old logs
find /var/log/rocketchat -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/rocketchat
```

## Additional Resources

- Official Documentation: https://docs.rocketchat.org/
- GitHub Repository: https://github.com/rocketchat/rocketchat
- Community Forum: https://forum.rocketchat.org/
- Best Practices Guide: https://docs.rocketchat.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
