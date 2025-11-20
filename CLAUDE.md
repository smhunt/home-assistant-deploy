# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Docker Compose-based deployment for Home Assistant with optional companion services (MQTT, Node-RED, ESPHome). Optimized for macOS with a Linux-specific configuration available.

## Architecture

**Two Deployment Modes:**

1. **macOS (default - `docker-compose.yml`)**:
   - Uses bridge networking with explicit port mapping
   - Compatible with Docker Desktop out of the box
   - Trade-off: Device discovery (mDNS, multicast) is limited
   - No `/etc/localtime` mount (doesn't exist on macOS)

2. **Linux (`docker-compose.linux.yml`)**:
   - Uses `network_mode: host` for Home Assistant and ESPHome
   - Full device discovery support (mDNS, UPnP, multicast)
   - Mounts `/etc/localtime` for timezone
   - Requires Linux host or Docker Desktop 4.34+ with host networking enabled

**Optional Services**: Mosquitto (MQTT broker), Node-RED (automation), and ESPHome (ESP device management) are disabled by default and enabled via Docker Compose profiles.

**Data Persistence**:
- `./config/` - Home Assistant configuration files
- `./data/` - Persistent data for optional services

## Platform Considerations

**macOS Limitations:**
- Bridge networking means integrations like HomeKit, Apple TV, and other multicast-dependent features may not work
- Users needing full device discovery should use Colima or enable Docker Desktop host networking (4.34+)
- Timezone set via `TZ` environment variable instead of volume mount

## Common Commands

**macOS (default):**
```bash
docker compose up -d
docker compose down
docker compose logs -f homeassistant
docker compose --profile mqtt --profile automation up -d
```

**Linux (host networking):**
```bash
docker compose -f docker-compose.linux.yml up -d
docker compose -f docker-compose.linux.yml down
docker compose -f docker-compose.linux.yml logs -f homeassistant
```

**Update containers:**
```bash
docker compose pull && docker compose up -d
# Or for Linux:
docker compose -f docker-compose.linux.yml pull && docker compose -f docker-compose.linux.yml up -d
```

## Environment Configuration

Configuration is managed via `.env` file (created from `.env.example`):
- `TZ` - Timezone for all containers (required on macOS, optional on Linux with /etc/localtime mount)
- `COMPOSE_PROFILES` - Comma-separated list of profiles to enable (mqtt, automation, esphome)

## Files

- `docker-compose.yml` - macOS-compatible configuration (bridge networking)
- `docker-compose.linux.yml` - Linux-optimized configuration (host networking)
- `.env.example` - Environment variable template
- `config/` - Home Assistant configuration directory (excluded from git)
- `data/` - Persistent data for optional services (excluded from git)
- `.env` - User's environment configuration (excluded from git)

## Version Control

**Excluded from git (via .gitignore):**
- `config/` - Contains user-specific Home Assistant configuration, secrets, and databases
- `data/` - Contains runtime data for optional services
- `.env` - Contains environment variables

These directories are created at runtime when Home Assistant first starts. The `.env` file is created by copying `.env.example`.

## Initial Setup

1. Copy `.env.example` to `.env` and configure
2. Run `docker compose up -d`
3. Access `http://localhost:8123` for initial Home Assistant setup wizard
4. Configuration files will be automatically created in `./config/`

## Configuration Management

**Check configuration before restarting:**
```bash
docker compose exec homeassistant ha core check
```

**Restart after configuration changes:**
```bash
docker compose restart homeassistant
```

**Access Home Assistant CLI:**
```bash
docker compose exec homeassistant bash
```
