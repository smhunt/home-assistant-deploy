# Home Assistant Deploy

Docker Compose deployment configuration for Home Assistant and related services, optimized for macOS.

## Overview

This repository provides a complete Docker Compose setup for running Home Assistant along with optional companion services like MQTT, Node-RED, and ESPHome. The default configuration is macOS-compatible with explicit port mapping.

## Prerequisites

- **macOS**: Docker Desktop 4.0 or later
- **Linux**: Docker Engine 20.10 or later
- Docker Compose v2.0 or later

## Quick Start

1. Clone this repository:
```bash
git clone https://github.com/smhunt/home-assistant-deploy.git
cd home-assistant-deploy
```

2. Create your environment configuration:
```bash
cp .env.example .env
# Edit .env to set your timezone and enable optional services
```

3. Start Home Assistant:
```bash
docker compose up -d
```

4. Access Home Assistant at `http://localhost:8123`

## Directory Structure

```
.
├── config/                    # Home Assistant configuration files (not in git)
├── data/                      # Persistent data for optional services (not in git)
├── docker-compose.yml         # macOS-compatible configuration
├── docker-compose.linux.yml   # Linux-optimized configuration
├── .env.example              # Environment variable template
└── .env                      # Your environment config (create from .env.example, not in git)
```

**Note:** The `config/` and `data/` directories are excluded from version control via `.gitignore` since they contain user-specific configuration, secrets, and runtime data.

## Platform-Specific Notes

### macOS (Default Configuration)

The default `docker-compose.yml` uses **bridge networking with explicit port mapping** for maximum compatibility with Docker Desktop on macOS. This works out of the box but has some limitations:

**Limitations:**
- mDNS/Bonjour device discovery may not work
- Some integrations requiring multicast (Apple TV, HomeKit, etc.) may have issues
- Network broadcasts won't be received properly

**For Better Device Discovery on macOS:**

1. **Enable Docker Desktop Host Networking** (Docker Desktop 4.34+):
   - Go to Settings > Resources > Network
   - Enable "Use host networking"
   - Requires login to Docker Desktop
   - Then use `docker-compose.linux.yml` instead

2. **Use Colima** (recommended for Apple Silicon):
   - Lightweight alternative to Docker Desktop
   - Better stability on M1/M2 Macs
   - Supports true host networking with `socket_vmnet`
   - See: https://blog.illixion.com/2023/04/colima-home-assistant/

### Linux

For optimal performance on Linux with full device discovery support, use the Linux-specific configuration:

```bash
docker compose -f docker-compose.linux.yml up -d
```

This uses `network_mode: host` for proper mDNS, multicast, and device discovery.

## Services

### Home Assistant (Core)
The main Home Assistant instance runs on port 8123.

### Optional Services

Enable optional services by setting `COMPOSE_PROFILES` in your `.env` file:

**MQTT (Mosquitto)**
```bash
COMPOSE_PROFILES=mqtt
```
- Ports: 1883 (MQTT), 9001 (WebSocket)
- Configuration: `./config/mosquitto/mosquitto.conf`

**Node-RED**
```bash
COMPOSE_PROFILES=automation
```
- Port: 1880
- For advanced automation workflows

**ESPHome**
```bash
COMPOSE_PROFILES=esphome
```
- Port: 6052
- For managing ESP8266/ESP32 devices

**Enable multiple services:**
```bash
COMPOSE_PROFILES=mqtt,automation,esphome
```

## Common Commands

### macOS (default)
```bash
# Start all services
docker compose up -d

# Start with specific profiles
docker compose --profile mqtt --profile automation up -d

# View logs
docker compose logs -f homeassistant

# Stop services
docker compose down

# Restart Home Assistant
docker compose restart homeassistant

# Update to latest images
docker compose pull
docker compose up -d
```

### Linux (host networking)
```bash
# Use the Linux-specific compose file
docker compose -f docker-compose.linux.yml up -d

# With profiles
docker compose -f docker-compose.linux.yml --profile mqtt up -d

# Other commands work the same, just add -f docker-compose.linux.yml
docker compose -f docker-compose.linux.yml logs -f homeassistant
```

## Configuration

### Initial Setup

When you first access Home Assistant at `http://localhost:8123`, you'll be guided through an initial setup wizard where you'll:
1. Create an admin account
2. Set your location and unit system
3. Configure basic settings

After setup, the `./config/` directory will contain all your Home Assistant configuration files.

### Managing Configuration

**Main configuration file:** `./config/configuration.yaml`

Edit configuration files in the `./config/` directory to customize your setup. After making changes:

```bash
# Restart Home Assistant to apply changes
docker compose restart homeassistant

# Or check configuration before restarting
docker compose exec homeassistant ha core check
```

**Important files:**
- `configuration.yaml` - Main configuration
- `automations.yaml` - Automation definitions
- `scripts.yaml` - Script definitions
- `secrets.yaml` - Sensitive data (passwords, API keys)
- `.storage/` - Internal Home Assistant storage (don't edit directly)

## Backup & Version Control

### Backing Up Your Configuration

**Full backup (recommended):**
```bash
# Create a backup of all config and data
tar -czf ha-backup-$(date +%Y%m%d).tar.gz config/ data/
```

**Configuration-only backup:**
```bash
# Backup just the config directory
tar -czf ha-config-$(date +%Y%m%d).tar.gz config/
```

### Version Control Considerations

This repository is configured with `.gitignore` to exclude:
- `config/` - Contains user-specific settings and secrets
- `data/` - Contains runtime data for optional services
- `.env` - Contains your environment variables

**If you want to version control your Home Assistant configuration:**

1. Create a separate git repository in the `config/` directory
2. Create a custom `.gitignore` inside `config/` to exclude sensitive files:
   ```bash
   cd config/
   git init
   echo "secrets.yaml" >> .gitignore
   echo ".storage/" >> .gitignore
   echo "*.log" >> .gitignore
   echo "*.db*" >> .gitignore
   git add .
   git commit -m "Initial Home Assistant configuration"
   ```

## Troubleshooting

### macOS
- **Can't access Home Assistant**: Ensure port 8123 is not blocked by macOS firewall
- **Device discovery not working**:
  - This is expected with bridge networking on macOS
  - Try enabling Docker Desktop host networking (4.34+) or use Colima
  - Manually add devices by IP address as a workaround
- **HomeKit/Apple TV not working**: These require multicast - use Colima or configure devices manually
- **Docker Desktop freezing (M1/M2)**: Switch to Colima for better stability

### Linux
- **Can't access Home Assistant**: Check firewall rules for port 8123
- **Device discovery not working**: Ensure you're using `docker-compose.linux.yml` with host networking
- **Permission errors**: Verify config/data directories have proper permissions (usually 755)

## License

MIT
