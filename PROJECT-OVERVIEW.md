# Gruham Project Overview

## What is Gruham?

**Gruham** (meaning "home" in Sanskrit) is your complete home server architecture - a professionally designed, self-hosted infrastructure that brings together media streaming, home automation, network services, and secure remote access, all running on your own hardware.

---

## 🎯 Project Goals

1. **Media Streaming**: Stream your movie, TV, and music library anywhere
2. **Home Automation**: Control smart home devices and create automations
3. **Network Services**: Network-wide ad blocking and DNS management
4. **Secure Access**: Access all services remotely without exposing ports
5. **AI Assistant**: Run Claude-powered assistant on your hardware
6. **Future-Proof**: Designed to scale and migrate to Kubernetes

---

## 📦 What You Get

### Core Infrastructure
- **Nginx Proxy Manager** - Reverse proxy with automatic SSL certificates
- **Portainer** - Docker management with beautiful web UI
- **Cloudflare Tunnel** - Zero-trust remote access (no port forwarding!)
- **Watchtower** - Automatic container updates

### Media Services
- **Plex Media Server** - Premium media streaming platform
- **Jellyfin** - Open-source alternative to Plex
- **Tautulli** - Plex monitoring and statistics
- **Overseerr** - Media request management

### Home Automation
- **Home Assistant** - Powerful smart home hub with 2000+ integrations
- **Mosquitto** - MQTT broker for IoT devices (optional)
- **Node-RED** - Visual automation builder (optional)

### Network Services
- **Pi-hole** - Network-wide ad blocking and DNS server
- **Uptime Kuma** - Beautiful service monitoring dashboard
- **Unbound** - Recursive DNS for enhanced privacy (optional)

### Management & Tools
- **Homepage/Dashy** - Unified dashboard for all services
- **Filebrowser** - Web-based file manager
- **Duplicati** - Automated encrypted backups
- **Vaultwarden** - Self-hosted password manager (Bitwarden compatible)

### Storage
- **NAS** - Network Attached Storage (Synology or TrueNAS)
  - 10TB currently (your existing drive)
  - Expandable to 50TB+
  - RAID redundancy (when adding drives)
  - Automated snapshots
  - SMB/NFS file sharing

### AI & Development
- **Claw** - Claude-powered AI assistant on your workstation
  - Integrates with Plex, Home Assistant, and other services
  - Natural language control and queries

---

## 🖥️ Hardware Architecture

### Your Hardware
- **Beelink SER3** (Ryzen 3, 16GB RAM, 500GB NVMe) → Main application server
- **Raspberry Pi 4** → Network services (Pi-hole, monitoring)
- **Workstation PC** → Development and AI (Claw)
- **NAS** (to purchase) → Storage server (10TB → 50TB)

### Network Topology
```
Internet
    ↓
Xfinity Router (192.168.1.1)
    ├→ Workstation (192.168.1.10) - Ethernet
    └→ Eero Mesh WiFi (192.168.4.1)
         ├→ Beelink SER3 (192.168.4.100) - Ethernet
         ├→ Raspberry Pi (192.168.4.101) - Ethernet  
         ├→ NAS (192.168.4.102) - Ethernet
         └→ Clients (WiFi) - Laptops, phones, tablets

Remote Access: Cloudflare Tunnel (encrypted, no open ports!)
```

---

## 💰 Cost Breakdown

### Initial Investment
| Item | Cost | Status |
|------|------|--------|
| Beelink SER3 | $0 | ✅ Owned |
| Raspberry Pi 4 | $0 | ✅ Owned |
| Workstation | $0 | ✅ Owned |
| 10TB HDD | $0 | ✅ Owned |
| Domain | $0 | ✅ Owned |
| **NAS Enclosure** | **$300-500** | 🛒 **To Purchase** |
| External Backup Drive (4TB) | $100 | 🛒 To Purchase |
| **Total Initial** | **$400-600** | |

### Ongoing Costs (Annual)
| Item | Cost/Year | Notes |
|------|-----------|-------|
| Domain renewal | $0 | Already owned |
| Cloudflare Tunnel | $0 | Free tier |
| Tailscale VPN | $0 | Free personal tier |
| Plex Pass | $0-60 | Optional ($120 lifetime) |
| Electricity (24/7) | ~$85 | Beelink + Pi + NAS |
| Cloud Backup | $0-60 | Backblaze B2 ($6/TB/month) |
| **Total Annual** | **$85-205** | Very affordable! |

### Software Costs
**$0** - Everything is free and open-source!

---

## 📚 Documentation Structure

### Quick Access
- **[README.md](README.md)** - Project introduction
- **[QUICKSTART.md](QUICKSTART.md)** - Fast implementation guide
- **[PROJECT-OVERVIEW.md](PROJECT-OVERVIEW.md)** - This file

### Detailed Guides
- **[docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)** - Complete architecture documentation
- **[docs/IMPLEMENTATION.md](docs/IMPLEMENTATION.md)** - Step-by-step implementation roadmap
- **[docs/NAS-SETUP.md](docs/NAS-SETUP.md)** - NAS hardware and configuration guide

### Configuration Files
- **[configs/.env.example](configs/.env.example)** - Environment variables template
- **[configs/beelink/](configs/beelink/)** - Docker Compose files for Beelink services
  - `core-services/` - Nginx, Portainer, Cloudflare Tunnel
  - `media-services/` - Plex, Jellyfin, Tautulli, Overseerr
  - `automation/` - Home Assistant, MQTT, Node-RED
  - `management/` - Homepage, Filebrowser, Duplicati
- **[configs/raspberry-pi/](configs/raspberry-pi/)** - Pi-hole and Uptime Kuma
- **[configs/workstation/](configs/workstation/)** - Claw setup guide

### Diagrams
- **[diagrams/network-architecture.excalidraw](diagrams/network-architecture.excalidraw)** - Network topology diagram
- **[diagrams/README.md](diagrams/README.md)** - How to view diagrams

---

## 🚀 Implementation Timeline

### Week 1: Foundation
- **Days 1-2**: Purchase and set up NAS
- **Days 3-4**: Install Ubuntu on Beelink, configure network
- **Days 5-6**: Set up Raspberry Pi
- **Day 7**: Deploy core infrastructure services

### Week 2: Media & Services
- **Days 8-9**: Deploy Plex and Jellyfin
- **Days 10-11**: Configure Cloudflare Tunnel for remote access
- **Days 12-14**: Deploy Pi-hole, Home Assistant, monitoring

### Week 3: Polish & Optimize
- **Days 15-16**: Set up Claw on workstation
- **Days 17-18**: Test all services and optimize
- **Days 19-21**: Configure backups, finalize setup

**Total Time**: ~3 weeks at a comfortable pace

---

## 🎨 Design Philosophy

### Start Simple, Scale Smart
- Begin with Docker Compose (easy to manage)
- Design with future Kubernetes migration in mind
- Add services as needed, not all at once

### Security First
- No port forwarding (Cloudflare Tunnel only)
- SSL everywhere
- 2FA on critical services
- Encrypted backups
- Regular updates (automated via Watchtower)

### Reliability Matters
- Separate DNS server (Pi-hole on dedicated Pi)
- Automated backups (multiple destinations)
- Monitoring and alerts (Uptime Kuma)
- NAS snapshots for data protection

### Open Source Priority
- Use open-source software where possible
- Support the community
- Learn and share knowledge

---

## 🔒 Security Features

### Network Security
- ✅ No exposed ports (Cloudflare Tunnel ingress only)
- ✅ UFW firewall on all servers
- ✅ Network segmentation (main vs. home network)
- ✅ Pi-hole with DNSSEC

### Access Control
- ✅ SSL/TLS for all web interfaces
- ✅ Strong passwords (20+ characters)
- ✅ 2FA where supported
- ✅ Password manager (Vaultwarden)
- ✅ Cloudflare Zero Trust policies

### Data Protection
- ✅ Encrypted backups (AES-256)
- ✅ NAS snapshots (ransomware protection)
- ✅ Multiple backup destinations
- ✅ Tested recovery procedures

### Updates
- ✅ Watchtower for container updates
- ✅ Regular OS updates
- ✅ Security advisories monitoring

---

## 📊 Performance Expectations

### Plex/Jellyfin Streaming
- **Direct Play**: Unlimited concurrent streams (network-limited)
- **Transcoding**: 1-2 simultaneous 1080p streams (Beelink)
- **4K**: Direct Play only (no transcoding)
- **Hardware Acceleration**: AMD GPU support

### Network Performance
- **Local streaming**: Gigabit speeds (900+ Mbps)
- **Remote streaming**: Limited by upload speed
- **NAS throughput**: ~100-120 MB/s on gigabit

### Resource Usage (Beelink)
- **RAM**: 8-12GB of 16GB (comfortable)
- **CPU**: 15-30% average, 50-80% when transcoding
- **Storage**: ~300-400GB of 500GB for apps/configs

### Power Consumption
- **Beelink**: ~20W (24/7 = ~$50/year)
- **Raspberry Pi**: ~5W (24/7 = ~$5/year)
- **NAS**: ~10W idle (24/7 = ~$30/year)
- **Total**: ~35W = **~$85/year**

---

## 🛠️ Technology Stack

### Operating Systems
- **Ubuntu Server 24.04 LTS** (Beelink)
- **Raspberry Pi OS Lite 64-bit** (Pi)
- **TrueNAS SCALE** or **Synology DSM** (NAS)
- **Windows 11 Pro** (Workstation)

### Container Platform
- **Docker Engine** (current)
- **Docker Compose** (orchestration)
- **K3s Kubernetes** (future migration path)

### Networking
- **Nginx Proxy Manager** (reverse proxy)
- **Cloudflare Tunnel** (remote access)
- **Tailscale** (VPN backup)
- **Pi-hole** (DNS + ad blocking)

### Storage
- **ZFS** (TrueNAS) or **BTRFS** (Synology)
- **NFS** (primary file sharing)
- **SMB/CIFS** (Windows compatibility)

---

## 🎯 Use Cases

### Media Enthusiast
- Stream your entire media library from anywhere
- Watch on any device (TV, phone, tablet, laptop)
- Share with family members
- Organize and track your collection

### Smart Home Builder
- Control all smart devices from one interface
- Create powerful automations
- Voice control integration
- Energy monitoring and optimization

### Privacy Advocate
- Ad-free browsing on all devices
- Self-hosted services (no data sold)
- Encrypted cloud backups
- Own your data

### Homelab Enthusiast
- Learn Docker, networking, Linux
- Experiment with new services
- Path to Kubernetes
- Join the homelab community

### Remote Worker
- Access files from anywhere
- Secure VPN access to home network
- Self-hosted cloud storage
- AI assistant for productivity

---

## 📈 Expansion Roadmap

### Phase 1: Foundation (Weeks 1-3)
✅ All core services operational
✅ Media streaming working
✅ Remote access configured
✅ Backups automated

### Phase 2: Enhancement (Months 1-3)
- [ ] Add more drives to NAS (RAID redundancy)
- [ ] Optimize Home Assistant automations
- [ ] Expand media library
- [ ] Add cameras + Frigate NVR
- [ ] Deploy Nextcloud (file sync)

### Phase 3: Advanced (Months 3-6)
- [ ] Grafana + Prometheus monitoring
- [ ] Immich (Google Photos alternative)
- [ ] Authentik (SSO/auth provider)
- [ ] Additional automation integrations
- [ ] Guest network isolation

### Phase 4: Kubernetes Migration (Months 6-12)
- [ ] Install K3s on Beelink (single node)
- [ ] Add Pi as worker node
- [ ] Convert Docker Compose to K8s manifests
- [ ] Implement Longhorn storage
- [ ] Use Helm charts for deployments
- [ ] Add laptops as worker nodes (on-demand)

---

## 🤝 Community & Support

### Get Help
- **r/homelab** - General homelab questions
- **r/selfhosted** - Self-hosting services
- **r/Plex** - Plex-specific help
- **r/HomeAssistant** - Home Assistant community
- **Discord servers** - Real-time help

### Share Your Build
- Post to r/homelab when complete
- Share lessons learned
- Help others starting out
- Contribute to open-source projects

### Stay Updated
- Follow service changelogs
- Join mailing lists for security updates
- Participate in community discussions
- Learn new technologies

---

## ✅ Success Criteria

Your Gruham implementation is successful when:

- [ ] All services accessible locally
- [ ] Remote access working securely
- [ ] Media streaming from anywhere
- [ ] Home automation functional
- [ ] Ads blocked network-wide
- [ ] Backups running automatically
- [ ] Monitoring alerts configured
- [ ] System documented
- [ ] You're actively using it!
- [ ] You've learned something new

---

## 🎓 What You'll Learn

### Technical Skills
- Linux system administration
- Docker containerization
- Networking (DNS, VPN, reverse proxy)
- Storage management (RAID, NFS, ZFS)
- Security best practices
- Automation and scripting

### Soft Skills
- Planning and documentation
- Troubleshooting methodology
- Project management
- Community engagement

---

## 🌟 Why Gruham?

### Control
- **Your hardware**, your rules
- No monthly fees (except optional)
- No data collection or privacy concerns

### Learning
- Hands-on with cutting-edge tech
- Real-world DevOps experience
- Builds valuable skills

### Cost-Effective
- One-time hardware investment
- Minimal ongoing costs
- Replaces multiple subscriptions

### Scalable
- Start simple, grow over time
- Add services as needed
- Path to professional-grade infrastructure

### Fun!
- Build something awesome
- Join a great community
- Impress friends and family

---

## 📞 Project Contacts & Resources

### Official Documentation
- Docker: https://docs.docker.com
- Plex: https://support.plex.tv
- Home Assistant: https://www.home-assistant.io/docs
- Pi-hole: https://docs.pi-hole.net

### Your Gruham Setup
- **Project Location**: `C:\Users\smpl\dev\projects\gruham`
- **Primary Server**: Beelink SER3 (192.168.4.100)
- **Network DNS**: Raspberry Pi (192.168.4.101)
- **Storage**: NAS (192.168.4.102)
- **AI Assistant**: Workstation (192.168.1.10)

### Next Steps
1. Read [QUICKSTART.md](QUICKSTART.md) for fast track
2. Read [docs/IMPLEMENTATION.md](docs/IMPLEMENTATION.md) for detailed steps
3. Purchase NAS hardware (see [docs/NAS-SETUP.md](docs/NAS-SETUP.md))
4. Start building!

---

## 🏆 Congratulations!

You now have a complete, professional-grade home server architecture planned and ready to implement. This isn't just a hobby project - it's a learning platform, a privacy solution, and a powerful home infrastructure that will serve you for years to come.

**Welcome to the Gruham project. Let's build something amazing!** 🚀

---

**Project Started**: February 4, 2026  
**Status**: Architecture Complete, Ready for Implementation  
**Version**: 1.0

---

*Gruham - Your complete home server solution.*
