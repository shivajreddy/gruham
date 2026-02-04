# Gruham Quick Start Guide

Get your home server up and running fast!

## Prerequisites Checklist

- [ ] Beelink SER3 Mini PC
- [ ] Raspberry Pi 4
- [ ] 10TB HDD
- [ ] Domain name (you have)
- [ ] NAS hardware (Synology DS423+ or DIY TrueNAS - purchase needed)
- [ ] External 4TB USB drive (purchase needed)
- [ ] 3x Ethernet cables

## Implementation Order

### Week 1: Foundation

**Day 1-2: NAS Setup**
1. Purchase NAS (recommended: Synology DS423+ ~$500)
2. Install 10TB drive
3. Configure basic storage
4. See: [NAS Setup Guide](docs/NAS-SETUP.md)

**Day 3-4: Beelink Server**
1. Install Ubuntu Server 24.04
2. Configure network (static IP: 192.168.4.100)
3. Install Docker
4. Mount NAS storage
5. See: [Implementation Guide - Phase 2](docs/IMPLEMENTATION.md#phase-2-beelink-server-setup)

**Day 5-6: Raspberry Pi**
1. Flash Raspberry Pi OS
2. Install Docker
3. Configure static IP (192.168.4.101)
4. See: [Implementation Guide - Phase 3](docs/IMPLEMENTATION.md#phase-3-raspberry-pi-setup)

**Day 7: Deploy Core Services**
1. Deploy Nginx Proxy Manager
2. Deploy Portainer
3. Set up Cloudflare Tunnel
4. See: [Implementation Guide - Phase 4](docs/IMPLEMENTATION.md#phase-4-deploy-core-services)

### Week 2: Media & Network Services

**Day 8-9: Media Services**
1. Deploy Plex Media Server
2. Deploy Jellyfin
3. Configure libraries
4. Deploy Tautulli & Overseerr
5. See: [Implementation Guide - Phase 5](docs/IMPLEMENTATION.md#phase-5-deploy-media-services)

**Day 10-11: Remote Access**
1. Configure Cloudflare Tunnel routes
2. Test remote access
3. Set up Tailscale (backup)
4. See: [Implementation Guide - Phase 6](docs/IMPLEMENTATION.md#phase-6-configure-remote-access)

**Day 12-14: Additional Services**
1. Deploy Pi-hole (DNS + ad blocking)
2. Deploy Uptime Kuma (monitoring)
3. Deploy Home Assistant
4. Deploy Homepage dashboard
5. See: [Implementation Guide - Phase 7](docs/IMPLEMENTATION.md#phase-7-deploy-additional-services)

### Week 3: Polish & Optimize

**Day 15-16: Workstation Setup**
1. Install Claw on workstation
2. Configure integrations
3. Expose via Cloudflare
4. See: [Claw Setup Guide](configs/workstation/claw-setup.md)

**Day 17-18: Testing**
1. Test all services locally
2. Test remote access
3. Test media streaming
4. Optimize performance
5. See: [Implementation Guide - Phase 9](docs/IMPLEMENTATION.md#phase-9-testing--optimization)

**Day 19-21: Backups & Monitoring**
1. Configure Duplicati backups
2. Set up monitoring alerts
3. Test disaster recovery
4. Document your setup

## Essential Commands

### Beelink Server

```bash
# SSH to Beelink
ssh gruham@192.168.4.100

# Check all containers
docker ps

# Check specific service logs
docker logs plex -f
docker logs homeassistant -f

# Restart a service
docker restart plex

# Deploy a stack
cd /opt/gruham
docker compose -f media-services/docker-compose.yml up -d

# Check NAS mounts
df -h | grep nas

# System resources
htop
docker stats
```

### Raspberry Pi

```bash
# SSH to Pi
ssh pi@192.168.4.101

# Check Pi-hole status
docker logs pihole

# Check Uptime Kuma
docker logs uptime-kuma

# Pi-hole CLI
docker exec -it pihole pihole status
```

### Useful URLs (Local Access)

After deployment, bookmark these:

- **Portainer**: https://192.168.4.100:9443
- **Nginx Proxy Manager**: http://192.168.4.100:81
- **Plex**: http://192.168.4.100:32400/web
- **Jellyfin**: http://192.168.4.100:8096
- **Home Assistant**: http://192.168.4.100:8123
- **Homepage Dashboard**: http://192.168.4.100:3000
- **Tautulli**: http://192.168.4.100:8181
- **Overseerr**: http://192.168.4.100:5055
- **Filebrowser**: http://192.168.4.100:8080
- **Duplicati**: http://192.168.4.100:8200
- **Pi-hole**: http://192.168.4.101/admin
- **Uptime Kuma**: http://192.168.4.101:3001
- **NAS (Synology)**: http://192.168.4.102:5000
- **NAS (TrueNAS)**: http://192.168.4.102

### Remote Access (via Cloudflare)

After configuring Cloudflare Tunnel:

- **Plex**: https://plex.yourdomain.com
- **Jellyfin**: https://jellyfin.yourdomain.com
- **Home Assistant**: https://home.yourdomain.com
- **Dashboard**: https://dash.yourdomain.com
- **Portainer**: https://portainer.yourdomain.com
- **NAS**: https://nas.yourdomain.com
- **Claw**: https://claw.yourdomain.com

## Common Issues & Solutions

### Can't access Beelink services

```bash
# Check if Beelink is reachable
ping 192.168.4.100

# Check if service is running
ssh gruham@192.168.4.100
docker ps | grep plex

# Check firewall
sudo ufw status
```

### NAS not mounting

```bash
# On Beelink
ping 192.168.4.102  # Is NAS reachable?
showmount -e 192.168.4.102  # Are shares exported?
sudo mount -a  # Remount all
```

### Pi-hole not blocking ads

1. Check Pi-hole is running: `docker ps | grep pihole`
2. Verify DNS in Eero app (should be 192.168.4.101)
3. Test: http://192.168.4.101/admin
4. Check query log for your device

### Plex not transcoding

1. Enable hardware acceleration in Plex settings
2. Verify GPU access: `ls /dev/dri` (should see renderD128)
3. Check transcode directory has space: `df -h /opt/gruham/plex/transcode`

### Cloudflare Tunnel not connecting

```bash
# Check tunnel status
docker logs cloudflared

# Should see "Registered tunnel connection"
# If not, check TUNNEL_TOKEN in .env
```

### Service using too much RAM

```bash
# Check resource usage
docker stats

# Limit a container (edit docker-compose.yml):
deploy:
  resources:
    limits:
      memory: 2G
      cpus: '1.0'

# Restart stack
docker compose -f <stack>.yml up -d
```

## Security Checklist

Before exposing services remotely:

- [ ] Changed all default passwords
- [ ] Enabled 2FA (Plex, Cloudflare, Home Assistant)
- [ ] Firewall (UFW) enabled on Beelink
- [ ] No port forwarding on router (using Cloudflare Tunnel only)
- [ ] SSL certificates configured (via Nginx Proxy Manager or Cloudflare)
- [ ] Backups configured and tested
- [ ] Pi-hole DNSSEC enabled
- [ ] Strong password manager in use (Vaultwarden)

## Performance Tips

### Plex Optimization

1. **Hardware Transcoding**: Settings → Transcoder → Use hardware acceleration
2. **Optimize Database**: Settings → Troubleshooting → Optimize database (monthly)
3. **Pre-convert 4K**: Keep 1080p versions for remote streaming
4. **Direct Play**: Configure clients to Direct Play when possible

### Network Optimization

1. **Ethernet over WiFi**: Use ethernet for servers (Beelink, NAS, Pi)
2. **QoS**: Configure Quality of Service on Eero (prioritize streaming)
3. **Network Speed**: Upgrade to gigabit internet if streaming remotely often

### Storage Optimization

1. **NAS Performance**: Use NFS instead of SMB for Plex (faster)
2. **SSD Cache**: Add SSD cache to NAS later for better performance
3. **Cleanup**: Remove old/watched content periodically

### Docker Optimization

1. **Prune regularly**: `docker system prune -a --volumes` (careful!)
2. **Update images**: Watchtower handles this automatically
3. **Limit resources**: Set memory/CPU limits on heavy containers

## Monitoring Dashboard Setup

Create a comprehensive monitoring dashboard:

### Uptime Kuma Monitors

Add monitors for:
- [ ] Plex (http://192.168.4.100:32400/web)
- [ ] Jellyfin (http://192.168.4.100:8096)
- [ ] Home Assistant (http://192.168.4.100:8123)
- [ ] NAS (http://192.168.4.102)
- [ ] Pi-hole (http://192.168.4.101/admin)
- [ ] Internet connectivity (8.8.8.8 ping)
- [ ] Cloudflare Tunnel (https://plex.yourdomain.com)

### Homepage Dashboard

Configure `~/gruham/homepage/config/services.yaml`:

```yaml
- Media:
    - Plex:
        href: http://192.168.4.100:32400
        icon: plex.png
    - Jellyfin:
        href: http://192.168.4.100:8096
        icon: jellyfin.png

- Home Automation:
    - Home Assistant:
        href: http://192.168.4.100:8123
        icon: home-assistant.png

- Management:
    - Portainer:
        href: https://192.168.4.100:9443
        icon: portainer.png
    - Pi-hole:
        href: http://192.168.4.101/admin
        icon: pi-hole.png

- Storage:
    - NAS:
        href: http://192.168.4.102
        icon: synology.png
```

## Backup Strategy

### What to Backup

**Critical (backup daily)**:
- Docker configs: `/opt/gruham/`
- Home Assistant config
- Vaultwarden data
- Pi-hole settings

**Important (backup weekly)**:
- Plex database
- Jellyfin metadata
- System configs: `/etc/`

**Not needed**:
- Media files (re-downloadable, too large)
- Container images (re-downloadable)

### Backup Destinations

1. **Primary**: NAS (`/mnt/nas/backups`) - Daily via Duplicati
2. **Secondary**: Cloud (Backblaze B2) - Weekly, encrypted
3. **Tertiary**: External USB - Monthly, off-site storage

### Testing Backups

**Monthly test**:
1. Restore a small file from Duplicati
2. Verify NAS snapshots are working
3. Check cloud backup size/costs

## Next Steps After Setup

### Week 4+: Use & Improve

1. **Add Media**: Populate Plex/Jellyfin libraries
2. **Smart Home**: Add devices to Home Assistant
3. **Automations**: Create useful automations
4. **Optimize**: Tune based on usage patterns
5. **Monitor**: Check Uptime Kuma weekly
6. **Maintain**: Follow maintenance schedule

### Future Enhancements

- [ ] Add more drives to NAS (redundancy!)
- [ ] Upgrade to K3s Kubernetes cluster
- [ ] Add more services (Nextcloud, Immich, etc.)
- [ ] Set up Grafana + Prometheus (advanced monitoring)
- [ ] Add cameras + Frigate NVR
- [ ] Expand automation rules

## Getting Help

### Documentation

- **Gruham Docs**: This repository!
- **Architecture**: [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)
- **Implementation**: [docs/IMPLEMENTATION.md](docs/IMPLEMENTATION.md)
- **NAS Guide**: [docs/NAS-SETUP.md](docs/NAS-SETUP.md)

### Community Resources

- **r/homelab**: https://reddit.com/r/homelab
- **r/selfhosted**: https://reddit.com/r/selfhosted
- **r/Plex**: https://reddit.com/r/Plex
- **r/HomeAssistant**: https://reddit.com/r/homeassistant
- **Plex Forums**: https://forums.plex.tv
- **Home Assistant Forums**: https://community.home-assistant.io

### Service-Specific Help

- **Plex**: https://support.plex.tv
- **Jellyfin**: https://jellyfin.org/docs
- **Home Assistant**: https://www.home-assistant.io/docs
- **Pi-hole**: https://docs.pi-hole.net
- **Docker**: https://docs.docker.com
- **Cloudflare**: https://developers.cloudflare.com

## Maintenance Schedule

### Daily (Automated)
- Watchtower checks for updates
- Pi-hole updates blocklists
- NAS snapshots
- Plex metadata refresh

### Weekly (5 minutes)
- Check Uptime Kuma (all green?)
- Review Pi-hole stats
- Check disk space
- Review Tautulli stats

### Monthly (30 minutes)
- Test backups
- Update OS (apt upgrade)
- Clean old media
- Review logs
- External USB backup

### Quarterly (1-2 hours)
- Test disaster recovery
- Security review
- Evaluate new services
- Update documentation

---

## You're All Set!

**Your Gruham home server includes**:

- ✅ **Media Server**: Plex + Jellyfin with 10TB storage
- ✅ **Home Automation**: Home Assistant for smart home
- ✅ **Ad Blocking**: Pi-hole for entire network
- ✅ **Remote Access**: Secure Cloudflare Tunnel
- ✅ **Monitoring**: Uptime Kuma + Tautulli
- ✅ **Management**: Portainer, Homepage, Filebrowser
- ✅ **Backups**: Automated with Duplicati
- ✅ **AI Assistant**: Claw on workstation
- ✅ **Expandable**: Ready to grow to 50TB+

**Enjoy your new home server!** 🎉

Questions? Check the detailed guides or ask in the homelab community.

---

**Last updated**: 2026-02-04
