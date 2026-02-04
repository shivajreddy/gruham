# Gruham Implementation Roadmap

This guide provides step-by-step instructions to implement your complete Gruham home server setup.

## Table of Contents
- [Pre-Implementation Checklist](#pre-implementation-checklist)
- [Phase 1: NAS Setup](#phase-1-nas-setup)
- [Phase 2: Beelink Server Setup](#phase-2-beelink-server-setup)
- [Phase 3: Raspberry Pi Setup](#phase-3-raspberry-pi-setup)
- [Phase 4: Deploy Core Services](#phase-4-deploy-core-services)
- [Phase 5: Deploy Media Services](#phase-5-deploy-media-services)
- [Phase 6: Configure Remote Access](#phase-6-configure-remote-access)
- [Phase 7: Deploy Additional Services](#phase-7-deploy-additional-services)
- [Phase 8: Workstation Setup (Claw)](#phase-8-workstation-setup-claw)
- [Phase 9: Testing & Optimization](#phase-9-testing--optimization)
- [Phase 10: Monitoring & Maintenance](#phase-10-monitoring--maintenance)

---

## Pre-Implementation Checklist

### Hardware Requirements
- [ ] Beelink SER3 Mini PC (you have)
- [ ] Raspberry Pi 4 (you have)
- [ ] NAS (purchase: Synology DS423+ or DIY TrueNAS)
- [ ] 10TB HDD (you have)
- [ ] External 4TB USB drive for backups (purchase)
- [ ] Ethernet cables (Cat6, at least 3)
- [ ] Workstation PC (you have)

### Network Requirements
- [ ] Xfinity router access (admin)
- [ ] Eero router access (admin)
- [ ] Domain name (you have)
- [ ] Cloudflare account (free tier)

### Software/Account Requirements
- [ ] Plex account (free)
- [ ] Cloudflare account
- [ ] Anthropic API key (for Claw, optional initially)
- [ ] Password manager (for storing credentials)

### Initial Steps
1. **Generate strong passwords** for all services
2. **Document your network** (take photos of current setup)
3. **Backup any existing data** on the 10TB drive
4. **Create a Gruham notebook** for tracking progress

---

## Phase 1: NAS Setup

**Estimated Time**: 2-3 hours

### 1.1 Purchase NAS Hardware

**Recommended**: Synology DS423+ (~$500)

**Alternative**: DIY TrueNAS build (~$400-600)

See [NAS Setup Guide](NAS-SETUP.md) for detailed hardware recommendations.

### 1.2 Physical Installation

1. **Unbox NAS** and inspect for damage
2. **Install RAM upgrade** (if Synology + optional RAM)
3. **Install 10TB drive** in Bay 1
4. **Connect network**:
   - Ethernet cable from NAS to Eero router
   - Plug in power
   - Power on

### 1.3 Initial Software Configuration

#### For Synology DS423+:

1. **Find NAS**: Visit http://find.synology.com
2. **Install DSM**: Follow prompts to install DiskStation Manager
3. **Create admin account**:
   - Username: admin
   - Password: [Generate strong password, save in password manager]
4. **Configure storage**:
   - Storage Manager → Create Storage Pool
   - Type: Basic (single drive, no RAID yet)
   - File system: Btrfs
   - Allocate all space
5. **Create shared folders**:
   ```
   /media      (for Plex/Jellyfin)
   /backups    (for server backups)
   /shares     (for general files)
   ```
6. **Enable file services**:
   - SMB/CIFS: Enabled
   - NFS: Enabled (for better Plex performance)

#### For TrueNAS:

See [NAS Setup Guide - TrueNAS Section](NAS-SETUP.md#truenas-scale-setup)

### 1.4 Configure Static IP

**Eero App**:
1. Open Eero app
2. Network Settings → Reservations & Port Forwarding
3. Add Reservation:
   - Device: [Your NAS]
   - IP: `192.168.4.102`

**Verify**: 
```bash
ping 192.168.4.102
# Should resolve to NAS
```

### 1.5 Organize Media Library

Create folder structure:
```
/volume1/media/
├── movies/
├── tv/
├── music/
└── photos/
```

See [NAS Setup Guide - Media Organization](NAS-SETUP.md#organizing-media-library) for details.

**✅ Phase 1 Complete** - NAS is accessible at `192.168.4.102`

---

## Phase 2: Beelink Server Setup

**Estimated Time**: 2-3 hours

### 2.1 Install Ubuntu Server

1. **Download Ubuntu Server 24.04 LTS**:
   - https://ubuntu.com/download/server
   - Create bootable USB with Rufus (Windows) or Balena Etcher

2. **Boot from USB**:
   - Insert USB into Beelink
   - Power on, press F7/F11/Del for boot menu
   - Select USB drive

3. **Install Ubuntu**:
   - Language: English
   - Keyboard: Your layout
   - Network: Configure ethernet (DHCP for now)
   - Guided storage: Use entire disk (500GB NVMe)
   - Profile setup:
     - Your name: [your name]
     - Server name: `beelink-gruham`
     - Username: `gruham` (or your preference)
     - Password: [Generate strong password]
   - SSH: Install OpenSSH server ✓
   - Featured Server Snaps: Skip for now
   - Install and reboot

4. **Remove USB drive** when prompted

### 2.2 Initial Server Configuration

SSH into Beelink from your workstation:

```bash
# Find Beelink IP (check Eero app or router)
ssh gruham@192.168.4.XXX  # Temporary DHCP IP

# Update system
sudo apt update && sudo apt upgrade -y

# Install essential tools
sudo apt install -y \
    curl wget git vim nano \
    htop btop ncdu \
    net-tools dnsutils \
    nfs-common cifs-utils

# Set timezone
sudo timedatectl set-timezone America/New_York  # Adjust to your timezone

# Verify
timedatectl
```

### 2.3 Configure Static IP

```bash
# Find current network interface name
ip a
# Look for interface like "enp0s1", "eth0", or "ens18"

# Edit netplan configuration
sudo nano /etc/netplan/00-installer-config.yaml
```

**Content** (adjust interface name):
```yaml
network:
  version: 2
  ethernets:
    enp0s1:  # Replace with your interface name
      dhcp4: no
      addresses:
        - 192.168.4.100/24
      routes:
        - to: default
          via: 192.168.4.1  # Eero router
      nameservers:
        addresses:
          - 192.168.4.101  # Pi-hole (will configure later)
          - 1.1.1.1        # Cloudflare (backup)
```

**Apply**:
```bash
sudo netplan apply

# Verify
ip a
# Should show 192.168.4.100

# Test connectivity
ping 8.8.8.8
ping google.com
```

**Reconnect SSH** using new IP:
```bash
exit
ssh gruham@192.168.4.100
```

### 2.4 Install Docker

```bash
# Remove old versions (if any)
sudo apt remove docker docker-engine docker.io containerd runc

# Install Docker (official script)
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Add user to docker group
sudo usermod -aG docker $USER

# Start and enable Docker
sudo systemctl enable docker
sudo systemctl start docker

# Logout and login for group changes
exit
ssh gruham@192.168.4.100

# Verify Docker installation
docker --version
docker compose version
docker ps
```

### 2.5 Create Gruham Directory Structure

```bash
# Create main directory
sudo mkdir -p /opt/gruham

# Set ownership
sudo chown -R $USER:$USER /opt/gruham

# Create subdirectories
cd /opt/gruham
mkdir -p \
    nginx-proxy-manager/{data,letsencrypt} \
    portainer/data \
    cloudflared \
    plex/{config,transcode} \
    jellyfin/{config,cache} \
    homeassistant/config \
    tautulli/config \
    overseerr/config \
    homepage/config \
    filebrowser \
    duplicati/{config,backups} \
    mosquitto/{config,data,log} \
    nodered/data

# Create NAS mount points
sudo mkdir -p /mnt/nas/{media,backups,shares}

# Verify structure
tree -L 2 /opt/gruham
```

### 2.6 Mount NAS Storage

```bash
# Test NFS mount manually first
sudo mount -t nfs 192.168.4.102:/volume1/media /mnt/nas/media

# If that works, verify
ls /mnt/nas/media
# Should see: movies/ tv/ music/ photos/

# Unmount
sudo umount /mnt/nas/media

# Configure automatic mounting
sudo nano /etc/fstab
```

**Add to /etc/fstab** (for Synology):
```
# NAS Mounts
192.168.4.102:/volume1/media    /mnt/nas/media    nfs    defaults,_netdev,rw,hard,intr    0 0
192.168.4.102:/volume1/backups  /mnt/nas/backups  nfs    defaults,_netdev,rw,hard,intr    0 0
192.168.4.102:/volume1/shares   /mnt/nas/shares   nfs    defaults,_netdev,rw,hard,intr    0 0
```

**Mount all**:
```bash
sudo mount -a

# Verify
df -h | grep nas
ls /mnt/nas/media
```

### 2.7 Configure Firewall (UFW)

```bash
# Install UFW (if not installed)
sudo apt install ufw -y

# Default policies
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH (from LAN only)
sudo ufw allow from 192.168.4.0/24 to any port 22

# Allow HTTP/HTTPS (from LAN)
sudo ufw allow from 192.168.4.0/24 to any port 80
sudo ufw allow from 192.168.4.0/24 to any port 443

# Allow Plex
sudo ufw allow from 192.168.4.0/24 to any port 32400

# Enable firewall
sudo ufw enable

# Verify
sudo ufw status verbose
```

**✅ Phase 2 Complete** - Beelink is ready for Docker containers

---

## Phase 3: Raspberry Pi Setup

**Estimated Time**: 1-2 hours

### 3.1 Install Raspberry Pi OS

1. **Download Raspberry Pi Imager**:
   - https://www.raspberrypi.com/software/

2. **Flash OS**:
   - Insert microSD card (32GB+)
   - Open Imager
   - OS: Raspberry Pi OS Lite (64-bit)
   - Storage: Select your microSD
   - Settings (gear icon):
     - Hostname: `raspberry-pihole`
     - Enable SSH ✓
     - Username: `pi`
     - Password: [Generate strong password]
     - WiFi: Configure (or use Ethernet)
     - Timezone: Your timezone
   - Write

3. **Boot Pi**:
   - Insert microSD into Pi
   - Connect Ethernet cable (recommended for DNS reliability)
   - Power on

### 3.2 Initial Configuration

```bash
# SSH into Pi (find IP in Eero app)
ssh pi@192.168.4.XXX

# Update system
sudo apt update && sudo apt upgrade -y

# Install Docker
curl -fsSL https://get.docker.com | sudo sh

# Add pi user to docker group
sudo usermod -aG docker pi

# Logout and login
exit
ssh pi@192.168.4.XXX

# Verify
docker --version
```

### 3.3 Configure Static IP

```bash
# Edit dhcpcd.conf
sudo nano /etc/dhcpcd.conf
```

**Add to end of file**:
```
# Static IP for Pi-hole
interface eth0
static ip_address=192.168.4.101/24
static routers=192.168.4.1
static domain_name_servers=1.1.1.1 1.0.0.1
```

**Reboot**:
```bash
sudo reboot
```

**Reconnect**:
```bash
ssh pi@192.168.4.101
```

### 3.4 Create Directory Structure

```bash
# Create Gruham directory
mkdir -p ~/gruham

cd ~/gruham
mkdir -p \
    pihole/{etc-pihole,etc-dnsmasq.d} \
    uptime-kuma/data \
    unbound/config
```

### 3.5 Create Environment File

```bash
cd ~/gruham
nano .env
```

**Content**:
```env
# Raspberry Pi Environment
TZ=America/New_York
PIHOLE_PASSWORD=YOUR_STRONG_PASSWORD_HERE
BEELINK_IP=192.168.4.100
LOCAL_NETWORK=192.168.4.0/24
DOCKER_DATA=/home/pi/gruham
```

**✅ Phase 3 Complete** - Raspberry Pi is ready

---

## Phase 4: Deploy Core Services

**Estimated Time**: 2-3 hours

### 4.1 Clone Gruham Configs to Beelink

```bash
# On Beelink
cd /opt/gruham

# Download configs from your repo (or copy manually)
# For now, create .env file:
nano .env
```

**Copy contents from** `configs/.env.example` and fill in your values.

Critical variables to set:
```env
TZ=America/New_York
PUID=1000
PGID=1000
GRUHAM_ROOT=/opt/gruham
NAS_MEDIA=/mnt/nas/media
NAS_BACKUPS=/mnt/nas/backups
DOMAIN_NAME=yourdomain.com
# ... etc (see .env.example)
```

### 4.2 Deploy Core Services Stack

```bash
cd /opt/gruham

# Copy core services docker-compose.yml
# (from configs/beelink/core-services/docker-compose.yml)

# Create the network first
docker network create gruham_network

# Start core services
docker compose -f core-services/docker-compose.yml up -d

# Check status
docker ps

# Check logs
docker compose -f core-services/docker-compose.yml logs -f
```

**Expected containers**:
- nginx-proxy-manager
- portainer
- cloudflared (will fail until tunnel configured)
- watchtower

### 4.3 Configure Nginx Proxy Manager

1. **Access NPM**: http://192.168.4.100:81
2. **Default login**:
   - Email: `admin@example.com`
   - Password: `changeme`
3. **Change password immediately**
4. **Update email** to your actual email

**You'll configure proxy hosts after setting up services**

### 4.4 Access Portainer

1. **Navigate to**: https://192.168.4.100:9443
2. **Create admin account**:
   - Username: admin
   - Password: [Generate strong password]
3. **Connect to local Docker environment**

**✅ Phase 4 Complete** - Core infrastructure running

---

## Phase 5: Deploy Media Services

**Estimated Time**: 2-3 hours

### 5.1 Get Plex Claim Token

1. Visit: https://www.plex.tv/claim/
2. Copy claim token (valid for 4 minutes)
3. Add to `/opt/gruham/.env`:
   ```env
   PLEX_CLAIM=claim-XXXXXXXXXXXXXXXXXXXX
   ```

### 5.2 Deploy Media Stack

```bash
cd /opt/gruham

# Start media services
docker compose -f media-services/docker-compose.yml up -d

# Check logs
docker compose -f media-services/docker-compose.yml logs -f plex
docker compose -f media-services/docker-compose.yml logs -f jellyfin
```

### 5.3 Configure Plex

1. **Access Plex**: http://192.168.4.100:32400/web
2. **Sign in** with your Plex account
3. **Name your server**: "Gruham Plex"
4. **Add library**:
   - Type: Movies
   - Folder: `/media/movies`
   - Add
5. **Repeat for**:
   - TV Shows: `/media/tv`
   - Music: `/media/music`
6. **Settings → Transcoder**:
   - Hardware acceleration: Use hardware acceleration (AMD)
   - Transcoder temporary directory: `/transcode`

### 5.4 Configure Jellyfin

1. **Access Jellyfin**: http://192.168.4.100:8096
2. **Setup wizard**:
   - Language: English
   - Create admin account
3. **Add libraries**:
   - Movies: `/media/movies`
   - TV Shows: `/media/tv`
   - Music: `/media/music`
4. **Settings → Playback → Transcoding**:
   - Hardware acceleration: Video Acceleration API (VAAPI)
   - VA-API Device: `/dev/dri/renderD128`

### 5.5 Configure Tautulli

1. **Access**: http://192.168.4.100:8181
2. **Plex server**: http://192.168.4.100:32400
3. **Authenticate** with Plex
4. **Configure notifications** (optional)

### 5.6 Configure Overseerr

1. **Access**: http://192.168.4.100:5055
2. **Sign in with Plex**
3. **Connect Plex server**
4. **Configure** (optional, for media requests)

**✅ Phase 5 Complete** - Media services operational

---

## Phase 6: Configure Remote Access

**Estimated Time**: 2-3 hours

### 6.1 Set Up Cloudflare Account

1. **Sign up** at https://cloudflare.com
2. **Add your domain**:
   - Follow prompts to add domain
   - Update nameservers at your registrar to Cloudflare's
   - Wait for DNS propagation (can take 24hrs)

### 6.2 Create Cloudflare Tunnel

1. **Go to** Zero Trust dashboard: https://one.dash.cloudflare.com
2. **Networks → Tunnels → Create tunnel**
3. **Name**: `gruham-tunnel`
4. **Copy token** (starts with `ey...`)
5. **Add to `.env`**:
   ```env
   TUNNEL_TOKEN=your_token_here
   ```

### 6.3 Restart Cloudflared

```bash
cd /opt/gruham

# Restart with tunnel token
docker compose -f core-services/docker-compose.yml restart cloudflared

# Verify logs
docker logs cloudflared
# Should show "Registered tunnel connection"
```

### 6.4 Configure Tunnel Routes

**In Cloudflare Zero Trust → Tunnels → gruham-tunnel → Public Hostnames**:

Add these hostnames:

| Subdomain | Domain | Service | 
|-----------|--------|---------|
| plex | yourdomain.com | http://192.168.4.100:32400 |
| jellyfin | yourdomain.com | http://192.168.4.100:8096 |
| home | yourdomain.com | http://192.168.4.100:8123 |
| dash | yourdomain.com | http://192.168.4.100:3000 |
| portainer | yourdomain.com | https://192.168.4.100:9443 |
| nas | yourdomain.com | http://192.168.4.102:5000 |

**Note**: Use `https://` for Portainer (it uses SSL)

### 6.5 Configure Nginx Proxy Manager (Optional, for local SSL)

**Nginx Proxy Manager** can provide SSL for local access.

1. **NPM UI**: http://192.168.4.100:81
2. **Hosts → Proxy Hosts → Add Proxy Host**:

**Example for Plex**:
- Domain: `plex.local` (or `plex.yourdomain.com`)
- Scheme: `http`
- Forward hostname: `192.168.4.100`
- Forward port: `32400`
- SSL tab: Request SSL certificate (Let's Encrypt)

Repeat for other services.

### 6.6 Test Remote Access

**From phone (disable WiFi, use cellular)**:

1. Visit `https://plex.yourdomain.com` - Should see Plex
2. Visit `https://home.yourdomain.com` - Should see Home Assistant
3. Visit `https://dash.yourdomain.com` - Should see dashboard

**✅ Phase 6 Complete** - Services accessible remotely!

---

## Phase 7: Deploy Additional Services

**Estimated Time**: 2-4 hours

### 7.1 Deploy Raspberry Pi Services

```bash
# SSH to Pi
ssh pi@192.168.4.101

cd ~/gruham

# Copy docker-compose.yml for Pi
# (from configs/raspberry-pi/docker-compose.yml)

# Start services
docker compose up -d

# Check logs
docker compose logs -f pihole
docker compose logs -f uptime-kuma
```

### 7.2 Configure Pi-hole

1. **Access**: http://192.168.4.101/admin
2. **Login** with password from `.env`
3. **Settings → DNS**:
   - Upstream DNS: Cloudflare (1.1.1.1, 1.0.0.1)
   - Or Unbound (if deployed): 127.0.0.1#5335
4. **Local DNS Records**:
   - Add custom DNS entries:
     ```
     192.168.4.100  plex.local
     192.168.4.100  jellyfin.local
     192.168.4.100  home.local
     192.168.4.102  nas.local
     192.168.4.101  pihole.local
     ```

### 7.3 Configure Network to Use Pi-hole

**Eero App**:
1. Settings → Network Settings → DNS
2. Custom DNS:
   - Primary: `192.168.4.101` (Pi-hole)
   - Secondary: `1.1.1.1` (Cloudflare backup)

**All devices** will now use Pi-hole for DNS (ad blocking enabled!)

### 7.4 Configure Uptime Kuma

1. **Access**: http://192.168.4.101:3001
2. **Create admin account**
3. **Add monitors** for all services:
   - Plex: http://192.168.4.100:32400/web
   - Jellyfin: http://192.168.4.100:8096
   - Home Assistant: http://192.168.4.100:8123
   - NAS: http://192.168.4.102
   - etc.

### 7.5 Deploy Home Assistant

```bash
# On Beelink
cd /opt/gruham

# Start Home Assistant
docker compose -f automation/docker-compose.yml up -d

# Check logs
docker logs homeassistant -f
```

**Configure**:
1. **Access**: http://192.168.4.100:8123
2. **Create account**
3. **Setup location** and timezone
4. **Add integrations** (Plex, smart devices, etc.)

### 7.6 Deploy Management Services

```bash
cd /opt/gruham

# Start management stack
docker compose -f management/docker-compose.yml up -d

# Check
docker ps
```

**Configure Homepage**:
1. **Access**: http://192.168.4.100:3000
2. **Edit config** at `/opt/gruham/homepage/config/`

**Configure Filebrowser**:
1. **Access**: http://192.168.4.100:8080
2. **Default**: admin / admin
3. **Change password immediately**

**Configure Duplicati**:
1. **Access**: http://192.168.4.100:8200
2. **Add backup**:
   - Source: `/source/docker-configs`
   - Destination: `/backups/nas` (NAS mount)
   - Encryption: AES-256
   - Schedule: Daily at 2 AM

**✅ Phase 7 Complete** - All services deployed!

---

## Phase 8: Workstation Setup (Claw)

**Estimated Time**: 1-2 hours

See [Claw Setup Guide](../configs/workstation/claw-setup.md) for detailed instructions.

**Summary**:
1. Install Python 3.12 on Windows
2. Clone Claw repository
3. Configure with Anthropic API key
4. Add Gruham service integrations
5. Expose via Cloudflare Tunnel
6. Set up auto-start

**✅ Phase 8 Complete** - Claw running on workstation

---

## Phase 9: Testing & Optimization

**Estimated Time**: 2-3 hours

### 9.1 Test Media Streaming

**Local**:
- [ ] Play movie in Plex (direct play)
- [ ] Play movie in Jellyfin
- [ ] Test transcoding (incompatible format)
- [ ] Test on mobile device (Plex app)

**Remote**:
- [ ] Access Plex via yourdomain.com (cellular)
- [ ] Stream content remotely
- [ ] Test download for offline

### 9.2 Test Home Automation

- [ ] Control smart device via Home Assistant
- [ ] Create simple automation
- [ ] Test mobile app access
- [ ] Verify remote access

### 9.3 Test Backups

```bash
# On Beelink
cd /opt/gruham

# Create test backup with Duplicati
# Verify backup appears in /mnt/nas/backups

# Test restore (restore a small file)
```

### 9.4 Test Monitoring

- [ ] Verify all monitors green in Uptime Kuma
- [ ] Check Pi-hole blocking ads (visit news site)
- [ ] Review Docker stats in Portainer

### 9.5 Performance Optimization

**Check resource usage**:
```bash
# On Beelink
htop  # Overall system
docker stats  # Per-container

# On Pi
htop
docker stats
```

**Optimize if needed**:
- Adjust container CPU/memory limits
- Enable hardware transcoding (Plex/Jellyfin)
- Optimize NFS mount settings

### 9.6 Security Hardening

- [ ] All services use strong passwords
- [ ] 2FA enabled (Plex, Cloudflare, HA)
- [ ] Firewall rules verified
- [ ] No unnecessary ports exposed
- [ ] SSL certificates active

**✅ Phase 9 Complete** - Everything tested and optimized!

---

## Phase 10: Monitoring & Maintenance

### 10.1 Daily

**Automated** (no action needed):
- Watchtower checks for updates
- Pi-hole updates blocklists
- NAS snapshots
- Plex metadata updates

### 10.2 Weekly

- [ ] Check Uptime Kuma (all services up?)
- [ ] Review Pi-hole stats (ads blocked)
- [ ] Check disk space (NAS, Beelink)
- [ ] Review Plex activity (Tautulli)

### 10.3 Monthly

- [ ] Review and test backups
- [ ] Update OS packages (Beelink, Pi)
- [ ] Clean old media (if needed)
- [ ] Review logs for errors
- [ ] External backup to USB drive

### 10.4 Quarterly

- [ ] Test disaster recovery
- [ ] Review security settings
- [ ] Evaluate new services to add
- [ ] Update documentation
- [ ] Clean unused Docker images

---

## Troubleshooting

### Common Issues

**Service won't start**:
```bash
docker logs <container-name>
docker compose -f <stack>.yml restart <service>
```

**Can't access service locally**:
- Check firewall: `sudo ufw status`
- Verify service running: `docker ps`
- Check network: `ping 192.168.4.100`

**Can't access remotely**:
- Check Cloudflare Tunnel: `docker logs cloudflared`
- Verify DNS: `nslookup plex.yourdomain.com`
- Check Zero Trust dashboard for tunnel status

**NAS not mounted**:
```bash
# Check NAS is up
ping 192.168.4.102

# Check NFS exports
showmount -e 192.168.4.102

# Remount
sudo umount /mnt/nas/media
sudo mount -a
```

**Plex transcoding issues**:
- Enable hardware acceleration
- Check `/transcode` has space
- Verify AMD GPU available: `ls /dev/dri`

---

## Next Steps

After full implementation:

1. **Use your system!**
   - Add media to Plex/Jellyfin
   - Set up smart home automations
   - Explore new services

2. **Learn and optimize**
   - Review logs and stats
   - Fine-tune performance
   - Add more automation

3. **Plan expansion**
   - Add more drives to NAS (redundancy!)
   - Consider K3s migration
   - Add more services as needed

4. **Share knowledge**
   - Document your learnings
   - Help others in homelab community

---

## Congratulations!

You now have a fully operational home server with:
- ✅ Media streaming (Plex + Jellyfin)
- ✅ Home automation (Home Assistant)
- ✅ Network-wide ad blocking (Pi-hole)
- ✅ Secure remote access (Cloudflare Tunnel)
- ✅ Automated backups
- ✅ Monitoring and management
- ✅ AI assistant (Claw)
- ✅ 10TB storage (expandable to 50TB)

**Enjoy your Gruham home server!** 🏡🚀
