# ttn Installation Guide

ttn is a free and open-source LoRaWAN platform. The Things Stack provides LoRaWAN network server

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
  - RAM: 4GB minimum
  - Storage: 20GB for data
  - Network: HTTP/LoRa
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 1885 (default ttn port)
  - Various services
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

# Install ttn
sudo dnf install -y ttn

# Enable and start service
sudo systemctl enable --now ttn

# Configure firewall
sudo firewall-cmd --permanent --add-port=1885/tcp
sudo firewall-cmd --reload

# Verify installation
ttn --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install ttn
sudo apt install -y ttn

# Enable and start service
sudo systemctl enable --now ttn

# Configure firewall
sudo ufw allow 1885

# Verify installation
ttn --version
```

### Arch Linux

```bash
# Install ttn
sudo pacman -S ttn

# Enable and start service
sudo systemctl enable --now ttn

# Verify installation
ttn --version
```

### Alpine Linux

```bash
# Install ttn
apk add --no-cache ttn

# Enable and start service
rc-update add ttn default
rc-service ttn start

# Verify installation
ttn --version
```

### openSUSE/SLES

```bash
# Install ttn
sudo zypper install -y ttn

# Enable and start service
sudo systemctl enable --now ttn

# Configure firewall
sudo firewall-cmd --permanent --add-port=1885/tcp
sudo firewall-cmd --reload

# Verify installation
ttn --version
```

### macOS

```bash
# Using Homebrew
brew install ttn

# Start service
brew services start ttn

# Verify installation
ttn --version
```

### FreeBSD

```bash
# Using pkg
pkg install ttn

# Enable in rc.conf
echo 'ttn_enable="YES"' >> /etc/rc.conf

# Start service
service ttn start

# Verify installation
ttn --version
```

### Windows

```bash
# Using Chocolatey
choco install ttn

# Or using Scoop
scoop install ttn

# Verify installation
ttn --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/ttn

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
ttn --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable ttn

# Start service
sudo systemctl start ttn

# Stop service
sudo systemctl stop ttn

# Restart service
sudo systemctl restart ttn

# Check status
sudo systemctl status ttn

# View logs
sudo journalctl -u ttn -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add ttn default

# Start service
rc-service ttn start

# Stop service
rc-service ttn stop

# Restart service
rc-service ttn restart

# Check status
rc-service ttn status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'ttn_enable="YES"' >> /etc/rc.conf

# Start service
service ttn start

# Stop service
service ttn stop

# Restart service
service ttn restart

# Check status
service ttn status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start ttn
brew services stop ttn
brew services restart ttn

# Check status
brew services list | grep ttn
```

### Windows Service Manager

```powershell
# Start service
net start ttn

# Stop service
net stop ttn

# Using PowerShell
Start-Service ttn
Stop-Service ttn
Restart-Service ttn

# Check status
Get-Service ttn
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream ttn_backend {
    server 127.0.0.1:1885;
}

server {
    listen 80;
    server_name ttn.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name ttn.example.com;

    ssl_certificate /etc/ssl/certs/ttn.example.com.crt;
    ssl_certificate_key /etc/ssl/private/ttn.example.com.key;

    location / {
        proxy_pass http://ttn_backend;
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
    ServerName ttn.example.com
    Redirect permanent / https://ttn.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName ttn.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/ttn.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/ttn.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:1885/
    ProxyPassReverse / http://127.0.0.1:1885/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend ttn_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/ttn.pem
    redirect scheme https if !{ ssl_fc }
    default_backend ttn_backend

backend ttn_backend
    balance roundrobin
    server ttn1 127.0.0.1:1885 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R ttn:ttn /etc/ttn
sudo chmod 750 /etc/ttn

# Configure firewall
sudo firewall-cmd --permanent --add-port=1885/tcp
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
sudo systemctl status ttn

# View logs
sudo journalctl -u ttn -f

# Monitor resource usage
top -p $(pgrep ttn)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/ttn"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/ttn-backup-$DATE.tar.gz" /etc/ttn /var/lib/ttn

echo "Backup completed: $BACKUP_DIR/ttn-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop ttn

# Restore from backup
tar -xzf /backup/ttn/ttn-backup-*.tar.gz -C /

# Start service
sudo systemctl start ttn
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u ttn -n 100
sudo tail -f /var/log/ttn/ttn.log

# Check configuration
ttn --version

# Check permissions
ls -la /etc/ttn
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 1885

# Test connectivity
telnet localhost 1885

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep ttn)

# Check disk I/O
iotop -p $(pgrep ttn)

# Check connections
ss -an | grep 1885
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  ttn:
    image: ttn:latest
    ports:
      - "1885:1885"
    volumes:
      - ./config:/etc/ttn
      - ./data:/var/lib/ttn
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update ttn

# Debian/Ubuntu
sudo apt update && sudo apt upgrade ttn

# Arch Linux
sudo pacman -Syu ttn

# Alpine Linux
apk update && apk upgrade ttn

# openSUSE
sudo zypper update ttn

# FreeBSD
pkg update && pkg upgrade ttn

# Always backup before updates
tar -czf /backup/ttn-pre-update-$(date +%Y%m%d).tar.gz /etc/ttn

# Restart after updates
sudo systemctl restart ttn
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/ttn

# Clean old logs
find /var/log/ttn -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/ttn
```

## Additional Resources

- Official Documentation: https://docs.ttn.org/
- GitHub Repository: https://github.com/ttn/ttn
- Community Forum: https://forum.ttn.org/
- Best Practices Guide: https://docs.ttn.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
