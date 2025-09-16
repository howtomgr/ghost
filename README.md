# ghost Installation Guide

ghost is a free and open-source publishing platform. Ghost provides modern publishing platform for professional publishers

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
  - CPU: 1 core minimum
  - RAM: 1GB minimum
  - Storage: 1GB for data
  - Network: HTTP/HTTPS access
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 2368 (default ghost port)
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

# Install ghost
sudo dnf install -y ghost

# Enable and start service
sudo systemctl enable --now ghost

# Configure firewall
sudo firewall-cmd --permanent --add-port=2368/tcp
sudo firewall-cmd --reload

# Verify installation
ghost --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install ghost
sudo apt install -y ghost

# Enable and start service
sudo systemctl enable --now ghost

# Configure firewall
sudo ufw allow 2368

# Verify installation
ghost --version
```

### Arch Linux

```bash
# Install ghost
sudo pacman -S ghost

# Enable and start service
sudo systemctl enable --now ghost

# Verify installation
ghost --version
```

### Alpine Linux

```bash
# Install ghost
apk add --no-cache ghost

# Enable and start service
rc-update add ghost default
rc-service ghost start

# Verify installation
ghost --version
```

### openSUSE/SLES

```bash
# Install ghost
sudo zypper install -y ghost

# Enable and start service
sudo systemctl enable --now ghost

# Configure firewall
sudo firewall-cmd --permanent --add-port=2368/tcp
sudo firewall-cmd --reload

# Verify installation
ghost --version
```

### macOS

```bash
# Using Homebrew
brew install ghost

# Start service
brew services start ghost

# Verify installation
ghost --version
```

### FreeBSD

```bash
# Using pkg
pkg install ghost

# Enable in rc.conf
echo 'ghost_enable="YES"' >> /etc/rc.conf

# Start service
service ghost start

# Verify installation
ghost --version
```

### Windows

```bash
# Using Chocolatey
choco install ghost

# Or using Scoop
scoop install ghost

# Verify installation
ghost --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/ghost

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
ghost --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable ghost

# Start service
sudo systemctl start ghost

# Stop service
sudo systemctl stop ghost

# Restart service
sudo systemctl restart ghost

# Check status
sudo systemctl status ghost

# View logs
sudo journalctl -u ghost -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add ghost default

# Start service
rc-service ghost start

# Stop service
rc-service ghost stop

# Restart service
rc-service ghost restart

# Check status
rc-service ghost status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'ghost_enable="YES"' >> /etc/rc.conf

# Start service
service ghost start

# Stop service
service ghost stop

# Restart service
service ghost restart

# Check status
service ghost status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start ghost
brew services stop ghost
brew services restart ghost

# Check status
brew services list | grep ghost
```

### Windows Service Manager

```powershell
# Start service
net start ghost

# Stop service
net stop ghost

# Using PowerShell
Start-Service ghost
Stop-Service ghost
Restart-Service ghost

# Check status
Get-Service ghost
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream ghost_backend {
    server 127.0.0.1:2368;
}

server {
    listen 80;
    server_name ghost.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name ghost.example.com;

    ssl_certificate /etc/ssl/certs/ghost.example.com.crt;
    ssl_certificate_key /etc/ssl/private/ghost.example.com.key;

    location / {
        proxy_pass http://ghost_backend;
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
    ServerName ghost.example.com
    Redirect permanent / https://ghost.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName ghost.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/ghost.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/ghost.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:2368/
    ProxyPassReverse / http://127.0.0.1:2368/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend ghost_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/ghost.pem
    redirect scheme https if !{ ssl_fc }
    default_backend ghost_backend

backend ghost_backend
    balance roundrobin
    server ghost1 127.0.0.1:2368 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R ghost:ghost /etc/ghost
sudo chmod 750 /etc/ghost

# Configure firewall
sudo firewall-cmd --permanent --add-port=2368/tcp
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
sudo systemctl status ghost

# View logs
sudo journalctl -u ghost -f

# Monitor resource usage
top -p $(pgrep ghost)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/ghost"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/ghost-backup-$DATE.tar.gz" /etc/ghost /var/lib/ghost

echo "Backup completed: $BACKUP_DIR/ghost-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop ghost

# Restore from backup
tar -xzf /backup/ghost/ghost-backup-*.tar.gz -C /

# Start service
sudo systemctl start ghost
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u ghost -n 100
sudo tail -f /var/log/ghost/ghost.log

# Check configuration
ghost --version

# Check permissions
ls -la /etc/ghost
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 2368

# Test connectivity
telnet localhost 2368

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep ghost)

# Check disk I/O
iotop -p $(pgrep ghost)

# Check connections
ss -an | grep 2368
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  ghost:
    image: ghost:latest
    ports:
      - "2368:2368"
    volumes:
      - ./config:/etc/ghost
      - ./data:/var/lib/ghost
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update ghost

# Debian/Ubuntu
sudo apt update && sudo apt upgrade ghost

# Arch Linux
sudo pacman -Syu ghost

# Alpine Linux
apk update && apk upgrade ghost

# openSUSE
sudo zypper update ghost

# FreeBSD
pkg update && pkg upgrade ghost

# Always backup before updates
tar -czf /backup/ghost-pre-update-$(date +%Y%m%d).tar.gz /etc/ghost

# Restart after updates
sudo systemctl restart ghost
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/ghost

# Clean old logs
find /var/log/ghost -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/ghost
```

## Additional Resources

- Official Documentation: https://docs.ghost.org/
- GitHub Repository: https://github.com/ghost/ghost
- Community Forum: https://forum.ghost.org/
- Best Practices Guide: https://docs.ghost.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
