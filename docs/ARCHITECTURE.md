# Gruham - Detailed Architecture

## Table of Contents
- [Overview](#overview)
- [Hardware Allocation](#hardware-allocation)
- [Network Design](#network-design)
- [Service Architecture](#service-architecture)
- [Storage Strategy](#storage-strategy)
- [Security Architecture](#security-architecture)
- [Technology Stack](#technology-stack)

## Overview

Gruham is a comprehensive home server setup designed to provide:
- **Media Streaming**: Plex + Jellyfin for movies, TV, music
- **Home Automation**: Home Assistant for smart home control
- **Network Services**: Pi-hole for ad-blocking, DNS management
- **Storage**: NAS with 10TB capacity, expandable to 50TB
- **Remote Access**: Secure access from anywhere via Cloudflare Tunnel
- **AI Assistant**: Claw running on powerful workstation
- **Monitoring**: Uptime tracking, service health, resource monitoring

**Design Philosophy**:
- Start simple with Docker Compose
- Design for future K3s migration
- Prioritize security and reliability
- Use open-source tools where possible
- Maintain high availability for core services

---

## Hardware Allocation

### 1. Beelink SER3 Mini PC - Primary Application Server

**Specifications**:
- CPU: AMD Ryzen 3 3200U (2C/4T @ 3.5GHz)
- RAM: 16GB DDR4
- Storage: 500GB NVMe PCIe 3.0 x4
- Network: WiFi 6, Gigabit Ethernet
- Power: ~15-25W idle

**Operating System**: Ubuntu Server 24.04 LTS

**Role**: Main application server running all containerized services

**Services Hosted**:
- Docker Engine + Docker Compose
- Nginx Proxy Manager (reverse proxy)
- Plex Media Server
- Jellyfin Media Server
- Home Assistant
- Portainer (Docker UI)
- Cloudflared (Cloudflare Tunnel)
- Homepage/Dashy (dashboard)
- Watchtower (auto-updates)
- Filebrowser
- Duplicati (backups)
- Tautulli (Plex monitoring)
- Overseerr (media requests)

**Why Beelink for this role**:
- Perfect balance of power and efficiency
- 16GB RAM sufficient for multiple containers
- Low power consumption for 24/7 operation
- AMD GPU can assist with Plex transcoding
- NVMe storage fast enough for databases and configs

**Storage Usage**:
- 50GB: OS and system files
- 100GB: Docker images and volumes
- 50GB: Application configs and databases
- 50GB: Temporary transcoding (Plex/Jellyfin)
- 250GB: Reserved/future use

**Network**: Ethernet connection to Eero router (preferred) or WiFi 6

---

### 2. Raspberry Pi 4 - Network Services Server

**Specifications**:
- Model: Raspberry Pi 4 (4GB or 8GB recommended)
- Storage: MicroSD 32GB+ (or USB SSD for better performance)
- Network: Gigabit Ethernet
- Power: ~3-7W

**Operating System**: Raspberry Pi OS Lite (64-bit) or DietPi

**Role**: Dedicated network services and lightweight monitoring

**Services Hosted**:
- Pi-hole (DNS server + ad blocking)
- Uptime Kuma (service monitoring)
- Optional: Unbound (recursive DNS for additional privacy)

**Why Raspberry Pi for this role**:
- Extremely low power consumption (24/7 operation)
- Perfect for DNS/DHCP services (critical network functions)
- Separating DNS from main server increases resilience
- If Beelink goes down, network DNS still functions
- Lightweight monitoring doesn't require much resources

**Network**: Ethernet connection to Eero router (critical for DNS reliability)

---

### 3. NAS (Network Attached Storage) - Storage Server

**Current Configuration**:
- 1x 10TB HDD (existing drive)
- Expandable to 4-8 bays for 50TB+ total

**Recommended Hardware Options** (see NAS-SETUP.md for details):

**Option A: DIY TrueNAS Build** (~$400-600)
- Used/refurb PC or server
- 4-8 bay hot-swap cage
- 8-16GB ECC RAM (recommended)
- Low-power CPU (Intel i3/Pentium or AMD)
- TrueNAS SCALE (free, Linux-based)

**Option B: Synology NAS** (~$300-500 for enclosure)
- DS423+ (4-bay) or DS923+ (4-bay with expansion)
- User-friendly web UI
- Excellent backup software
- Can run Docker containers
- Great mobile apps

**Option C: QNAP NAS** (~$250-450)
- TS-464 (4-bay) or similar
- Good price/performance
- Runs containers and VMs
- Decent software ecosystem

**Option D: Budget DIY** (~$200-300)
- Old PC + OpenMediaVault
- 4x SATA ports minimum
- Low power Celeron/Pentium
- OpenMediaVault (Debian-based, free)

**Recommendation**: **Synology DS423+ or DIY TrueNAS** for best balance

**Services Hosted**:
- TrueNAS SCALE or Synology DSM
- SMB/CIFS file shares (Windows compatibility)
- NFS shares (Linux/Unix, better for Plex)
- Automated snapshots
- Backup target for other servers
- Optional: Docker containers (if using TrueNAS SCALE/Synology)

**Storage Layout**:
```
Pool 1 (Current): 10TB single drive
├── Media Library (8TB)
│   ├── Movies (4TB)
│   ├── TV Shows (3TB)
│   └── Music (500GB)
├── Backups (1TB)
│   ├── Server configs
│   └── Photos/Documents
└── Shared Files (1TB)

Future Expansion:
Pool 2: 4x 10TB in RAID-Z1 (30TB usable)
Total Capacity: 40TB usable → 50TB with optimization
```

**Network**: Gigabit Ethernet to Eero router

---

### 4. Workstation PC - Development & AI Compute

**Role**: Development workstation and AI assistant host

**Services**:
- Claw (Claude-powered assistant)
- Development environments
- Heavy compute tasks (video encoding, AI inference)
- Emergency backup server if Beelink fails

**Network**: Ethernet to Xfinity main router (main network)

**Usage Pattern**: Not 24/7, on-demand for development and AI tasks

---

### 5. Client Devices

- **3x Laptops**: Access all services, Plex clients
- **Multiple Phones/iPads**: Mobile access via apps
- **Smart TVs** (assumed): Plex/Jellyfin clients

---

## Network Design

### Network Topology

```
┌─────────────────────────────────────────────────────────────────┐
│                          INTERNET                               │
└────────────────────┬────────────────────────────────────────────┘
                     │
         ┌───────────▼──────────────┐
         │  Xfinity Router (Main)   │
         │  Gateway: 192.168.1.1    │
         └───┬──────────────────┬───┘
             │                  │
    ┌────────▼─────┐      ┌────▼────────────────────┐
    │ Workstation  │      │   Eero Router/Mesh      │
    │ 192.168.1.10 │      │   192.168.4.1           │
    │  (Ethernet)  │      │   (WiFi 6 + Ethernet)   │
    └──────────────┘      └────┬────────────────────┘
                               │
                ┌──────────────┼──────────────┐
                │              │              │
         ┌──────▼─────┐ ┌─────▼─────┐ ┌─────▼──────┐
         │  Beelink   │ │ Rasp Pi   │ │    NAS     │
         │ .4.100     │ │ .4.101    │ │  .4.102    │
         │ (Ethernet) │ │(Ethernet) │ │ (Ethernet) │
         └────────────┘ └───────────┘ └────────────┘
                               │
                         ┌─────▼──────┐
                         │ Laptops,   │
                         │ Phones,    │
                         │ iPads      │
                         │ (WiFi)     │
                         └────────────┘

External Access:
┌──────────────────┐
│ Cloudflare       │──► Tunnels to Beelink services
│ Tunnel           │    (plex, home, dash, etc.)
└──────────────────┘

┌──────────────────┐
│ Tailscale VPN    │──► Mesh network for direct access
└──────────────────┘
```

### IP Address Scheme

**Main Network (Xfinity)**: `192.168.1.0/24`
- Gateway: `192.168.1.1`
- Workstation: `192.168.1.10` (static)

**Eero Network**: `192.168.4.0/24` (adjust if different)
- Gateway: `192.168.4.1`
- Beelink SER3: `192.168.4.100` (static DHCP reservation)
- Raspberry Pi: `192.168.4.101` (static DHCP reservation)
- NAS: `192.168.4.102` (static DHCP reservation)
- DHCP Pool: `192.168.4.150-250` (dynamic clients)

**Note**: Verify your actual Eero subnet and adjust accordingly.

### DNS Configuration

**Primary DNS**: Pi-hole (192.168.4.101)
**Secondary DNS**: Cloudflare (1.1.1.1) or Google (8.8.8.8)

**Pi-hole Configuration**:
- Upstream DNS: Cloudflare (1.1.1.1, 1.0.0.1)
- Or use Unbound for recursive DNS (more privacy)
- Block lists: Standard + additional privacy lists
- Custom DNS entries for local services

**Local DNS Records** (configured in Pi-hole):
```
gruham.local        → 192.168.4.100 (Beelink)
plex.local          → 192.168.4.100
jellyfin.local      → 192.168.4.100
home.local          → 192.168.4.100
nas.local           → 192.168.4.102
pihole.local        → 192.168.4.101
```

### Port Strategy

**Internal Ports** (LAN only):
- Nginx Proxy Manager: 81 (admin), 80, 443 (proxied services)
- Plex: 32400
- Jellyfin: 8096
- Home Assistant: 8123
- Portainer: 9443
- Pi-hole: 80 (web UI), 53 (DNS)
- NAS: 5000 (Synology), 443 (TrueNAS)

**External Access**: NO PORT FORWARDING
- All external access via Cloudflare Tunnel
- Backup access via Tailscale VPN

### Network Shares (NAS → Servers)

**Beelink mounts from NAS**:
```
//192.168.4.102/media → /mnt/nas/media (NFS or SMB)
//192.168.4.102/backups → /mnt/nas/backups
```

**Docker volumes use NAS storage**:
- Plex library: `/mnt/nas/media`
- Jellyfin library: `/mnt/nas/media` (same content)
- Backup target: `/mnt/nas/backups`

---

## Service Architecture

### Application Layers

```
┌─────────────────────────────────────────────────────┐
│             External Access Layer                    │
│  Cloudflare Tunnel  |  Tailscale VPN               │
└────────────────────┬────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────┐
│          Reverse Proxy Layer (Beelink)              │
│         Nginx Proxy Manager (Port 80/443)           │
│  - SSL Termination                                  │
│  - Routing: subdomain → service                     │
│  - Access Control Lists                             │
└────────────────────┬────────────────────────────────┘
                     │
      ┌──────────────┼──────────────┐
      │              │              │
┌─────▼─────┐ ┌─────▼──────┐ ┌────▼────────┐
│   Media   │ │Automation  │ │ Management  │
│  Services │ │  Services  │ │  Services   │
│           │ │            │ │             │
│ - Plex    │ │ - Home     │ │ - Portainer │
│ - Jellyfin│ │   Assistant│ │ - Homepage  │
│ - Tautulli│ │            │ │ - Filebrowser│
│ - Overseerr│ │           │ │ - Duplicati │
└───────────┘ └────────────┘ └─────────────┘
      │              │              │
      └──────────────┼──────────────┘
                     │
           ┌─────────▼──────────┐
           │   Storage Layer    │
           │   NAS (10TB)       │
           │   - Media files    │
           │   - Backups        │
           │   - Snapshots      │
           └────────────────────┘
```

### Service Dependencies

```
Internet Access Required:
- Cloudflare Tunnel (for remote access)
- Plex (metadata, authentication)
- Home Assistant (some integrations)
- Watchtower (container updates)

NAS Required:
- Plex (media library)
- Jellyfin (media library)
- Duplicati (backup target)

Pi-hole Required (recommended):
- All devices (DNS + ad blocking)

Can operate independently:
- Nginx Proxy Manager
- Portainer
- Home Assistant (local control)
- Jellyfin (local streaming)
```

### Docker Compose Stack Organization

**Single docker-compose.yml approach** vs **Multiple compose files**:

For Gruham, we'll use **multiple compose files** for better organization:

```
/opt/gruham/
├── core-services/
│   └── docker-compose.yml    # Nginx, Portainer, Cloudflared
├── media-services/
│   └── docker-compose.yml    # Plex, Jellyfin, Tautulli, Overseerr
├── automation/
│   └── docker-compose.yml    # Home Assistant
├── management/
│   └── docker-compose.yml    # Homepage, Filebrowser, Duplicati
└── .env                       # Shared environment variables
```

**Benefits**:
- Start/stop service groups independently
- Easier troubleshooting
- Clearer organization
- Can update one stack without affecting others

---

## Storage Strategy

### NAS Storage Design

**Current State**: 1x 10TB drive
**Future State**: 4-6 drives, 30-50TB usable

**File System Options**:

**For TrueNAS**:
- ZFS (recommended)
- Built-in snapshots, compression, deduplication
- RAID-Z1 (1 drive parity) or RAID-Z2 (2 drive parity)

**For Synology/QNAP**:
- BTRFS (modern, snapshots, data integrity)
- Synology Hybrid RAID (flexible expansion)

**Current Setup** (Single 10TB drive):
```
⚠️  No redundancy - backups critical!

/volume1/
├── media/              (8TB - Plex/Jellyfin content)
│   ├── movies/
│   ├── tv/
│   ├── music/
│   └── photos/
├── backups/            (1TB - Server configs, snapshots)
│   ├── beelink/
│   ├── raspberry-pi/
│   └── workstation/
└── shares/             (1TB - General file storage)
    ├── documents/
    ├── downloads/
    └── temp/
```

**Future Expansion** (4x 10TB in RAID-Z1):
```
Pool: 4x 10TB drives
RAID-Z1: 30TB usable (1 drive can fail)

/volume1/
├── media/              (25TB)
├── backups/            (3TB)
├── shares/             (2TB)
└── snapshots/          (automatic, ~10% overhead)

Total: ~30TB usable
```

**Backup Strategy**:
```
Critical Data (configs, photos, documents):
├── On-NAS snapshots (hourly/daily)
├── Duplicati → Cloud (Backblaze B2 / Google Drive)
└── Monthly backup to external USB drive (off-site)

Media Library (movies/TV):
├── No cloud backup (too large, re-downloadable)
└── On-NAS snapshots only

Database/Configs:
├── Daily backup to NAS
└── Weekly cloud backup (encrypted)
```

### NAS Share Configuration

**SMB/CIFS Shares** (Windows compatibility):
```
\\192.168.4.102\media       (Read-only for most users)
\\192.168.4.102\shares      (Read-write)
\\192.168.4.102\backups     (Restricted to admin)
```

**NFS Shares** (Better performance for Linux):
```
192.168.4.102:/mnt/pool1/media     → Beelink /mnt/nas/media
192.168.4.102:/mnt/pool1/backups   → Beelink /mnt/nas/backups
```

**Permissions**:
- Media: Read-only for Plex/Jellyfin users
- Backups: Root/admin only
- Shares: User-based permissions

### Beelink Local Storage

**500GB NVMe allocation**:
```
/ (root)                    50GB   (OS)
/var/lib/docker            200GB   (Container images, volumes)
/opt/gruham                100GB   (Configs, databases)
/tmp/transcode             50GB    (Plex/Jellyfin temp)
/home                      20GB    (User files)
Swap                       16GB    (Match RAM)
Reserved                   64GB    (Future)
```

**What lives on Beelink vs NAS**:

**Beelink (Fast NVMe)**:
- Docker container images
- Application databases (Plex metadata, HA config)
- Temporary transcoding files
- Docker volumes for stateful apps
- OS and system files

**NAS (Large HDD)**:
- Media files (movies, TV, music)
- Long-term backups
- Shared documents
- Photo libraries
- Archive data

---

## Security Architecture

### Defense in Depth

**Layer 1: Network Perimeter**
- No exposed ports to internet
- Cloudflare Tunnel as only ingress
- Tailscale VPN as backup access
- UFW firewall on all servers

**Layer 2: Application Security**
- Nginx Proxy Manager with SSL/TLS
- HTTP Strict Transport Security (HSTS)
- Access Control Lists per service
- Rate limiting on public services

**Layer 3: Authentication**
- Strong passwords (20+ characters)
- 2FA where supported (Home Assistant, Plex, Cloudflare)
- Vaultwarden for password management
- Separate user accounts per person

**Layer 4: Data Security**
- Encrypted backups (Duplicati with AES-256)
- HTTPS/TLS for all web interfaces
- NAS snapshots for ransomware protection
- Regular security updates (Watchtower)

**Layer 5: Monitoring & Response**
- Uptime Kuma for service monitoring
- Pi-hole query logs for suspicious DNS
- Portainer for container security scanning
- Failed login alerts (Cloudflare, HA)

### Firewall Rules (UFW on Beelink)

```bash
# Default policies
ufw default deny incoming
ufw default allow outgoing

# SSH (from LAN only)
ufw allow from 192.168.4.0/24 to any port 22

# HTTP/HTTPS (from LAN, Cloudflare proxies traffic)
ufw allow from 192.168.4.0/24 to any port 80
ufw allow from 192.168.4.0/24 to any port 443

# Plex (LAN only, remote via tunnel)
ufw allow from 192.168.4.0/24 to any port 32400

# Tailscale
ufw allow in on tailscale0

# NFS (if using, from NAS only)
ufw allow from 192.168.4.102 to any port 2049

# Enable
ufw enable
```

### Cloudflare Tunnel Configuration

**Tunnel Architecture**:
```
User (anywhere) → Cloudflare Edge
                      ↓ (encrypted tunnel)
                  cloudflared on Beelink
                      ↓ (localhost)
                  Nginx Proxy Manager
                      ↓
                  Backend Services
```

**Tunnel Routing**:
```
plex.yourdomain.com      → localhost:32400
jellyfin.yourdomain.com  → localhost:8096
home.yourdomain.com      → localhost:8123
dash.yourdomain.com      → localhost:3000
nas.yourdomain.com       → 192.168.4.102:5000
portainer.yourdomain.com → localhost:9443
```

**Security Features**:
- Zero Trust access policies
- Email verification required
- Geographic restrictions (optional)
- Rate limiting
- DDoS protection
- WAF (Web Application Firewall)

### Backup Security

**Duplicati Configuration**:
- Encryption: AES-256
- Password: Strong passphrase (stored in Vaultwarden)
- Backup targets: Encrypted cloud storage
- Backup frequency: Daily (configs), Weekly (full)
- Retention: 30 daily, 12 weekly, 12 monthly

**Backup Contents**:
```
/opt/gruham/           (All Docker configs)
/home/                 (User files)
/etc/                  (System configs)
NAS snapshots          (Handled by NAS itself)
```

---

## Technology Stack

### Operating Systems

| Device | OS | Version | Rationale |
|--------|----|---------| ---------|
| Beelink | Ubuntu Server | 24.04 LTS | Stable, 5yr support, excellent Docker support |
| Raspberry Pi | Raspberry Pi OS Lite | 64-bit | Optimized for hardware, lightweight |
| NAS | TrueNAS SCALE or Synology DSM | Latest | ZFS support (TrueNAS) or ease of use (Synology) |
| Workstation | Windows 11 Pro | Current | Development environment, WSL2 for Linux |

### Container Orchestration

**Current**: Docker + Docker Compose
- Version: Docker Engine 24.x+
- Compose: v2.x (docker compose, not docker-compose)

**Future**: K3s Kubernetes
- Lightweight K8s distribution
- Perfect for homelab/edge
- Migration path when ready

### Core Infrastructure Services

| Service | Technology | Purpose |
|---------|-----------|---------|
| Reverse Proxy | Nginx Proxy Manager | Route traffic, SSL termination |
| Tunnel | Cloudflare Tunnel (cloudflared) | Secure remote access |
| VPN | Tailscale | Backup access, mesh network |
| DNS | Pi-hole | Ad blocking, local DNS |
| Container UI | Portainer CE | Docker management |
| Monitoring | Uptime Kuma | Service uptime tracking |
| Updates | Watchtower | Auto-update containers |
| Dashboard | Homepage or Dashy | Unified service access |

### Media Services

| Service | Technology | Purpose |
|---------|-----------|---------|
| Media Server 1 | Plex Media Server | Primary media streaming |
| Media Server 2 | Jellyfin | Open-source alternative, comparison |
| Plex Monitoring | Tautulli | Watch statistics, notifications |
| Media Requests | Overseerr | User requests, integration with *arr |
| File Management | Filebrowser | Web-based file uploads/organization |

### Automation Services

| Service | Technology | Purpose |
|---------|-----------|---------|
| Home Automation | Home Assistant | Smart home hub, automation engine |
| MQTT Broker | Mosquitto (optional) | IoT device messaging |
| Node-RED | Node-RED (optional) | Visual automation builder |

### Storage Services

| Service | Technology | Purpose |
|---------|-----------|---------|
| NAS OS | TrueNAS SCALE or Synology DSM | Storage management |
| File System | ZFS or BTRFS | Data integrity, snapshots |
| Protocols | SMB/CIFS, NFS | File sharing |
| Backup | Duplicati | Cloud backup with encryption |

### AI & Development

| Service | Technology | Host |
|---------|-----------|------|
| AI Assistant | Claw | Workstation |
| Development | VSCode, WSL2 | Workstation |

### Future/Optional Services

- Vaultwarden (password manager)
- Nextcloud (cloud storage)
- Grafana + Prometheus (advanced monitoring)
- Loki (log aggregation)
- Immich (photo backup, Google Photos alternative)
- Frigate (camera NVR with AI detection)
- Authentik (SSO/auth provider)

---

## Service Priority Tiers

### Tier 1: Critical (Must work 24/7)
- Pi-hole (network DNS)
- Home Assistant (smart home control)
- NAS (storage)
- Nginx Proxy Manager (routing)
- Cloudflare Tunnel (remote access)

### Tier 2: Important (Should work, acceptable downtime)
- Plex (media streaming)
- Jellyfin (media alternative)
- Portainer (management)
- Uptime Kuma (monitoring)

### Tier 3: Nice to Have
- Tautulli (statistics)
- Overseerr (requests)
- Homepage (dashboard)
- Filebrowser (file management)
- Duplicati (backups can run on schedule)

### Tier 4: Optional/Future
- Vaultwarden
- Nextcloud
- Grafana/Prometheus
- Additional services

---

## Disaster Recovery Plan

### Backup Strategy

**What gets backed up**:
1. Docker configs (`/opt/gruham/`) - Daily
2. Home Assistant config - Daily
3. Plex database - Weekly
4. System configs (`/etc/`) - Weekly
5. NAS snapshots - Hourly/Daily (on NAS itself)

**Backup destinations**:
1. Primary: NAS (`/backups/`)
2. Secondary: Cloud (Backblaze B2 or Google Drive) - Encrypted
3. Tertiary: External USB drive (monthly, off-site)

### Recovery Procedures

**Beelink failure**:
1. Install Ubuntu Server on replacement/repaired system
2. Restore Docker configs from NAS or cloud backup
3. Run Docker Compose to recreate containers
4. All data intact on NAS

**NAS failure** (single drive, no redundancy):
1. ⚠️ All media lost unless backed up
2. Restore critical data from cloud backups
3. Rebuild media library (re-download)
4. **Prevention**: Upgrade to multi-drive RAID soon

**Raspberry Pi failure**:
1. Flash new Pi-hole image
2. Restore Pi-hole config (teleporter backup)
3. Minimal downtime, can use 8.8.8.8 as temp DNS

**Network failure**:
1. Services still work on LAN
2. Remote access unavailable
3. Local access via .local domains

### High Availability Considerations

**Future improvements**:
- Add second Beelink/NUC for failover
- Set up K3s cluster (Beelink + laptops)
- Use Longhorn for distributed storage replication
- Database replication for critical services
- Load balancing for media streaming

---

## Cost Analysis

### Initial Investment

| Item | Cost | Notes |
|------|------|-------|
| Beelink SER3 | $0 | Already owned |
| Raspberry Pi 4 | $0 | Already owned |
| Workstation | $0 | Already owned |
| 10TB HDD | $0 | Already owned |
| Domain name | $0 | Already owned |
| **NAS Enclosure** | **$300-500** | **Synology DS423+ or DIY** |
| Extra drives (future) | $0 now | 3x 10TB @ ~$500 later |
| External backup drive | $100 | 4TB USB for off-site backup |
| **Total Initial** | **$400-600** | **Just NAS + backup drive** |

### Ongoing Costs

| Item | Cost/Year | Notes |
|------|-----------|-------|
| Domain renewal | $0 | Already owned |
| Cloudflare Tunnel | $0 | Free tier |
| Tailscale | $0 | Personal tier (20 devices) |
| Plex Pass | $0 or $60 | Optional, $120 lifetime available |
| Electricity (Beelink 24/7) | ~$50 | 20W avg @ $0.12/kWh |
| Electricity (RPi 24/7) | ~$5 | 5W avg |
| Electricity (NAS 24/7) | ~$30 | 10W idle |
| Cloud backup (Backblaze B2) | $0-60 | 100GB free, then $6/TB/month |
| **Total Annual** | **$85-205** | **Very reasonable** |

### Software Costs

**$0 - All free/open-source**:
- Ubuntu Server (free)
- Docker (free)
- TrueNAS SCALE (free) or OpenMediaVault (free)
- All services are open-source
- Plex (free tier works great, Pass optional)

---

## Performance Expectations

### Plex Transcoding

**Beelink SER3 capability**:
- Direct Play: Unlimited streams (network-limited)
- 1080p transcode: 1-2 simultaneous streams
- 4K transcode: Not recommended (use Direct Play)
- Hardware acceleration: Yes (AMD GPU)

**Optimization tips**:
- Encourage Direct Play (compatible formats)
- Use hardware transcoding (Plex Pass required)
- Consider separate GPU if heavy transcoding needed
- 4K content: Keep 1080p versions for remote users

### Network Performance

**Gigabit Ethernet** (1000 Mbps):
- 1080p stream: ~10-20 Mbps (50-100 simultaneous)
- 4K stream: ~50-100 Mbps (10-20 simultaneous)
- NAS read speed: ~100-120 MB/s (800-960 Mbps)
- More than sufficient for home use

**Bottlenecks**:
- NAS HDD speed: ~120 MB/s sequential read
- WiFi 6: Varies (200-900 Mbps typical)
- Internet upload: Limits remote streaming

### Resource Usage (Beelink)

**Expected RAM usage**:
- OS: 1-2GB
- Plex: 2-4GB (depending on libraries)
- Jellyfin: 1-2GB
- Home Assistant: 500MB-1GB
- Other containers: 2-3GB
- **Total: 8-12GB / 16GB** ✓ Comfortable

**Expected CPU usage**:
- Idle: 5-10%
- Plex transcoding: 50-80% per stream
- Normal operation: 15-30%

**Expected storage I/O**:
- Low with media on NAS
- Metadata fetching: Moderate
- Transcode: High (uses local /tmp)

---

## Monitoring & Maintenance

### Daily Automated Tasks
- Watchtower checks for container updates
- Pi-hole updates blocklists
- NAS snapshots
- Plex metadata updates

### Weekly Manual Tasks
- Check Uptime Kuma dashboard
- Review Pi-hole statistics
- Check disk space (NAS, Beelink)
- Review Plex/Jellyfin activity (Tautulli)

### Monthly Tasks
- Review and test backups
- Update OS packages (Ubuntu, RPi)
- Check for service updates
- Review and clean old media
- External drive backup (off-site)

### Quarterly Tasks
- Review and update documentation
- Test disaster recovery procedures
- Evaluate new services to add
- Review security posture
- Clean up unused Docker images

---

This architecture provides a solid foundation for your Gruham home server project. Next steps:
1. Review NAS hardware options (NAS-SETUP.md)
2. Follow implementation roadmap (IMPLEMENTATION.md)
3. Deploy services using provided Docker Compose configs
