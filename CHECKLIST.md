# Gruham Implementation Checklist

Use this checklist to track your progress as you build your Gruham home server.

## Pre-Implementation

### Hardware Preparation
- [ ] Beelink SER3 Mini PC (owned ✓)
- [ ] Raspberry Pi 4 (owned ✓)
- [ ] 10TB HDD (owned ✓)
- [ ] Workstation PC (owned ✓)
- [ ] Domain name (owned ✓)
- [ ] **Purchase: NAS enclosure** (Synology DS423+ or DIY TrueNAS) - ~$400-600
- [ ] **Purchase: External 4TB USB drive** for backups - ~$100
- [ ] **Purchase: 3x Ethernet cables** (Cat6 recommended)

### Software/Accounts
- [ ] Create Cloudflare account (free)
- [ ] Create Plex account (free)
- [ ] Download Ubuntu Server 24.04 LTS ISO
- [ ] Download Raspberry Pi Imager
- [ ] Install password manager (for storing credentials)
- [ ] Generate strong passwords for all services
- [ ] Optional: Create Anthropic account for Claw (API key)

### Documentation
- [ ] Read [PROJECT-OVERVIEW.md](PROJECT-OVERVIEW.md)
- [ ] Read [QUICKSTART.md](QUICKSTART.md)
- [ ] Review [ARCHITECTURE.md](docs/ARCHITECTURE.md)
- [ ] Review [NAS-SETUP.md](docs/NAS-SETUP.md)
- [ ] Open [network diagram](diagrams/network-architecture.excalidraw) in Excalidraw

---

## Week 1: Foundation

### Day 1-2: NAS Setup
- [ ] Receive NAS hardware
- [ ] Install 10TB drive in Bay 1
- [ ] Connect NAS to Eero router via Ethernet
- [ ] Power on NAS
- [ ] Access NAS web interface (find.synology.com or IP)
- [ ] Install DSM (Synology) or TrueNAS SCALE
- [ ] Create admin account (save password!)
- [ ] Configure static IP: 192.168.4.102
- [ ] Create storage pool (single drive, Basic)
- [ ] Create shared folders: `/media`, `/backups`, `/shares`
- [ ] Enable SMB/CIFS file sharing
- [ ] Enable NFS file sharing
- [ ] Configure NFS permissions for Beelink (192.168.4.100)
- [ ] Test access from workstation
- [ ] Set up snapshot schedule (hourly/daily/weekly)

### Day 3-4: Beelink Server Setup
- [ ] Create Ubuntu Server 24.04 bootable USB
- [ ] Boot Beelink from USB
- [ ] Install Ubuntu Server
  - [ ] Hostname: `beelink-gruham`
  - [ ] Username: `gruham` (or your choice)
  - [ ] Enable SSH server
- [ ] Reboot and remove USB
- [ ] SSH into Beelink
- [ ] Update system: `sudo apt update && sudo apt upgrade -y`
- [ ] Install essential tools
- [ ] Set timezone
- [ ] Configure static IP: 192.168.4.100
- [ ] Test connectivity (ping google.com, ping NAS)
- [ ] Install Docker
- [ ] Add user to docker group
- [ ] Verify Docker installation
- [ ] Create `/opt/gruham` directory structure
- [ ] Create NAS mount points: `/mnt/nas/{media,backups,shares}`
- [ ] Configure NFS mounts in `/etc/fstab`
- [ ] Mount NAS shares: `sudo mount -a`
- [ ] Verify NAS mounts: `df -h | grep nas`
- [ ] Configure UFW firewall
- [ ] Test SSH, Docker, NAS access

### Day 5-6: Raspberry Pi Setup
- [ ] Flash Raspberry Pi OS Lite (64-bit) to microSD
  - [ ] Hostname: `raspberry-pihole`
  - [ ] Enable SSH
  - [ ] Configure WiFi or plan for Ethernet
- [ ] Boot Raspberry Pi
- [ ] SSH into Pi
- [ ] Update system: `sudo apt update && sudo apt upgrade -y`
- [ ] Install Docker
- [ ] Configure static IP: 192.168.4.101
- [ ] Reboot and verify static IP
- [ ] Create `~/gruham` directory structure
- [ ] Create `.env` file with Pi configuration
- [ ] Test Docker installation

### Day 7: Deploy Core Services
- [ ] On Beelink: Copy/create Docker Compose files
- [ ] Create `/opt/gruham/.env` from `.env.example`
- [ ] Fill in all environment variables in `.env`
- [ ] Set file permissions: `chmod 600 .env`
- [ ] Create Docker network: `docker network create gruham_network`
- [ ] Deploy core services: `docker compose -f core-services/docker-compose.yml up -d`
- [ ] Verify containers running: `docker ps`
- [ ] Access Nginx Proxy Manager: http://192.168.4.100:81
  - [ ] Login (admin@example.com / changeme)
  - [ ] Change admin password
  - [ ] Update email address
- [ ] Access Portainer: https://192.168.4.100:9443
  - [ ] Create admin account
  - [ ] Connect to local Docker environment
- [ ] Note: Cloudflared will fail until tunnel configured (Week 2)

---

## Week 2: Media & Services

### Day 8-9: Media Services
- [ ] Get Plex claim token: https://www.plex.tv/claim/
- [ ] Add `PLEX_CLAIM` to `.env` file
- [ ] Deploy media services: `docker compose -f media-services/docker-compose.yml up -d`
- [ ] Access Plex: http://192.168.4.100:32400/web
  - [ ] Sign in with Plex account
  - [ ] Name server: "Gruham Plex"
  - [ ] Add Movies library: `/media/movies`
  - [ ] Add TV Shows library: `/media/tv`
  - [ ] Add Music library: `/media/music`
  - [ ] Enable hardware transcoding (Settings → Transcoder)
- [ ] Access Jellyfin: http://192.168.4.100:8096
  - [ ] Complete setup wizard
  - [ ] Create admin account
  - [ ] Add same libraries as Plex
  - [ ] Enable hardware acceleration (VAAPI)
- [ ] Access Tautulli: http://192.168.4.100:8181
  - [ ] Connect to Plex server
  - [ ] Authenticate
- [ ] Access Overseerr: http://192.168.4.100:5055
  - [ ] Sign in with Plex
  - [ ] Connect Plex server
- [ ] Test streaming a movie locally

### Day 10-11: Remote Access
- [ ] Log into Cloudflare dashboard
- [ ] Add your domain to Cloudflare
- [ ] Update nameservers at domain registrar
- [ ] Wait for DNS propagation (can take 24hrs)
- [ ] Go to Zero Trust dashboard: https://one.dash.cloudflare.com
- [ ] Create new tunnel: "gruham-tunnel"
- [ ] Copy tunnel token
- [ ] Add `TUNNEL_TOKEN` to `.env`
- [ ] Restart cloudflared: `docker compose -f core-services/docker-compose.yml restart cloudflared`
- [ ] Check logs: `docker logs cloudflared` (should see "Registered tunnel")
- [ ] In Cloudflare: Configure public hostnames
  - [ ] plex.yourdomain.com → http://192.168.4.100:32400
  - [ ] jellyfin.yourdomain.com → http://192.168.4.100:8096
  - [ ] home.yourdomain.com → http://192.168.4.100:8123
  - [ ] dash.yourdomain.com → http://192.168.4.100:3000
  - [ ] portainer.yourdomain.com → https://192.168.4.100:9443
  - [ ] nas.yourdomain.com → http://192.168.4.102:5000
- [ ] Test remote access (use phone on cellular)
  - [ ] Visit https://plex.yourdomain.com
  - [ ] Visit https://jellyfin.yourdomain.com
- [ ] Optional: Set up Tailscale VPN as backup access

### Day 12-14: Additional Services
- [ ] **Pi-hole (Raspberry Pi)**:
  - [ ] Copy docker-compose.yml for Pi
  - [ ] Create `.env` with Pi-hole password
  - [ ] Deploy: `docker compose up -d`
  - [ ] Access: http://192.168.4.101/admin
  - [ ] Configure upstream DNS (Cloudflare: 1.1.1.1)
  - [ ] Add local DNS records (plex.local, nas.local, etc.)
  - [ ] Update Eero DNS to use 192.168.4.101
  - [ ] Test ad blocking (visit news website)

- [ ] **Uptime Kuma (Raspberry Pi)**:
  - [ ] Access: http://192.168.4.101:3001
  - [ ] Create admin account
  - [ ] Add monitors for all services:
    - [ ] Plex
    - [ ] Jellyfin
    - [ ] Home Assistant
    - [ ] NAS
    - [ ] Internet (ping 8.8.8.8)
    - [ ] Cloudflare tunnel (https://plex.yourdomain.com)
  - [ ] Configure notifications (optional)

- [ ] **Home Assistant (Beelink)**:
  - [ ] Deploy: `docker compose -f automation/docker-compose.yml up -d`
  - [ ] Access: http://192.168.4.100:8123
  - [ ] Create account
  - [ ] Set location and timezone
  - [ ] Add integrations (Plex, smart devices, etc.)
  - [ ] Test basic automation

- [ ] **Management Services (Beelink)**:
  - [ ] Deploy: `docker compose -f management/docker-compose.yml up -d`
  - [ ] Homepage: http://192.168.4.100:3000
    - [ ] Edit config: `/opt/gruham/homepage/config/`
    - [ ] Add all services
  - [ ] Filebrowser: http://192.168.4.100:8080
    - [ ] Login (admin/admin)
    - [ ] Change password
  - [ ] Duplicati: http://192.168.4.100:8200
    - [ ] Create backup job
    - [ ] Source: `/source/docker-configs`
    - [ ] Destination: `/backups/nas` (or cloud)
    - [ ] Encryption: AES-256
    - [ ] Schedule: Daily at 2 AM
  - [ ] Optional: Vaultwarden (password manager)

---

## Week 3: Polish & Optimize

### Day 15-16: Workstation Setup (Claw)
- [ ] Review [Claw Setup Guide](configs/workstation/claw-setup.md)
- [ ] Install Python 3.12 on Windows
- [ ] Clone Claw repository
- [ ] Create virtual environment
- [ ] Install dependencies
- [ ] Create `.env` with Anthropic API key
- [ ] Get Plex token
- [ ] Get Home Assistant token
- [ ] Configure Claw integrations
- [ ] Test Claw locally: http://localhost:3030
- [ ] Add Claw to Cloudflare Tunnel
  - [ ] claw.yourdomain.com → http://192.168.1.10:3030
- [ ] Test remote access
- [ ] Optional: Set up auto-start on Windows boot
- [ ] Add to Uptime Kuma

### Day 17-18: Testing & Optimization
- [ ] **Test media streaming**:
  - [ ] Local direct play (Plex & Jellyfin)
  - [ ] Local transcoding
  - [ ] Remote streaming (cellular)
  - [ ] Multiple devices simultaneously
  - [ ] Mobile apps (Plex, Jellyfin)

- [ ] **Test home automation**:
  - [ ] Control devices via Home Assistant
  - [ ] Create simple automation
  - [ ] Test mobile app access
  - [ ] Verify remote access

- [ ] **Test backups**:
  - [ ] Run manual Duplicati backup
  - [ ] Verify files in /mnt/nas/backups
  - [ ] Test restore of small file
  - [ ] Verify NAS snapshots working

- [ ] **Test monitoring**:
  - [ ] Check Uptime Kuma (all green?)
  - [ ] Review Pi-hole stats
  - [ ] Check Portainer resource usage
  - [ ] Verify Watchtower checking for updates

- [ ] **Performance optimization**:
  - [ ] Check resource usage: `htop`, `docker stats`
  - [ ] Optimize Plex transcoding settings
  - [ ] Verify hardware acceleration working
  - [ ] Tune NFS mount options if needed
  - [ ] Set Docker resource limits if needed

- [ ] **Security hardening**:
  - [ ] Verify all services have strong passwords
  - [ ] Enable 2FA (Plex, Cloudflare, Home Assistant)
  - [ ] Check firewall rules: `sudo ufw status`
  - [ ] Verify no unnecessary ports exposed
  - [ ] Test SSL certificates active
  - [ ] Review Cloudflare security settings

### Day 19-21: Finalize & Document
- [ ] Create backup of all Docker configs
- [ ] Export Plex library metadata
- [ ] Export Home Assistant configuration
- [ ] Export Pi-hole settings (Teleporter)
- [ ] Document any customizations made
- [ ] Take screenshots of all dashboards
- [ ] Update network diagram if changed
- [ ] Create troubleshooting notes
- [ ] Test external USB backup drive
- [ ] Perform full backup to USB drive
- [ ] Label all cables and devices
- [ ] Create printed emergency access sheet

---

## Post-Implementation

### Immediate (First Month)
- [ ] Monitor system stability daily
- [ ] Watch for errors in logs
- [ ] Adjust automations as needed
- [ ] Add more media content
- [ ] Fine-tune performance
- [ ] Join r/homelab and share your build!

### Short-term (Months 1-3)
- [ ] Purchase 2-3 more drives for NAS (RAID redundancy)
- [ ] Expand NAS to RAID configuration
- [ ] Add more Home Assistant integrations
- [ ] Explore additional services:
  - [ ] Nextcloud (file sync)
  - [ ] Immich (photo backup)
  - [ ] Frigate (camera NVR)
  - [ ] Authentik (SSO)
- [ ] Optimize automations
- [ ] Create advanced monitoring dashboard

### Long-term (Months 3-12)
- [ ] Evaluate K3s migration
- [ ] Add more storage (towards 50TB goal)
- [ ] Expand smart home devices
- [ ] Implement advanced monitoring (Grafana/Prometheus)
- [ ] Consider adding second server for HA
- [ ] Share knowledge with community

---

## Maintenance Schedule

### Daily (Automated)
- ✓ Watchtower checks for updates
- ✓ Pi-hole updates blocklists
- ✓ NAS snapshots
- ✓ Plex metadata updates

### Weekly (5 min)
- [ ] Check Uptime Kuma dashboard
- [ ] Review Pi-hole statistics
- [ ] Check disk space (Beelink, NAS)
- [ ] Review media server activity (Tautulli)
- [ ] Check for Docker container updates

### Monthly (30 min)
- [ ] Review and test backups
- [ ] Update OS packages: `sudo apt update && sudo apt upgrade`
- [ ] Clean old/watched media
- [ ] Review logs for errors
- [ ] External USB backup
- [ ] Review resource usage trends

### Quarterly (1-2 hours)
- [ ] Test disaster recovery procedures
- [ ] Review security settings
- [ ] Update documentation
- [ ] Evaluate new services
- [ ] Clean Docker images: `docker system prune`
- [ ] Review and optimize automations

---

## Success Metrics

You know your Gruham implementation is successful when:

- ✓ All services accessible locally and remotely
- ✓ Media streaming works flawlessly
- ✓ Home automation reliable
- ✓ Ads blocked network-wide
- ✓ Backups running automatically
- ✓ Monitoring shows all services healthy
- ✓ No manual intervention needed for daily operation
- ✓ Family members using services
- ✓ You've learned new skills
- ✓ System has run stable for 30+ days

---

## Troubleshooting Quick Reference

**Service won't start**: `docker logs <container-name>`  
**Can't access locally**: Check firewall, verify service running  
**Can't access remotely**: Check Cloudflare tunnel logs  
**NAS not mounted**: `ping NAS`, `showmount -e NAS`, `sudo mount -a`  
**Slow performance**: Check `htop`, `docker stats`, network speed  
**Plex transcoding issues**: Verify HW accel, check GPU access

See [IMPLEMENTATION.md](docs/IMPLEMENTATION.md#troubleshooting) for detailed troubleshooting.

---

## Notes & Customizations

Use this space to track your specific setup details:

**Network Info**:
- Xfinity Router IP: _______________
- Eero Router IP: _______________
- Actual subnet: _______________

**Passwords** (store in password manager!):
- Beelink user: _______________
- Pi user: _______________
- NAS admin: _______________
- Plex account: _______________

**Custom Configuration**:
- Timezone: _______________
- Domain name: _______________
- Cloudflare email: _______________

**Additional Services Added**:
- _______________________________
- _______________________________
- _______________________________

---

**Implementation Started**: _______________  
**Implementation Completed**: _______________  
**Current Status**: _______________

---

*Happy building! You've got this!* 🚀
