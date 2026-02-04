# Gruham Network Diagrams

## Opening the Excalidraw Diagram

### Option 1: Excalidraw Web (Recommended)

1. Go to https://excalidraw.com
2. Click "Open" (folder icon in top left)
3. Select `network-architecture.excalidraw` from this directory
4. Edit and export as needed

### Option 2: VS Code with Excalidraw Extension

1. Install "Excalidraw" extension in VS Code
2. Open `network-architecture.excalidraw`
3. Edit directly in VS Code

## What's in the Diagram

The network architecture diagram shows:

- **Internet Connection** → Xfinity Router → Eero Mesh Network
- **Workstation** (Main network, runs Claw)
- **Beelink SER3** (Primary server with all Docker services)
- **Raspberry Pi** (Pi-hole DNS + Uptime Kuma)
- **NAS** (Storage server - Synology/TrueNAS)
- **Client Devices** (Laptops, phones, iPads on WiFi)
- **Cloudflare Tunnel** (Secure remote access, no port forwarding)

## Network Details

### IP Addresses
- Xfinity Router: `192.168.1.1` (main network)
- Eero Router: `192.168.4.1` (home devices)
- Workstation: `192.168.1.10` (main network)
- Beelink: `192.168.4.100` (Eero network)
- Raspberry Pi: `192.168.4.101` (Eero network)
- NAS: `192.168.4.102` (Eero network)

### Connections
- **Ethernet** (preferred): Beelink, NAS, Raspberry Pi, Workstation
- **WiFi**: Laptops, phones, tablets

### Data Flows
1. **Internet → Cloudflare → Beelink**: Remote access to services
2. **NAS → Beelink**: Media files via NFS/SMB
3. **All devices → Pi-hole**: DNS queries with ad blocking
4. **Clients → Beelink**: Local streaming and service access

## Exporting Diagrams

In Excalidraw:
- **PNG**: File → Export image → PNG (for documentation)
- **SVG**: File → Export image → SVG (for high quality)
- **PDF**: Export as PNG, then convert to PDF

## Creating Additional Diagrams

You can create additional diagrams for:
- **Docker container architecture** (service relationships)
- **Storage layout** (NAS pool structure)
- **Backup strategy** (data flow for backups)
- **Future K3s cluster** (when migrating to Kubernetes)

Use the same style and color scheme:
- Infrastructure: Blue (#1565c0)
- Networking: Teal (#00897b)
- Media services: Orange (#f57c00)
- Automation: Green (#43a047)
- Security: Red (#c62828)
- Development: Purple (#6a1b9a)

## Tips

- Use the reference architecture images you provided as inspiration
- Keep diagrams up to date as you add services
- Screenshot the diagram and include in your documentation
- Share with the homelab community!

---

**Next Steps**: Return to [Implementation Guide](../docs/IMPLEMENTATION.md)
