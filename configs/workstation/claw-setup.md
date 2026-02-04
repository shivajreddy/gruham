# Claw Setup on Workstation

## Overview
Claw is a Claude-powered AI assistant that can run on your powerful workstation. This guide covers installation and integration with Gruham.

## Prerequisites
- Windows 11 Pro (your workstation)
- WSL2 installed (Windows Subsystem for Linux)
- Docker Desktop (optional, or native Docker in WSL2)
- Anthropic API key (for Claude)

## Installation Options

### Option 1: Native Windows (Recommended for Workstation)

#### Step 1: Install Prerequisites

```powershell
# Open PowerShell as Administrator

# Install WSL2 if not already installed
wsl --install

# Install Python (if not already installed)
winget install Python.Python.3.12

# Verify installation
python --version
```

#### Step 2: Install Claw

```powershell
# Create directory for Claw
mkdir C:\Users\smpl\claw
cd C:\Users\smpl\claw

# Clone Claw repository (replace with actual repo when available)
git clone https://github.com/anthropics/claw.git .

# Create virtual environment
python -m venv venv

# Activate virtual environment
.\venv\Scripts\Activate.ps1

# Install dependencies
pip install -r requirements.txt
```

#### Step 3: Configure Claw

```powershell
# Create .env file
cp .env.example .env

# Edit .env with your settings
notepad .env
```

**.env contents**:
```env
# Anthropic API Key
ANTHROPIC_API_KEY=sk-ant-XXXXXXXXXXXXXXXXXXXXX

# Model selection
CLAUDE_MODEL=claude-3-5-sonnet-20241022

# Server settings
PORT=3030
HOST=0.0.0.0

# Integration with Gruham services
PLEX_URL=http://192.168.4.100:32400
PLEX_TOKEN=YOUR_PLEX_TOKEN

HOME_ASSISTANT_URL=http://192.168.4.100:8123
HOME_ASSISTANT_TOKEN=YOUR_HA_TOKEN

# Optional: Logging
LOG_LEVEL=INFO
```

#### Step 4: Run Claw

```powershell
# Start Claw
python -m claw.server

# Or run in background with PM2 (if using Node.js version)
npm install -g pm2
pm2 start claw
pm2 save
```

#### Step 5: Access Claw

- **Local**: http://localhost:3030
- **From other devices**: http://192.168.1.10:3030 (workstation IP on main network)

---

### Option 2: Docker Container (Alternative)

#### docker-compose.yml for Claw

```yaml
version: '3.8'

services:
  claw:
    image: anthropic/claw:latest  # Replace with actual image
    container_name: claw
    restart: unless-stopped
    ports:
      - "3030:3030"
    environment:
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
      - CLAUDE_MODEL=claude-3-5-sonnet-20241022
      - PLEX_URL=http://192.168.4.100:32400
      - PLEX_TOKEN=${PLEX_TOKEN}
      - HOME_ASSISTANT_URL=http://192.168.4.100:8123
      - HOME_ASSISTANT_TOKEN=${HOME_ASSISTANT_TOKEN}
    volumes:
      - ./claw/data:/data
      - ./claw/config:/config
    networks:
      - gruham_network

networks:
  gruham_network:
    external: true
```

**Deploy**:
```powershell
docker compose up -d
```

---

## Integration with Gruham Services

### 1. Expose Claw via Cloudflare Tunnel

Add to your Cloudflare Tunnel configuration:

```yaml
tunnel: YOUR_TUNNEL_ID
credentials-file: /etc/cloudflared/cert.json

ingress:
  # ... other services ...
  
  - hostname: claw.yourdomain.com
    service: http://192.168.1.10:3030  # Workstation IP
    originRequest:
      noTLSVerify: true
  
  # ... catch-all ...
```

Or in Cloudflare Zero Trust Dashboard:
1. Go to your tunnel
2. Add Public Hostname
3. Subdomain: `claw`
4. Domain: `yourdomain.com`
5. Service: `HTTP://192.168.1.10:3030`

### 2. Add Claw to Homepage Dashboard

Edit `/opt/gruham/homepage/config/services.yaml` on Beelink:

```yaml
- AI & Development:
    - Claw:
        icon: anthropic.png
        href: https://claw.yourdomain.com
        description: Claude AI Assistant
        server: workstation
        container: claw
        widget:
          type: claw
          url: http://192.168.1.10:3030
```

### 3. Home Assistant Integration

Create a REST command in Home Assistant to interact with Claw:

Edit `/opt/gruham/homeassistant/config/configuration.yaml`:

```yaml
rest_command:
  ask_claw:
    url: "http://192.168.1.10:3030/api/ask"
    method: POST
    content_type: "application/json"
    payload: '{"question": "{{ question }}"}'

# Example automation
automation:
  - alias: "Ask Claw about system status"
    trigger:
      - platform: time
        at: "09:00:00"
    action:
      - service: rest_command.ask_claw
        data:
          question: "What's the status of all Gruham services?"
```

---

## Getting API Keys

### Anthropic API Key
1. Go to https://console.anthropic.com/
2. Sign up / Log in
3. Go to API Keys section
4. Create new API key
5. Copy and save securely (you won't see it again)

### Plex Token
```bash
# Method 1: From web interface
# 1. Play any item in Plex Web
# 2. Open browser dev tools (F12)
# 3. Go to Network tab
# 4. Look for any request, check query params for X-Plex-Token

# Method 2: From command line
curl -X POST \
  'https://plex.tv/users/sign_in.json' \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -d 'user[login]=YOUR_EMAIL&user[password]=YOUR_PASSWORD'

# Response will include authToken
```

### Home Assistant Token
1. In Home Assistant UI: http://192.168.4.100:8123
2. Click your profile (bottom left)
3. Scroll to "Long-Lived Access Tokens"
4. Click "Create Token"
5. Name: "Claw Integration"
6. Copy token and save

---

## Firewall Configuration (Windows)

Allow Claw to accept connections:

```powershell
# Run as Administrator
New-NetFirewallRule -DisplayName "Claw Server" -Direction Inbound -Port 3030 -Protocol TCP -Action Allow
```

---

## Auto-start Claw on Windows Boot

### Option 1: Task Scheduler

1. Open Task Scheduler
2. Create Basic Task
3. Name: "Start Claw"
4. Trigger: "When the computer starts"
5. Action: "Start a program"
6. Program: `C:\Users\smpl\claw\venv\Scripts\python.exe`
7. Arguments: `-m claw.server`
8. Start in: `C:\Users\smpl\claw`

### Option 2: NSSM (Non-Sucking Service Manager)

```powershell
# Download NSSM
winget install NSSM

# Install Claw as Windows service
nssm install Claw "C:\Users\smpl\claw\venv\Scripts\python.exe"
nssm set Claw AppParameters "-m claw.server"
nssm set Claw AppDirectory "C:\Users\smpl\claw"
nssm set Claw DisplayName "Claw AI Assistant"
nssm set Claw Description "Claude-powered AI assistant for Gruham"
nssm set Claw Start SERVICE_AUTO_START

# Start service
nssm start Claw

# Check status
nssm status Claw
```

---

## Usage Examples

### Web Interface

Access: https://claw.yourdomain.com

**Example prompts**:
- "Show me Plex server statistics"
- "What movies were added to Plex this week?"
- "Turn off bedroom lights" (via Home Assistant)
- "What's the current temperature in the living room?"
- "List all running Docker containers on Beelink"
- "When was the last backup?"

### API Usage

```bash
# Ask a question
curl -X POST http://192.168.1.10:3030/api/ask \
  -H "Content-Type: application/json" \
  -d '{"question": "What services are running on Gruham?"}'

# Control Home Assistant device
curl -X POST http://192.168.1.10:3030/api/ha/control \
  -H "Content-Type: application/json" \
  -d '{"entity_id": "light.bedroom", "state": "off"}'

# Get Plex stats
curl http://192.168.1.10:3030/api/plex/stats
```

---

## Monitoring

Add Claw to Uptime Kuma:

1. Access Uptime Kuma: http://192.168.4.101:3001
2. Add New Monitor
3. Monitor Type: HTTP(s)
4. Friendly Name: Claw AI
5. URL: http://192.168.1.10:3030/health
6. Heartbeat Interval: 60 seconds
7. Save

---

## Security Considerations

1. **API Key Security**:
   - Never commit .env files to git
   - Use Windows Credential Manager for sensitive data
   - Rotate API keys periodically

2. **Network Access**:
   - Claw runs on main network (Xfinity) separate from servers (Eero)
   - Only expose via Cloudflare Tunnel, not direct port forwarding
   - Consider adding authentication layer (OAuth, basic auth)

3. **Rate Limiting**:
   - Monitor Anthropic API usage (has costs)
   - Set up usage alerts in Anthropic console
   - Implement request rate limiting if needed

---

## Troubleshooting

### Claw won't start

```powershell
# Check Python installation
python --version

# Check dependencies
pip list

# Check logs
type logs\claw.log

# Test API key
curl https://api.anthropic.com/v1/models \
  -H "x-api-key: YOUR_API_KEY" \
  -H "anthropic-version: 2023-06-01"
```

### Can't access from other devices

```powershell
# Check if service is running
netstat -an | findstr 3030

# Check firewall
Get-NetFirewallRule -DisplayName "Claw Server"

# Test from Beelink
# SSH to Beelink, then:
curl http://192.168.1.10:3030/health
```

### High API costs

1. Check usage: https://console.anthropic.com/usage
2. Implement caching for repeated queries
3. Use smaller model for simple tasks (claude-3-haiku)
4. Set monthly budget limits

---

## Future Enhancements

- [ ] Voice interface (Whisper API for STT)
- [ ] Integration with Plex "On Deck" recommendations
- [ ] Automated media library organization
- [ ] Smart home routine suggestions
- [ ] Proactive system health monitoring
- [ ] Natural language Docker container management

---

## Notes

- Claw is best run on workstation due to powerful hardware
- Not required 24/7 (workstation on-demand)
- Can move to Beelink if you want 24/7 availability
- Consider K3s deployment if migrating to Kubernetes later

---

**Next**: Return to [Implementation Guide](../IMPLEMENTATION.md)
