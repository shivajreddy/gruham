# Gruham - Complete Home Server Architecture

> A comprehensive home server setup with media streaming, home automation, NAS storage, and secure remote access.

## Quick Links

- [Architecture Overview](docs/ARCHITECTURE.md)
- [NAS Setup Guide](docs/NAS-SETUP.md)
- [Implementation Roadmap](docs/IMPLEMENTATION.md)
- [Network Diagram](diagrams/network-architecture.excalidraw)
- [Docker Compose Configs](configs/)

## Hardware Overview

| Device | Role | Services |
|--------|------|----------|
| **Beelink SER3** | Primary Server | Docker host, Plex, Home Assistant, Nginx, Portainer |
| **Raspberry Pi** | Network Services | Pi-hole, Uptime Kuma |
| **NAS (TrueNAS/Synology)** | Storage Server | 10TB media storage, backups, file shares |
| **Workstation PC** | Development & AI | Claw (Claude bot), heavy compute |
| **3x Laptops** | Clients | Access services |

## Service Stack

### Core Services (Beelink)
- Nginx Proxy Manager - Reverse proxy with SSL
- Portainer - Docker management UI
- Plex Media Server - Media streaming
- Jellyfin - Open-source media server (alternative/comparison)
- Home Assistant - Home automation hub
- Cloudflared - Secure remote access tunnel
- Homepage - Unified dashboard
- Watchtower - Auto-update containers

### Network Services (Raspberry Pi)
- Pi-hole - DNS ad-blocking
- Uptime Kuma - Service monitoring

### Storage (NAS)
- TrueNAS SCALE or Synology DSM
- SMB/NFS file shares
- Automated backups
- Plex media library storage

### AI & Development (Workstation)
- Claw - Claude-powered assistant
- Development environment

### Planned Services
- Duplicati - Cloud backups
- Filebrowser - Web file manager
- Tautulli - Plex monitoring
- Overseerr - Media requests
- Vaultwarden - Password manager
- Nextcloud - Private cloud storage

## Network Architecture

```
Internet → Xfinity Router (Main)
              ├─► Workstation (Ethernet) - Development & Claw
              └─► Eero WiFi Network
                    ├─► Beelink SER3 (Ethernet) - Main server
                    ├─► Raspberry Pi - Network services
                    ├─► NAS - Storage server
                    └─► Laptops, Phones, iPads

Remote Access: Cloudflare Tunnel + Tailscale VPN
```

## Quick Start

1. Review [Architecture Documentation](docs/ARCHITECTURE.md)
2. Read [NAS Setup Guide](docs/NAS-SETUP.md) for hardware recommendations
3. Follow [Implementation Roadmap](docs/IMPLEMENTATION.md)
4. Deploy services using Docker Compose configs in `configs/`

## Remote Access

- **Domain**: Use your existing domain
- **Method**: Cloudflare Tunnel (primary) + Tailscale (backup)
- **Services**:
  - `plex.yourdomain.com` - Plex
  - `jellyfin.yourdomain.com` - Jellyfin
  - `home.yourdomain.com` - Home Assistant
  - `dash.yourdomain.com` - Homepage Dashboard
  - `nas.yourdomain.com` - NAS UI

## Security

- No port forwarding (Cloudflare Tunnel only)
- SSL certificates for all services
- 2FA where supported
- Encrypted backups
- UFW firewall on servers
- VPN backup access (Tailscale)

## Future Scaling

- Current: Docker Compose (simple, manageable)
- Future: Migrate to K3s Kubernetes cluster
- Storage: 10TB now → 50TB capability planned
- Can add more worker nodes (laptops) to K3s cluster

## Project Status

🚧 **In Planning Phase** - Architecture design complete, implementation ready to begin

---

Created with ❤️ for a complete home server experience
