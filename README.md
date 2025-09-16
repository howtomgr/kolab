# Kolab Installation Guide

Kolab is a free and open-source Groupware Suite. A comprehensive groupware solution with email, calendaring, and collaboration

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
  - CPU: 2 cores minimum (4+ cores recommended)
  - RAM: 2GB minimum (4GB+ recommended)
  - Storage: 1GB for installation
  - Network: 443/143/993 ports
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 443/143/993 (default kolab port)
- **Dependencies**:
  - kolab-webadmin, roundcubemail
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

# Install kolab
sudo dnf install -y kolab kolab-webadmin, roundcubemail

# Enable and start service
sudo systemctl enable --now kolab

# Configure firewall
sudo firewall-cmd --permanent --add-service=kolab
sudo firewall-cmd --reload

# Verify installation
kolab --version || systemctl status kolab
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install kolab
sudo apt install -y kolab kolab-webadmin, roundcubemail

# Enable and start service
sudo systemctl enable --now kolab

# Configure firewall
sudo ufw allow 443/143/993

# Verify installation
kolab --version || systemctl status kolab
```

### Arch Linux

```bash
# Install kolab
sudo pacman -S kolab

# Enable and start service
sudo systemctl enable --now kolab

# Verify installation
kolab --version || systemctl status kolab
```

### Alpine Linux

```bash
# Install kolab
apk add --no-cache kolab

# Enable and start service
rc-update add kolab default
rc-service kolab start

# Verify installation
kolab --version || rc-service kolab status
```

### openSUSE/SLES

```bash
# Install kolab
sudo zypper install -y kolab kolab-webadmin, roundcubemail

# Enable and start service
sudo systemctl enable --now kolab

# Configure firewall
sudo firewall-cmd --permanent --add-service=kolab
sudo firewall-cmd --reload

# Verify installation
kolab --version || systemctl status kolab
```

### macOS

```bash
# Using Homebrew
brew install kolab

# Start service
brew services start kolab

# Verify installation
kolab --version
```

### FreeBSD

```bash
# Using pkg
pkg install kolab

# Enable in rc.conf
echo 'kolab_enable="YES"' >> /etc/rc.conf

# Start service
service kolab start

# Verify installation
kolab --version || service kolab status
```

### Windows

```powershell
# Using Chocolatey
choco install kolab

# Or using Scoop
scoop install kolab

# Verify installation
kolab --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory if needed
sudo mkdir -p /etc/kolab

# Set up basic configuration
sudo tee /etc/kolab/kolab.conf << 'EOF'
# Kolab Configuration
imap_max_connections = 1000
EOF

# Test configuration
sudo kolab -t || sudo kolab configtest

# Reload service
sudo systemctl reload kolab
```

### Security Hardening

```bash
# Set appropriate permissions
sudo chown -R kolab:kolab /etc/kolab
sudo chmod 750 /etc/kolab

# Enable security features
# See security section for detailed hardening steps
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable kolab

# Start service
sudo systemctl start kolab

# Stop service
sudo systemctl stop kolab

# Restart service
sudo systemctl restart kolab

# Reload configuration
sudo systemctl reload kolab

# Check status
sudo systemctl status kolab

# View logs
sudo journalctl -u kolab -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add kolab default

# Start service
rc-service kolab start

# Stop service
rc-service kolab stop

# Restart service
rc-service kolab restart

# Check status
rc-service kolab status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'kolab_enable="YES"' >> /etc/rc.conf

# Start service
service kolab start

# Stop service
service kolab stop

# Restart service
service kolab restart

# Check status
service kolab status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start kolab
brew services stop kolab
brew services restart kolab

# Check status
brew services list | grep kolab
```

### Windows Service Manager

```powershell
# Start service
net start kolab

# Stop service
net stop kolab

# Using PowerShell
Start-Service kolab
Stop-Service kolab
Restart-Service kolab

# Check status
Get-Service kolab
```

## Advanced Configuration

### Performance Optimization

```bash
# Configure performance settings
cat >> /etc/kolab/kolab.conf << 'EOF'
imap_max_connections = 1000
EOF

# Apply system tuning
sudo sysctl -w net.core.somaxconn=65535
sudo sysctl -w net.ipv4.tcp_max_syn_backlog=65535

# Restart service
sudo systemctl restart kolab
```

### Clustering and High Availability

```bash
# Configure clustering (if supported)
# See official documentation for cluster setup

# Basic load balancing setup example
# Configure multiple instances on different ports
```

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream kolab_backend {
    server 127.0.0.1:443/143/993;
    server 127.0.0.1:{default_port}1 backup;
}

server {
    listen 80;
    server_name kolab.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name kolab.example.com;

    ssl_certificate /etc/ssl/certs/kolab.example.com.crt;
    ssl_certificate_key /etc/ssl/private/kolab.example.com.key;

    location / {
        proxy_pass http://kolab_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # WebSocket support (if needed)
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

### Apache Configuration

```apache
<VirtualHost *:80>
    ServerName kolab.example.com
    Redirect permanent / https://kolab.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName kolab.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/kolab.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/kolab.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:443/143/993/
    ProxyPassReverse / http://127.0.0.1:443/143/993/
    
    # WebSocket support (if needed)
    RewriteEngine on
    RewriteCond %{HTTP:Upgrade} websocket [NC]
    RewriteCond %{HTTP:Connection} upgrade [NC]
    RewriteRule ^/?(.*) "ws://127.0.0.1:443/143/993/$1" [P,L]
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend kolab_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/kolab.pem
    redirect scheme https if !{ ssl_fc }
    default_backend kolab_backend

backend kolab_backend
    balance roundrobin
    option httpchk GET /health
    server kolab1 127.0.0.1:443/143/993 check
    server kolab2 127.0.0.1:{default_port}1 check backup
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R kolab:kolab /etc/kolab
sudo chmod 750 /etc/kolab

# Configure firewall
sudo firewall-cmd --permanent --add-service=kolab
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on

# Configure fail2ban
sudo tee /etc/fail2ban/jail.d/kolab.conf << 'EOF'
[kolab]
enabled = true
port = 443/143/993
filter = kolab
logpath = /var/log/kolab/*.log
maxretry = 5
bantime = 3600
EOF
```

### SSL/TLS Configuration

```bash
# Generate SSL certificates
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /etc/ssl/private/kolab.key \
    -out /etc/ssl/certs/kolab.crt

# Configure SSL in kolab
# See official documentation for SSL configuration
```

## Database Setup

### PostgreSQL Backend (if applicable)

```bash
# Create database and user
sudo -u postgres psql << EOF
CREATE DATABASE kolab_db;
CREATE USER kolab_user WITH ENCRYPTED PASSWORD 'secure_password';
GRANT ALL PRIVILEGES ON DATABASE kolab_db TO kolab_user;
EOF

# Configure kolab to use PostgreSQL
# See official documentation for database configuration
```

### MySQL/MariaDB Backend (if applicable)

```bash
# Create database and user
sudo mysql << EOF
CREATE DATABASE kolab_db;
CREATE USER 'kolab_user'@'localhost' IDENTIFIED BY 'secure_password';
GRANT ALL PRIVILEGES ON kolab_db.* TO 'kolab_user'@'localhost';
FLUSH PRIVILEGES;
EOF
```

## Performance Optimization

### System Tuning

```bash
# Kernel parameters
sudo tee -a /etc/sysctl.conf << EOF
net.core.somaxconn = 65535
net.ipv4.tcp_max_syn_backlog = 65535
net.ipv4.ip_local_port_range = 1024 65535
net.core.netdev_max_backlog = 5000
vm.swappiness = 10
EOF

sudo sysctl -p

# Kolab specific tuning
imap_max_connections = 1000
```

### Resource Limits

```bash
# Configure system limits
sudo tee -a /etc/security/limits.conf << EOF
kolab soft nofile 65535
kolab hard nofile 65535
kolab soft nproc 32768
kolab hard nproc 32768
EOF
```

## Monitoring

### Prometheus Integration

```yaml
# prometheus.yml configuration
scrape_configs:
  - job_name: 'kolab'
    static_configs:
      - targets: ['localhost:443/143/993']
    metrics_path: '/metrics'
```

### Health Checks

```bash
# Basic health check script
#!/bin/bash
if systemctl is-active --quiet kolab; then
    echo "Kolab is running"
    exit 0
else
    echo "Kolab is not running"
    exit 1
fi
```

### Log Monitoring

```bash
# Configure log rotation
sudo tee /etc/logrotate.d/kolab << 'EOF'
/var/log/kolab/*.log {
    daily
    rotate 14
    compress
    delaycompress
    missingok
    notifempty
    create 0640 kolab kolab
    postrotate
        systemctl reload kolab > /dev/null 2>&1 || true
    endscript
}
EOF
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Kolab backup script
BACKUP_DIR="/backup/kolab"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"

# Stop service (if required)
systemctl stop kolab

# Backup configuration
tar -czf "$BACKUP_DIR/kolab-config-$DATE.tar.gz" /etc/kolab

# Backup data (adjust paths as needed)
tar -czf "$BACKUP_DIR/kolab-data-$DATE.tar.gz" /var/lib/kolab

# Start service
systemctl start kolab

# Clean old backups (keep 30 days)
find "$BACKUP_DIR" -name "*.tar.gz" -mtime +30 -delete

echo "Backup completed: $BACKUP_DIR"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop kolab

# Restore configuration
sudo tar -xzf /backup/kolab/kolab-config-*.tar.gz -C /

# Restore data
sudo tar -xzf /backup/kolab/kolab-data-*.tar.gz -C /

# Set permissions
sudo chown -R kolab:kolab /etc/kolab
sudo chown -R kolab:kolab /var/lib/kolab

# Start service
sudo systemctl start kolab
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u kolab -n 100
sudo tail -f /var/log/kolab/*.log

# Check configuration
sudo kolab -t || sudo kolab configtest

# Check permissions
ls -la /etc/kolab
ls -la /var/lib/kolab
```

2. **Connection refused**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 443/143/993
sudo netstat -tlnp | grep 443/143/993

# Check firewall
sudo firewall-cmd --list-all
sudo iptables -L -n

# Test connection
telnet localhost 443/143/993
nc -zv localhost 443/143/993
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep kolabd)
htop -p $(pgrep kolabd)

# Check connections
ss -ant | grep :443/143/993 | wc -l

# Monitor I/O
iotop -p $(pgrep kolabd)
```

### Debug Mode

```bash
# Run in debug mode
sudo kolab -d
# or
sudo kolab debug

# Increase log verbosity
# Edit configuration to enable debug logging
```

## Integration Examples

### Docker Compose

```yaml
version: '3.8'
services:
  kolab:
    image: kolab:latest
    container_name: kolab
    ports:
      - "443/143/993:443/143/993"
    volumes:
      - ./config:/etc/kolab
      - ./data:/var/lib/kolab
    environment:
      - kolab_CONFIG=/etc/kolab/kolab.conf
    restart: unless-stopped
    networks:
      - kolab_net

networks:
  kolab_net:
    driver: bridge
```

### Kubernetes Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kolab
spec:
  replicas: 3
  selector:
    matchLabels:
      app: kolab
  template:
    metadata:
      labels:
        app: kolab
    spec:
      containers:
      - name: kolab
        image: kolab:latest
        ports:
        - containerPort: 443/143/993
        volumeMounts:
        - name: config
          mountPath: /etc/kolab
      volumes:
      - name: config
        configMap:
          name: kolab-config
---
apiVersion: v1
kind: Service
metadata:
  name: kolab
spec:
  selector:
    app: kolab
  ports:
  - port: 443/143/993
    targetPort: 443/143/993
  type: LoadBalancer
```

### Ansible Playbook

```yaml
---
- name: Install and configure Kolab
  hosts: all
  become: yes
  tasks:
    - name: Install kolab
      package:
        name: kolab
        state: present
    
    - name: Configure kolab
      template:
        src: kolab.conf.j2
        dest: /etc/kolab/kolab.conf
        owner: kolab
        group: kolab
        mode: '0640'
      notify: restart kolab
    
    - name: Start and enable kolab
      systemd:
        name: kolab
        state: started
        enabled: yes
  
  handlers:
    - name: restart kolab
      systemd:
        name: kolab
        state: restarted
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update kolab

# Debian/Ubuntu
sudo apt update && sudo apt upgrade kolab

# Arch Linux
sudo pacman -Syu kolab

# Alpine Linux
apk update && apk upgrade kolab

# openSUSE
sudo zypper update kolab

# FreeBSD
pkg update && pkg upgrade kolab

# Always backup before updates
tar -czf /backup/kolab-pre-update-$(date +%Y%m%d).tar.gz /etc/kolab

# Restart after updates
sudo systemctl restart kolab
```

### Regular Maintenance Tasks

```bash
# Clean logs
find /var/log/kolab -name "*.log" -mtime +30 -delete

# Verify integrity
sudo kolab --verify || sudo kolab check

# Update databases (if applicable)
sudo kolab-update-db

# Optimize performance
sudo kolab-optimize

# Check for security updates
sudo kolab --security-check
```

## Additional Resources

- Official Documentation: https://docs.kolab.org/
- GitHub Repository: https://github.com/kolab/kolab
- Community Forum: https://forum.kolab.org/
- Wiki: https://wiki.kolab.org/
- Comparison vs Exchange, Zimbra, Kopano, SOGo: https://docs.kolab.org/comparison

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
