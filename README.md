# arangodb Installation Guide

arangodb is a free and open-source multi-model NoSQL database. ArangoDB combines document, graph, and key-value data models with one query language, serving as an alternative to using multiple specialized databases

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
  - CPU: 2+ cores recommended
  - RAM: 2GB minimum (8GB+ recommended)
  - Storage: 10GB+ for data
  - Network: HTTP/HTTPS access
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 8529 (default arangodb port)
  - Cluster ports if distributed
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

# Install arangodb
sudo dnf install -y arangodb

# Enable and start service
sudo systemctl enable --now arangodb

# Configure firewall
sudo firewall-cmd --permanent --add-port=8529/tcp
sudo firewall-cmd --reload

# Verify installation
arangod --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install arangodb
sudo apt install -y arangodb

# Enable and start service
sudo systemctl enable --now arangodb

# Configure firewall
sudo ufw allow 8529

# Verify installation
arangod --version
```

### Arch Linux

```bash
# Install arangodb
sudo pacman -S arangodb

# Enable and start service
sudo systemctl enable --now arangodb

# Verify installation
arangod --version
```

### Alpine Linux

```bash
# Install arangodb
apk add --no-cache arangodb

# Enable and start service
rc-update add arangodb default
rc-service arangodb start

# Verify installation
arangod --version
```

### openSUSE/SLES

```bash
# Install arangodb
sudo zypper install -y arangodb

# Enable and start service
sudo systemctl enable --now arangodb

# Configure firewall
sudo firewall-cmd --permanent --add-port=8529/tcp
sudo firewall-cmd --reload

# Verify installation
arangod --version
```

### macOS

```bash
# Using Homebrew
brew install arangodb

# Start service
brew services start arangodb

# Verify installation
arangod --version
```

### FreeBSD

```bash
# Using pkg
pkg install arangodb

# Enable in rc.conf
echo 'arangodb_enable="YES"' >> /etc/rc.conf

# Start service
service arangodb start

# Verify installation
arangod --version
```

### Windows

```bash
# Using Chocolatey
choco install arangodb

# Or using Scoop
scoop install arangodb

# Verify installation
arangod --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/arangodb

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
arangod --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable arangodb

# Start service
sudo systemctl start arangodb

# Stop service
sudo systemctl stop arangodb

# Restart service
sudo systemctl restart arangodb

# Check status
sudo systemctl status arangodb

# View logs
sudo journalctl -u arangodb -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add arangodb default

# Start service
rc-service arangodb start

# Stop service
rc-service arangodb stop

# Restart service
rc-service arangodb restart

# Check status
rc-service arangodb status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'arangodb_enable="YES"' >> /etc/rc.conf

# Start service
service arangodb start

# Stop service
service arangodb stop

# Restart service
service arangodb restart

# Check status
service arangodb status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start arangodb
brew services stop arangodb
brew services restart arangodb

# Check status
brew services list | grep arangodb
```

### Windows Service Manager

```powershell
# Start service
net start arangodb

# Stop service
net stop arangodb

# Using PowerShell
Start-Service arangodb
Stop-Service arangodb
Restart-Service arangodb

# Check status
Get-Service arangodb
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream arangodb_backend {
    server 127.0.0.1:8529;
}

server {
    listen 80;
    server_name arangodb.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name arangodb.example.com;

    ssl_certificate /etc/ssl/certs/arangodb.example.com.crt;
    ssl_certificate_key /etc/ssl/private/arangodb.example.com.key;

    location / {
        proxy_pass http://arangodb_backend;
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
    ServerName arangodb.example.com
    Redirect permanent / https://arangodb.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName arangodb.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/arangodb.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/arangodb.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:8529/
    ProxyPassReverse / http://127.0.0.1:8529/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend arangodb_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/arangodb.pem
    redirect scheme https if !{ ssl_fc }
    default_backend arangodb_backend

backend arangodb_backend
    balance roundrobin
    server arangodb1 127.0.0.1:8529 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R arangodb:arangodb /etc/arangodb
sudo chmod 750 /etc/arangodb

# Configure firewall
sudo firewall-cmd --permanent --add-port=8529/tcp
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
sudo systemctl status arangodb

# View logs
sudo journalctl -u arangodb -f

# Monitor resource usage
top -p $(pgrep arangodb)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/arangodb"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/arangodb-backup-$DATE.tar.gz" /etc/arangodb /var/lib/arangodb

echo "Backup completed: $BACKUP_DIR/arangodb-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop arangodb

# Restore from backup
tar -xzf /backup/arangodb/arangodb-backup-*.tar.gz -C /

# Start service
sudo systemctl start arangodb
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u arangodb -n 100
sudo tail -f /var/log/arangodb/arangodb.log

# Check configuration
arangod --version

# Check permissions
ls -la /etc/arangodb
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 8529

# Test connectivity
telnet localhost 8529

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep arangodb)

# Check disk I/O
iotop -p $(pgrep arangodb)

# Check connections
ss -an | grep 8529
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  arangodb:
    image: arangodb:latest
    ports:
      - "8529:8529"
    volumes:
      - ./config:/etc/arangodb
      - ./data:/var/lib/arangodb
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update arangodb

# Debian/Ubuntu
sudo apt update && sudo apt upgrade arangodb

# Arch Linux
sudo pacman -Syu arangodb

# Alpine Linux
apk update && apk upgrade arangodb

# openSUSE
sudo zypper update arangodb

# FreeBSD
pkg update && pkg upgrade arangodb

# Always backup before updates
tar -czf /backup/arangodb-pre-update-$(date +%Y%m%d).tar.gz /etc/arangodb

# Restart after updates
sudo systemctl restart arangodb
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/arangodb

# Clean old logs
find /var/log/arangodb -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/arangodb
```

## Additional Resources

- Official Documentation: https://docs.arangodb.org/
- GitHub Repository: https://github.com/arangodb/arangodb
- Community Forum: https://forum.arangodb.org/
- Best Practices Guide: https://docs.arangodb.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
