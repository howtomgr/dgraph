# dgraph Installation Guide

dgraph is a free and open-source distributed graph database. Dgraph provides a native GraphQL database with a graph backend, serving as an open-source alternative to Neo4j or Amazon Neptune

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
  - CPU: 2+ cores minimum (4+ recommended)
  - RAM: 4GB minimum (8GB+ recommended)
  - Storage: 10GB+ for data
  - Network: HTTP and gRPC
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 8080 (default dgraph port)
  - Port 9080 for gRPC
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

# Install dgraph
sudo dnf install -y dgraph

# Enable and start service
sudo systemctl enable --now dgraph

# Configure firewall
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload

# Verify installation
dgraph version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install dgraph
sudo apt install -y dgraph

# Enable and start service
sudo systemctl enable --now dgraph

# Configure firewall
sudo ufw allow 8080

# Verify installation
dgraph version
```

### Arch Linux

```bash
# Install dgraph
sudo pacman -S dgraph

# Enable and start service
sudo systemctl enable --now dgraph

# Verify installation
dgraph version
```

### Alpine Linux

```bash
# Install dgraph
apk add --no-cache dgraph

# Enable and start service
rc-update add dgraph default
rc-service dgraph start

# Verify installation
dgraph version
```

### openSUSE/SLES

```bash
# Install dgraph
sudo zypper install -y dgraph

# Enable and start service
sudo systemctl enable --now dgraph

# Configure firewall
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload

# Verify installation
dgraph version
```

### macOS

```bash
# Using Homebrew
brew install dgraph

# Start service
brew services start dgraph

# Verify installation
dgraph version
```

### FreeBSD

```bash
# Using pkg
pkg install dgraph

# Enable in rc.conf
echo 'dgraph_enable="YES"' >> /etc/rc.conf

# Start service
service dgraph start

# Verify installation
dgraph version
```

### Windows

```bash
# Using Chocolatey
choco install dgraph

# Or using Scoop
scoop install dgraph

# Verify installation
dgraph version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/dgraph

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
dgraph version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable dgraph

# Start service
sudo systemctl start dgraph

# Stop service
sudo systemctl stop dgraph

# Restart service
sudo systemctl restart dgraph

# Check status
sudo systemctl status dgraph

# View logs
sudo journalctl -u dgraph -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add dgraph default

# Start service
rc-service dgraph start

# Stop service
rc-service dgraph stop

# Restart service
rc-service dgraph restart

# Check status
rc-service dgraph status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'dgraph_enable="YES"' >> /etc/rc.conf

# Start service
service dgraph start

# Stop service
service dgraph stop

# Restart service
service dgraph restart

# Check status
service dgraph status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start dgraph
brew services stop dgraph
brew services restart dgraph

# Check status
brew services list | grep dgraph
```

### Windows Service Manager

```powershell
# Start service
net start dgraph

# Stop service
net stop dgraph

# Using PowerShell
Start-Service dgraph
Stop-Service dgraph
Restart-Service dgraph

# Check status
Get-Service dgraph
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream dgraph_backend {
    server 127.0.0.1:8080;
}

server {
    listen 80;
    server_name dgraph.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name dgraph.example.com;

    ssl_certificate /etc/ssl/certs/dgraph.example.com.crt;
    ssl_certificate_key /etc/ssl/private/dgraph.example.com.key;

    location / {
        proxy_pass http://dgraph_backend;
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
    ServerName dgraph.example.com
    Redirect permanent / https://dgraph.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName dgraph.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/dgraph.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/dgraph.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:8080/
    ProxyPassReverse / http://127.0.0.1:8080/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend dgraph_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/dgraph.pem
    redirect scheme https if !{ ssl_fc }
    default_backend dgraph_backend

backend dgraph_backend
    balance roundrobin
    server dgraph1 127.0.0.1:8080 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R dgraph:dgraph /etc/dgraph
sudo chmod 750 /etc/dgraph

# Configure firewall
sudo firewall-cmd --permanent --add-port=8080/tcp
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
sudo systemctl status dgraph

# View logs
sudo journalctl -u dgraph -f

# Monitor resource usage
top -p $(pgrep dgraph)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/dgraph"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/dgraph-backup-$DATE.tar.gz" /etc/dgraph /var/lib/dgraph

echo "Backup completed: $BACKUP_DIR/dgraph-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop dgraph

# Restore from backup
tar -xzf /backup/dgraph/dgraph-backup-*.tar.gz -C /

# Start service
sudo systemctl start dgraph
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u dgraph -n 100
sudo tail -f /var/log/dgraph/dgraph.log

# Check configuration
dgraph version

# Check permissions
ls -la /etc/dgraph
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 8080

# Test connectivity
telnet localhost 8080

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep dgraph)

# Check disk I/O
iotop -p $(pgrep dgraph)

# Check connections
ss -an | grep 8080
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  dgraph:
    image: dgraph:latest
    ports:
      - "8080:8080"
    volumes:
      - ./config:/etc/dgraph
      - ./data:/var/lib/dgraph
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update dgraph

# Debian/Ubuntu
sudo apt update && sudo apt upgrade dgraph

# Arch Linux
sudo pacman -Syu dgraph

# Alpine Linux
apk update && apk upgrade dgraph

# openSUSE
sudo zypper update dgraph

# FreeBSD
pkg update && pkg upgrade dgraph

# Always backup before updates
tar -czf /backup/dgraph-pre-update-$(date +%Y%m%d).tar.gz /etc/dgraph

# Restart after updates
sudo systemctl restart dgraph
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/dgraph

# Clean old logs
find /var/log/dgraph -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/dgraph
```

## Additional Resources

- Official Documentation: https://docs.dgraph.org/
- GitHub Repository: https://github.com/dgraph/dgraph
- Community Forum: https://forum.dgraph.org/
- Best Practices Guide: https://docs.dgraph.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
