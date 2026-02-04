# NAS Setup Guide for Gruham

## Overview

This guide will help you choose and configure a NAS (Network Attached Storage) solution for your Gruham home server. Your requirements:
- **Current**: 1x 10TB HDD (already owned)
- **Future**: Expandable to 50TB total capacity
- **Use cases**: Plex/Jellyfin media storage, backups, file shares

## Table of Contents
- [NAS Options Comparison](#nas-options-comparison)
- [Recommended Configurations](#recommended-configurations)
- [Hardware Shopping List](#hardware-shopping-list)
- [Installation Guide](#installation-guide)
- [Configuration Steps](#configuration-steps)
- [Integration with Gruham](#integration-with-gruham)

---

## NAS Options Comparison

### Option 1: Synology NAS (Recommended for Beginners)

**Pros:**
- Extremely user-friendly web interface (DSM)
- Excellent mobile apps
- Automatic updates and maintenance
- Great backup software (Hyper Backup, Active Backup)
- Can run Docker containers
- Strong community support
- Plex/Jellyfin packages available
- 5-year warranty

**Cons:**
- Higher upfront cost
- Proprietary software (though Linux-based)
- Limited hardware customization
- Smaller RAM options (upgradeable)

**Best Models for Gruham:**

| Model | Bays | Price | CPU | RAM | Notes |
|-------|------|-------|-----|-----|-------|
| **DS423+** | 4 | ~$500 | Intel Celeron J4125 | 2GB (upgradeable to 6GB) | Best value, 4K transcoding |
| **DS923+** | 4 | ~$600 | AMD Ryzen R1600 | 4GB (upgradeable to 32GB) | More powerful, expandable to 9 bays |
| DS224+ | 2 | ~$300 | Intel Celeron J4125 | 2GB | Budget option, limited expansion |
| DS1522+ | 5 | ~$700 | AMD Ryzen R1600 | 8GB | Premium, 5 bays + expansion |

**Recommendation**: **Synology DS423+**
- 4 bays: Start with 1x 10TB, add 3 more later
- Intel CPU with QuickSync (hardware transcoding)
- Reasonable price
- Can reach 40TB usable with 4x 10TB in SHR-1 (Synology Hybrid RAID)

**Expansion Path**:
```
Phase 1: 1x 10TB (10TB usable, no redundancy)
Phase 2: Add 2x 10TB (20TB usable in SHR-1, 1-drive redundancy)
Phase 3: Add 1x 10TB (30TB usable in SHR-1)
Future: Upgrade all to larger drives (4x 16TB = 48TB usable)
```

---

### Option 2: DIY TrueNAS Build (Best Performance & Flexibility)

**Pros:**
- Most powerful and flexible
- ZFS file system (best data integrity)
- ECC RAM support (error correction)
- Full control over hardware
- Unlimited expandability
- Free software (TrueNAS SCALE)
- Can run full Kubernetes clusters
- Best for tech enthusiasts

**Cons:**
- Requires building/configuring
- Steeper learning curve
- No official support (community only)
- Higher power consumption (depends on parts)
- Need to source parts

**Recommended DIY Build** (~$400-600):

**Component List:**

| Component | Option A (New) | Option B (Used/Budget) | Notes |
|-----------|----------------|----------------------|-------|
| **Case** | Fractal Design Node 804 (~$110) | Rosewill RSV-L4500U (~$140) | Hot-swap bays ideal |
| **Motherboard** | ASRock J5040-ITX (~$150) | Used Dell T30/T40 mobo (~$50) | 4+ SATA ports required |
| **CPU** | Intel Celeron J5040 (integrated) | Intel Pentium/i3 (used) | Low power, sufficient |
| **RAM** | 16GB ECC DDR4 (~$80) | 8GB ECC DDR4 (~$40) | ECC recommended for ZFS |
| **Boot Drive** | 32GB USB 3.0 (~$10) | 16GB USB (~$5) | TrueNAS boot drive |
| **PSU** | 300-450W 80+ Bronze (~$50) | Reuse existing | Efficient for 24/7 |
| **HDD Cage** | ICY DOCK 4-bay hot-swap (~$100) | DIY 3.5" bays | Hot-swap very convenient |
| **Total** | **~$500-600** | **~$235-350** | Excludes HDDs |

**Pre-built Options** (easier):
- **Used HP MicroServer Gen8/Gen10** (~$200-400 on eBay)
  - 4 bays, low power, upgradeable
  - Popular for homelab
- **Used Dell PowerEdge T30/T40** (~$250-500)
  - 4x 3.5" bays, expandable
  - More powerful, slightly higher power use

**Software**: TrueNAS SCALE (free, Linux-based, runs Docker/K8s)

**Expansion Path**:
```
Phase 1: 1x 10TB (10TB usable, no redundancy)
Phase 2: Add 2x 10TB (3x 10TB RAID-Z1 = 20TB usable, 1-drive fault)
Phase 3: Add 1x 10TB (4x 10TB RAID-Z1 = 30TB usable)
Phase 4: Add more bays or upgrade to larger drives
```

---

### Option 3: QNAP NAS (Middle Ground)

**Pros:**
- Good price/performance ratio
- Powerful hardware
- Can run VMs and containers
- Good software (QTS)
- Decent apps and ecosystem

**Cons:**
- Software less polished than Synology
- Security concerns in past (improved)
- Smaller community than Synology

**Best Models:**

| Model | Bays | Price | CPU | RAM | Notes |
|-------|------|-------|-----|-----|-------|
| **TS-464** | 4 | ~$500 | Intel Celeron N5105/N5095 | 8GB | Good value, M.2 SSD cache |
| TS-453E | 4 | ~$450 | Intel Celeron J6412 | 8GB | Budget friendly |

**Recommendation**: If going QNAP, get **TS-464**

---

### Option 4: Budget DIY with OpenMediaVault (~$200-300)

**Pros:**
- Cheapest option
- Reuse old PC or buy cheap hardware
- OpenMediaVault (OMV) is free and simple
- Debian-based (very stable)
- Good for learning

**Cons:**
- Less features than TrueNAS/Synology
- No ZFS by default (can add)
- Basic web UI
- Limited support

**Recommended Approach**:
- Use old PC with 4+ SATA ports
- Install OMV 7 (latest)
- Use SnapRAID + MergerFS for "flexible RAID"
- Perfect for media storage (Plex/Jellyfin)

**Hardware**: 
- Any old PC with 4GB+ RAM
- 4+ SATA ports
- Low power CPU (Pentium, Celeron, old i3)
- Total cost: ~$0-200 if reusing hardware

---

## Final Recommendation for Gruham

### For You: **Synology DS423+** or **DIY TrueNAS Build**

**Choose Synology DS423+ if:**
- You want it to "just work"
- You prefer web UI over CLI
- You want official support
- Time is more valuable than money
- You want excellent mobile apps

**Choose DIY TrueNAS if:**
- You enjoy building and tinkering
- You want maximum performance
- You want ECC RAM and ZFS
- You want to learn NAS/storage in-depth
- You're comfortable with Linux

**My personal recommendation for you**: **Synology DS423+**
- You're already building a complex stack (Docker, Plex, HA, etc.)
- Having NAS "just work" lets you focus on other services
- DSM is genuinely excellent software
- Easy expansion path
- Great Docker support if you want NAS to run containers too

---

## Hardware Shopping List

### Option A: Synology DS423+ Setup

**Immediate Purchase**:
- [ ] Synology DS423+ NAS (4-bay) - **~$500**
- [ ] 1x Crucial/Samsung 8GB DDR4-3200 SODIMM - **~$20** (RAM upgrade, optional)
- [ ] External 4TB USB drive for backups - **~$100**

**Already Have**:
- [x] 1x 10TB HDD (install in Bay 1)

**Future Purchases** (when ready to add redundancy):
- [ ] 2x 10TB HDDs (WD Red Plus or Seagate IronWolf) - **~$400-500**
- [ ] 1x 10TB HDD (4th drive) - **~$200-250**

**Total Initial**: ~$620
**Total Future**: ~$600-750

### Option B: DIY TrueNAS Build

**Parts List** (New Build):
- [ ] Fractal Design Node 804 case - **~$110**
- [ ] ASRock J5040-ITX motherboard (CPU included) - **~$150**
- [ ] Crucial 16GB ECC DDR4 UDIMM - **~$80**
- [ ] EVGA 450W 80+ Bronze PSU - **~$50**
- [ ] ICY DOCK 4-bay hot-swap cage - **~$100**
- [ ] 32GB USB 3.0 flash drive - **~$10**
- [ ] External 4TB USB drive for backups - **~$100**

**Already Have**:
- [x] 1x 10TB HDD

**Total Initial**: ~$600

**OR Parts List** (Used/Budget):
- [ ] Used HP MicroServer Gen10 - **~$300** (includes case, mobo, CPU, PSU)
- [ ] 16GB ECC DDR4 RAM - **~$80**
- [ ] 16GB USB flash drive - **~$5**
- [ ] External 4TB USB drive - **~$100**

**Total Initial (Used)**: ~$485

---

## Recommended Hard Drives

### For NAS Use (CMR Drives Required)

**⚠️ Avoid SMR drives for NAS/RAID!** (Seagate Barracuda, WD Blue)

**Recommended Brands/Models**:

| Brand | Model | Capacity | Price (approx) | Warranty | Notes |
|-------|-------|----------|----------------|----------|-------|
| **WD Red Plus** | WD100EFBX | 10TB | ~$220 | 3 years | CMR, NAS-optimized |
| **Seagate IronWolf** | ST10000VN0008 | 10TB | ~$220 | 3 years | CMR, NAS-rated |
| WD Red Pro | WD101KFBX | 10TB | ~$280 | 5 years | Higher performance |
| Toshiba N300 | HDWG11A | 10TB | ~$210 | 3 years | Good value |

**For 50TB Future Expansion** (better value):
| Model | Capacity | Price | Notes |
|-------|----------|-------|-------|
| WD Red Plus 14TB | 14TB | ~$280 | Better $/TB ratio |
| Seagate IronWolf 16TB | 16TB | ~$320 | Even better value |
| WD Red Plus 18TB | 18TB | ~$350 | Best $/TB |

**Recommendation**: 
- **Now**: Use existing 10TB
- **Phase 2**: Buy 2x WD Red Plus 10TB (~$440)
- **Future**: Upgrade to 14-18TB drives for better capacity

---

## Installation Guide

### Synology DS423+ Setup

#### Step 1: Physical Installation

1. **Unbox and inspect** NAS
2. **Install RAM upgrade** (optional):
   - Power off NAS
   - Remove bottom cover (4 screws)
   - Install 8GB SODIMM in empty slot (2+8=10GB total)
   - Replace cover
3. **Install your 10TB drive**:
   - Open drive bay 1
   - Slide drive into tray (no screws needed with tool-less design)
   - Push tray into bay until it clicks
4. **Connect network**:
   - Ethernet cable from NAS to Eero router
   - Plug in power adapter
   - Press power button

#### Step 2: Initial Software Setup

1. **Find your NAS**:
   - Go to http://find.synology.com from computer on same network
   - Or use Synology Assistant app
2. **Install DSM**:
   - Click "Connect" when NAS is found
   - Download and install latest DSM (DiskStation Manager)
   - Wait 10-15 minutes for installation
3. **Create admin account**:
   - Set strong admin password (save in password manager)
   - Create QuickConnect ID (for remote access)
4. **Configure storage**:
   - Create Storage Pool:
     - Storage Manager → Create → Storage Pool
     - Single drive (no RAID yet, only 1 drive)
   - Create Volume:
     - Volume 1, Btrfs file system
     - Allocate all available space
5. **Create shared folders**:
   ```
   /media       (for Plex/Jellyfin)
   /backups     (for server backups)
   /shares      (for general files)
   ```

#### Step 3: Enable Services

1. **SMB/CIFS** (Windows file sharing):
   - Control Panel → File Services → SMB → Enable
2. **NFS** (Linux file sharing, better for Plex):
   - Control Panel → File Services → NFS → Enable
   - Enable NFSv4
3. **Configure NFS permissions for /media**:
   - Control Panel → Shared Folder → media → Edit → NFS Permissions
   - Add rule:
     - Server: `192.168.4.100` (Beelink IP)
     - Privilege: Read/Write
     - Squash: Map all users to admin
     - Security: sys

#### Step 4: Configure Snapshots

1. **Snapshot Replication**:
   - Open Snapshot Replication package
   - Create snapshot schedule:
     - Shared folder: media
     - Schedule: Hourly (keep 24), Daily (keep 7), Weekly (keep 4)
   - Repeat for backups and shares folders

#### Step 5: Set Up Backups

1. **Install Hyper Backup**:
   - Package Center → Search "Hyper Backup" → Install
2. **Configure backup to USB drive**:
   - Plug in external 4TB USB drive
   - Hyper Backup → Create task
   - Destination: External drive (USB)
   - Folders to backup: Critical data, configs
   - Schedule: Daily at 2 AM
   - Encryption: Yes (AES-256, strong password)

---

### TrueNAS SCALE Setup

#### Step 1: Prepare Installation Media

1. **Download TrueNAS SCALE**:
   - Visit: https://www.truenas.com/download-truenas-scale/
   - Download latest .iso
2. **Create bootable USB**:
   - Use Rufus (Windows) or Balena Etcher
   - Flash ISO to USB drive (separate from boot drive)
3. **Insert USB boot drive** (16-32GB) that will become OS drive

#### Step 2: Install TrueNAS

1. **Boot from installation media**:
   - Plug installation USB into server
   - Boot from USB (check BIOS boot order)
2. **Install TrueNAS SCALE**:
   - Select "Install/Upgrade"
   - Choose destination: Your 16-32GB USB drive (will be OS drive)
   - Set root password (strong password)
   - Wait for installation (5-10 minutes)
   - Remove installation media, reboot
3. **First boot**:
   - TrueNAS boots from USB
   - Note the IP address shown (e.g., 192.168.4.102)

#### Step 3: Initial Web Configuration

1. **Access web UI**:
   - From browser: http://192.168.4.102
   - Login: root / [your password]
2. **Network configuration**:
   - Network → Interfaces → Edit enp0s1 (or similar)
   - Set static IP: `192.168.4.102`
   - Gateway: `192.168.4.1`
   - DNS: `192.168.4.101` (Pi-hole)
3. **Set timezone**:
   - System → General → Timezone
   - Select your timezone

#### Step 4: Create Storage Pool

**Single Drive (Current)**:
1. **Storage → Pools → Create Pool**
2. **Name**: `pool1`
3. **Layout**: Stripe (no redundancy, 1 drive)
4. **Add disk**: Select your 10TB drive
5. **Create** (warning about no redundancy - Accept)

**Future: RAID-Z1 (When you have 3-4 drives)**:
1. Add more drives to server
2. Storage → Pools → Expand pool1 OR Create pool2
3. Layout: RAID-Z1 (1-disk parity)
4. Add all drives
5. Create

#### Step 5: Create Datasets

1. **Storage → Pools → pool1 → Add Dataset**:
   ```
   Dataset name: media
   Compression: LZ4 (on)
   Snapshots: Enabled
   Quota: None
   ```
2. **Repeat for**:
   - `backups` dataset
   - `shares` dataset

#### Step 6: Configure Sharing

**SMB Shares**:
1. **Sharing → Windows (SMB) Shares → Add**:
   - Path: `/mnt/pool1/media`
   - Name: `media`
   - Purpose: Default share parameters
   - Enable: Yes
2. **Repeat for** backups and shares

**NFS Shares** (recommended for Plex):
1. **Sharing → Unix (NFS) Shares → Add**:
   - Path: `/mnt/pool1/media`
   - Networks: `192.168.4.100/32` (Beelink only)
   - Mapall User: root
   - Mapall Group: root
2. **Repeat for** backups

#### Step 7: Enable Snapshots

1. **Data Protection → Periodic Snapshot Tasks → Add**:
   - Dataset: `pool1/media`
   - Recursive: Yes
   - Schedule:
     - Hourly: Keep 24
     - Daily: Keep 7
     - Weekly: Keep 4
   - Enable: Yes
2. **Repeat for other datasets**

---

## Configuration for Gruham Integration

### Network Configuration

#### Set Static IP on Eero Router

1. **Open Eero app**
2. **Network Settings → Advanced → DHCP & NAT**
3. **Add Reservation**:
   - Device: Select your NAS (Synology/TrueNAS)
   - IP Address: `192.168.4.102`
   - Save

**Verify**: NAS should now always have `192.168.4.102`

#### Configure DNS (Pi-hole)

1. **SSH into Raspberry Pi** (or use Pi-hole web UI)
2. **Add custom DNS entry**:
   ```bash
   # Edit local DNS
   sudo nano /etc/pihole/custom.list
   
   # Add line:
   192.168.4.102 nas.local
   
   # Restart DNS
   pihole restartdns
   ```
3. **Test**: 
   ```bash
   ping nas.local
   # Should resolve to 192.168.4.102
   ```

---

### Mount NAS on Beelink

#### Install NFS Client on Beelink

```bash
# SSH into Beelink
ssh your-user@192.168.4.100

# Install NFS client
sudo apt update
sudo apt install nfs-common -y

# Create mount points
sudo mkdir -p /mnt/nas/{media,backups,shares}
```

#### Configure Automatic Mounting

**Option A: /etc/fstab (Automatic on boot)**

```bash
# Edit fstab
sudo nano /etc/fstab

# Add lines (adjust if using Synology vs TrueNAS paths):
# For Synology:
192.168.4.102:/volume1/media    /mnt/nas/media    nfs    defaults,_netdev,rw,hard,intr    0 0
192.168.4.102:/volume1/backups  /mnt/nas/backups  nfs    defaults,_netdev,rw,hard,intr    0 0

# For TrueNAS:
192.168.4.102:/mnt/pool1/media    /mnt/nas/media    nfs    defaults,_netdev,rw,hard,intr    0 0
192.168.4.102:/mnt/pool1/backups  /mnt/nas/backups  nfs    defaults,_netdev,rw,hard,intr    0 0

# Save and exit (Ctrl+X, Y, Enter)

# Test mount
sudo mount -a

# Verify
df -h | grep nas
```

**Option B: Systemd Mount (More robust)**

```bash
# Create mount unit
sudo nano /etc/systemd/system/mnt-nas-media.mount

# Content:
[Unit]
Description=Mount NAS Media Share
Requires=network-online.target
After=network-online.target

[Mount]
What=192.168.4.102:/volume1/media
Where=/mnt/nas/media
Type=nfs
Options=defaults,_netdev,rw,hard,intr

[Install]
WantedBy=multi-user.target

# Save and create for backups too
# Then enable:
sudo systemctl daemon-reload
sudo systemctl enable mnt-nas-media.mount
sudo systemctl start mnt-nas-media.mount
```

#### Set Permissions

```bash
# Make docker user owner (adjust UID/GID as needed)
sudo chown -R 1000:1000 /mnt/nas/media
sudo chmod -R 755 /mnt/nas/media

# Or for Plex specifically (runs as plex user):
sudo chown -R 1000:1000 /mnt/nas/media
```

---

### Test NAS Integration

```bash
# Test write
echo "test" | sudo tee /mnt/nas/media/test.txt

# Test read
cat /mnt/nas/media/test.txt

# Check speed (optional)
sudo dd if=/dev/zero of=/mnt/nas/media/testfile bs=1M count=1000
# Should see ~100-120 MB/s on gigabit

# Clean up
sudo rm /mnt/nas/media/testfile /mnt/nas/media/test.txt
```

---

## Organizing Media Library

### Recommended Folder Structure

```
/mnt/nas/media/
├── movies/
│   ├── Movie Title (Year)/
│   │   ├── Movie Title (Year).mkv
│   │   └── Movie Title (Year)-poster.jpg
│   └── Another Movie (2023)/
│       └── Another Movie (2023).mp4
├── tv/
│   ├── TV Show Name/
│   │   ├── Season 01/
│   │   │   ├── TV Show - S01E01 - Episode Title.mkv
│   │   │   └── TV Show - S01E02 - Episode Title.mkv
│   │   └── Season 02/
│   │       └── TV Show - S02E01 - Episode Title.mkv
│   └── Another Show/
│       └── Season 01/
├── music/
│   ├── Artist Name/
│   │   ├── Album Name (Year)/
│   │   │   ├── 01 - Track Name.flac
│   │   │   └── cover.jpg
│   │   └── Another Album (Year)/
│   └── Another Artist/
└── photos/
    ├── 2024/
    │   ├── 01-January/
    │   └── 02-February/
    └── 2023/
```

**This structure works perfectly with**:
- Plex (automatic metadata matching)
- Jellyfin (automatic scraping)
- Kodi
- Emby

---

## Backup Strategy for NAS

### Tier 1: On-NAS Snapshots (Synology/TrueNAS)
- **Frequency**: Hourly, Daily, Weekly
- **Retention**: 24 hours, 7 days, 4 weeks
- **Purpose**: Protection against accidental deletion, ransomware
- **Storage**: Same NAS (no protection against hardware failure)

### Tier 2: External USB Backup
- **Frequency**: Daily
- **Retention**: Full backups, 30-day rotation
- **Purpose**: Hardware failure protection
- **Storage**: 4TB external USB drive
- **Encrypted**: Yes (AES-256)

### Tier 3: Cloud Backup (Optional, for critical data only)
- **Frequency**: Weekly
- **Retention**: 90 days
- **Purpose**: Off-site disaster protection
- **Storage**: Backblaze B2 (~$6/TB/month) or Google Drive
- **What**: Configs, photos, documents (NOT media files)
- **Encrypted**: Yes

### What NOT to Backup
- Media files (movies/TV) - Too large, re-downloadable
- Plex/Jellyfin metadata - Can be regenerated
- Temporary files

### What MUST Backup
- Docker configs (`/opt/gruham/`)
- Home Assistant config
- Plex/Jellyfin watch history and preferences
- Personal photos and documents
- NAS configuration files

---

## Performance Tuning

### NFS Mount Options

**For Plex/Jellyfin** (read-heavy):
```bash
192.168.4.102:/volume1/media /mnt/nas/media nfs rw,hard,intr,rsize=8192,wsize=8192,timeo=14 0 0
```

**Explanation**:
- `rw`: Read-write access
- `hard`: Wait for NAS recovery if it goes down (prevents corruption)
- `intr`: Allow interrupts (can cancel if needed)
- `rsize/wsize=8192`: Read/write buffer size (can try 16384 or 32768)
- `timeo=14`: Timeout in deciseconds (1.4 seconds)

### Synology DSM Optimizations

1. **Enable SMB Multi-Channel** (if you add 2nd NIC later):
   - Control Panel → File Services → SMB → Advanced → Enable SMB3 multi-channel
2. **Disable unused services**:
   - Control Panel → Task Scheduler → Disable unused scheduled tasks
3. **Enable caching** (if adding SSD cache later):
   - Storage Manager → SSD Cache → Create cache
   - Read-write cache for best performance

### TrueNAS Optimizations

1. **ZFS ARC tuning**:
   - System → Tunables → Add
   - `vfs.zfs.arc_max`: Set to 50-75% of RAM
   - Example: 8GB RAM → 4-6GB ARC
2. **Enable L2ARC** (if adding SSD cache):
   - Add SSD to pool as cache device
3. **Scrub schedule**:
   - Data Protection → Scrub Tasks
   - Monthly scrub of pool1

---

## Expansion Guide

### Adding Drives to Synology (SHR-1)

**When you get 2 more 10TB drives**:

1. **Install drives**:
   - Power down NAS
   - Install drives in bays 2 and 3
   - Power on
2. **Expand Storage Pool**:
   - Storage Manager → Storage Pool 1 → Action → Add Drive
   - Select new drives
   - Change RAID type: SHR-1 (or RAID 5)
   - Confirm (⚠️ this will take 12-24 hours!)
3. **Wait for expansion**:
   - Monitor progress in Storage Manager
   - NAS remains usable during expansion
   - Do NOT power off during this process
4. **Verify**:
   - Should now show 20TB usable (with 3x 10TB in SHR-1)

**Adding 4th drive later**:
- Same process
- Will expand to 30TB usable

### Adding Drives to TrueNAS (RAID-Z1)

**Current state**: 1x 10TB stripe (no redundancy)

**When you get 2-3 more drives**:

**Option A: Create new pool** (recommended):
1. **Install new drives** in server
2. **Storage → Create Pool**:
   - Name: `pool2`
   - Layout: RAID-Z1
   - Disks: Select 3-4 drives (including your original 10TB)
   - Warning: ⚠️ This will ERASE all drives!
3. **Migrate data**:
   - Before creating pool2, backup data from pool1
   - Create pool2
   - Restore data to pool2
   - Delete pool1

**Option B: Expand existing pool** (complex):
- Not easily done in ZFS
- Better to backup, recreate, restore

---

## Troubleshooting

### NAS Not Accessible

```bash
# Check if NAS is reachable
ping 192.168.4.102

# Check if NFS service is running (from Beelink)
showmount -e 192.168.4.102
# Should list exports

# Check mount status
df -h | grep nas
mount | grep nas

# Try manual mount
sudo mount -t nfs 192.168.4.102:/volume1/media /mnt/nas/media

# Check NAS logs (Synology)
# DSM → Log Center → Connection

# Check NAS logs (TrueNAS)
# System → Advanced → Shell
journalctl -u nfs-server -f
```

### Slow Transfer Speeds

```bash
# Test network speed to NAS
iperf3 -c 192.168.4.102
# Should see ~900+ Mbps on gigabit

# Test disk write speed
dd if=/dev/zero of=/mnt/nas/media/testfile bs=1M count=1000 oflag=direct
# Should see 100-120 MB/s

# Check NFS stats
nfsstat -c
```

### Permission Issues

```bash
# Check ownership
ls -la /mnt/nas/media

# Fix permissions (run on Beelink)
sudo chown -R 1000:1000 /mnt/nas/media

# Or match Plex user
id plex
# Note UID/GID, then:
sudo chown -R 998:998 /mnt/nas/media  # Example if plex is 998
```

---

## Next Steps

1. ✅ Choose your NAS solution (Synology DS423+ recommended)
2. ✅ Order hardware
3. ✅ Install and configure NAS
4. ✅ Mount NAS on Beelink
5. ✅ Organize media library
6. ➡️ Continue to [Implementation Roadmap](IMPLEMENTATION.md)
7. ➡️ Deploy Docker services

---

## Additional Resources

- Synology Community: https://community.synology.com/
- TrueNAS Forums: https://www.truenas.com/community/
- r/HomeServer: https://reddit.com/r/homeserver
- r/DataHoarder: https://reddit.com/r/datahoarder
- r/Synology: https://reddit.com/r/synology
- r/TrueNAS: https://reddit.com/r/truenas

**Need help?** Check the troubleshooting section or ask in the homelab communities above!
