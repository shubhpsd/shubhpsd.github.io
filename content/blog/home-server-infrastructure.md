---
title: "Building a Self-Hosted Home Server Infrastructure"
date: 2025-05-23
draft: false
description:
  "Setting up a complete self-hosted infrastructure with media streaming, cloud
  storage, and global access using Docker and Tailscale"
tags:
  [
    "self-hosting",
    "docker",
    "linux",
    "infrastructure",
    "jellyfin",
    "nextcloud",
    "immich",
    "tailscale",
  ]
---

In an era where data privacy and ownership are becoming increasingly important,
I decided to build my own self-hosted infrastructure. This project resulted in a
complete home server setup using a repurposed Dell G5 5587 gaming laptop with
2TB RAID storage, featuring multiple services accessible globally through
Tailscale.

## Why Self-Host?

**Data Ownership**: Complete control over my personal data and media
**Privacy**: No third-party services scanning or monetizing my content  
**Cost Savings**: Eliminate monthly subscriptions for cloud services **Learning
Experience**: Deep dive into infrastructure, networking, and system
administration **Customization**: Tailor services exactly to my needs

## Hardware Setup

I repurposed a gaming laptop into a capable home server, providing excellent
performance while saving on dedicated server hardware costs.

### Server Specifications

- **Device**: Repurposed Dell G5 5587 gaming laptop
- **CPU**: Intel i7-8750H (6 cores, 12 threads)
- **GPU**: Nvidia GTX 1050ti (used for hardware transcoding)
- **RAM**: 16GB DDR4
- **Storage**: 2TB RAID 1 configuration for redundancy
- **OS**: Fedora Server Edition
- **Network**: Gigabit Ethernet with static IP
- **Power Management**: Custom cooling stand with additional fans

### RAID Configuration

```bash
# Setting up RAID 1 for data redundancy
sudo mdadm --create --verbose /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc

# Create filesystem
sudo mkfs.ext4 /dev/md0

# Mount and configure auto-mount
sudo mkdir /mnt/raid1
echo '/dev/md0 /mnt/raid1 ext4 defaults,nofail,discard 0 0' | sudo tee -a /etc/fstab
```

## Service Architecture

### Docker-First Approach

All services run in Docker containers for easy management, updates, and
isolation:

```yaml
version: "3.8"

services:
  jellyfin:
    image: jellyfin/jellyfin:latest
    container_name: jellyfin
    volumes:
      - /mnt/raid1/jellyfin/config:/config
      - /mnt/raid1/jellyfin/cache:/cache
      - /mnt/raid1/media:/media
    ports:
      - "8096:8096"
    restart: unless-stopped

  nextcloud:
    image: nextcloud:latest
    container_name: nextcloud
    volumes:
      - /mnt/raid1/nextcloud:/var/www/html
    environment:
      - MYSQL_PASSWORD=${MYSQL_PASSWORD}
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
      - MYSQL_HOST=db
    ports:
      - "8080:80"
    restart: unless-stopped

  immich-server:
    image: ghcr.io/immich-app/immich-server:release
    container_name: immich_server
    volumes:
      - /mnt/raid1/immich/upload:/usr/src/app/upload
    environment:
      - DB_HOSTNAME=immich-postgres
      - DB_USERNAME=postgres
      - DB_PASSWORD=${DB_PASSWORD}
      - DB_DATABASE_NAME=immich
    ports:
      - "2283:3001"
    restart: unless-stopped
```

## Core Services

### 1. Jellyfin - Media Streaming

**Purpose**: Netflix-like interface for personal media collection **Features**:

- Hardware transcoding for multiple device support
- Mobile apps for iOS/Android
- User management with parental controls
- Subtitle support and metadata fetching

```bash
# Hardware transcoding setup for Intel QuickSync
sudo usermod -a -G render jellyfin
sudo chown -R jellyfin:jellyfin /dev/dri
```

### 2. Nextcloud - Cloud Storage

**Purpose**: Google Drive/iCloud replacement **Features**:

- File synchronization across devices
- Calendar and contacts sync
- Document collaboration
- Photo backup from mobile devices

**Custom Configuration**:

```php
// config.php customizations
'memcache.local' => '\OC\Memcache\APCu',
'memcache.redis' => [
    'host' => 'redis',
    'port' => 6379,
],
'default_phone_region' => 'IN',
'trusted_domains' => [
    'nextcloud.local',
    '192.168.1.100',
],
```

### 3. Immich - Photo Management

**Purpose**: Google Photos alternative with AI features **Features**:

- Automatic photo backup from mobile
- Face recognition and object detection
- Timeline view and smart albums
- Shared albums for family

### 4. Additional Services

**Portainer**: Docker container management GUI

```yaml
portainer:
  image: portainer/portainer-ce:latest
  container_name: portainer
  volumes:
    - /var/run/docker.sock:/var/run/docker.sock
    - portainer_data:/data
  ports:
    - "9000:9000"
```

**Nginx Proxy Manager**: Reverse proxy with SSL

```yaml
nginx-proxy-manager:
  image: jc21/nginx-proxy-manager:latest
  container_name: nginx-proxy-manager
  ports:
    - "80:80"
    - "443:443"
    - "81:81"
  volumes:
    - ./data:/data
    - ./letsencrypt:/etc/letsencrypt
```

## Global Access with Tailscale

### Why Tailscale?

- **Zero-config VPN**: No complex networking setup
- **End-to-end encryption**: Secure access from anywhere
- **Multi-platform**: Works on all devices
- **No port forwarding**: Bypass router configurations

### Setup Process

```bash
# Install Tailscale on Ubuntu Server
curl -fsSL https://tailscale.com/install.sh | sh

# Authenticate and connect
sudo tailscale up

# Enable subnet routing for local network access
sudo tailscale up --advertise-routes=192.168.1.0/24
```

### Device Configuration

Each device (laptop, phone, tablet) connects to the Tailscale network, providing
seamless access to all services using their Tailscale IPs.

## Monitoring and Maintenance

### System Monitoring

```yaml
prometheus:
  image: prom/prometheus:latest
  container_name: prometheus
  volumes:
    - ./prometheus.yml:/etc/prometheus/prometheus.yml
  ports:
    - "9090:9090"

grafana:
  image: grafana/grafana:latest
  container_name: grafana
  volumes:
    - grafana-storage:/var/lib/grafana
  ports:
    - "3000:3000"
```

### Automated Backups

```bash
#!/bin/bash
# Daily backup script
DATE=$(date +%Y%m%d)
BACKUP_DIR="/mnt/raid1/backups"

# Backup Docker volumes
docker run --rm -v nextcloud_data:/data -v $BACKUP_DIR:/backup ubuntu tar czf /backup/nextcloud_$DATE.tar.gz -C /data .

# Backup databases
docker exec mysql mysqldump -u root -p$MYSQL_ROOT_PASSWORD --all-databases | gzip > $BACKUP_DIR/mysql_$DATE.sql.gz

# Clean old backups (keep last 7 days)
find $BACKUP_DIR -name "*.tar.gz" -mtime +7 -delete
```

## Security Measures

### 1. Firewall Configuration

```bash
# UFW rules for external access
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw allow 41641/udp  # Tailscale
sudo ufw enable
```

### 2. SSL Certificates

All services secured with Let's Encrypt certificates via Nginx Proxy Manager:

- `jellyfin.mydomain.com`
- `nextcloud.mydomain.com`
- `immich.mydomain.com`

### 3. Regular Updates

```bash
#!/bin/bash
# Update script
docker-compose pull
docker-compose up -d
docker system prune -f
sudo apt update && sudo apt upgrade -y
```

## Performance Optimizations

### 1. SSD Caching

```bash
# Setup bcache for HDD acceleration with SSD
sudo make-bcache -B /dev/sdb -C /dev/nvme0n1p1
```

### 2. Network Optimization

```bash
# Optimize network settings for media streaming
echo 'net.core.rmem_max = 16777216' | sudo tee -a /etc/sysctl.conf
echo 'net.core.wmem_max = 16777216' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

## Results and Benefits

**Hardware Efficiency**: Repurposed laptop serves as a fully functional server
**Storage Capacity**: 2TB usable space with RAID 1 redundancy (1TB effective
capacity) **Service Uptime**: 99.8% availability over 6 months **Global
Access**: Seamless connectivity from 15+ countries via Tailscale **Cost
Savings**: $200+/year saved on cloud subscriptions, plus ~$1000 saved by
repurposing existing hardware **Performance**: Local network speeds up to 1Gbps
**Power Consumption**: Lower than a traditional server (45-65W under load)

## Lessons Learned

### Technical Insights

- **Laptops make great servers**: Lower power consumption, built-in UPS
  (battery), and compact form factor
- **Cooling matters**: Added external cooling solution to manage heat in 24/7
  operation
- **RAID is essential**: Had one drive failure with zero data loss
- **Docker simplifies management**: Easy updates and rollbacks
- **Monitoring is crucial**: Caught issues before they became problems
- **Automation saves time**: Backup and update scripts are invaluable

### Infrastructure Design

- **Start simple**: Begin with core services, expand gradually
- **Document everything**: Configuration notes saved countless hours
- **Security first**: VPN access is much safer than port forwarding
- **Regular maintenance**: Scheduled updates prevent security issues

## Future Expansions

**Planned Additions**:

- **Home automation**: Home Assistant integration
- **Git hosting**: Self-hosted GitLab or Gitea
- **Email server**: Complete email independence
- **NAS expansion**: Additional storage as needs grow

**Infrastructure Improvements**:

- **Load balancing**: High availability for critical services
- **Backup strategies**: Off-site backup rotation
- **Power management**: UPS integration for clean shutdowns

## Impact on Daily Life

This self-hosted infrastructure has transformed how I handle digital content:

- **Media streaming**: 4K movies anywhere in the world
- **Photo management**: Automatic backup and AI organization
- **File access**: All documents available on any device
- **Privacy**: Complete control over personal data

## Technical Skills Developed

**System Administration**: Linux server management, networking, storage
**Containerization**: Docker expertise, service orchestration **Security**: VPN
setup, SSL certificates, firewall management **Monitoring**: Prometheus/Grafana,
log analysis **Automation**: Bash scripting, cron jobs, backup strategies

## Cost Analysis

**Initial Investment**: ~$200 (mostly for external drives, as laptop was
repurposed) **Hardware Savings**: ~$600 (cost avoided by not purchasing a
dedicated server) **Monthly Costs**: ~$10 (electricity - laptop is more power
efficient than a traditional server) **Annual Savings**: ~$200 (vs cloud
subscriptions) **ROI Timeline**: ~1 year (much faster due to repurposing
existing hardware)

This project demonstrates that self-hosting is not just feasible but highly
beneficial for anyone seeking data ownership, privacy, and learning
opportunities in infrastructure management.

---

**Key Technologies**: Docker, Ubuntu Server, Tailscale, Jellyfin, Nextcloud,
Immich, RAID, Nginx
