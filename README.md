# ArrStack

A Docker Compose setup for running a complete media server stack on Linux. Designed for beginners who want to get started with automated media management.

## What's Included

### Media Players
| Service | Description | Default Port |
|---------|-------------|--------------|
| [Plex](https://www.plex.tv/) | Media server for streaming movies and TV | 32400 |
| [Jellyfin](https://jellyfin.org/) | Open-source media server alternative to Plex | 8096 |

### Downloaders
| Service | Description | Default Port |
|---------|-------------|--------------|
| [SABnzbd](https://sabnzbd.org/) | Usenet downloader | 8080 |
| [qBittorrent](https://www.qbittorrent.org/) | Torrent client (routes through VPN) | 8112 |

### *Arr Apps
| Service | Description | Default Port |
|---------|-------------|--------------|
| [Prowlarr](https://prowlarr.com/) | Indexer manager for *arr apps | 9696 |
| [Radarr](https://radarr.video/) | Movie collection manager | 7878 |
| [Sonarr](https://sonarr.tv/) | TV show collection manager | 8989 |
| [Bazarr](https://www.bazarr.media/) | Subtitle downloader and manager | 6767 |

### Monitors and Utilities
| Service | Description | Default Port |
|---------|-------------|--------------|
| [Overseerr](https://overseerr.dev/) | Media request manager for Plex | 5055 |
| [Tautulli](https://tautulli.com/) | Plex monitoring and statistics | 8181 |
| [Notifiarr](https://notifiarr.com/) | Discord/notification integration for *arr apps | 5454 |
| [Cloudflare Tunnel](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/) | Secure remote access without port forwarding | - |
| [Gluetun](https://github.com/qdm12/gluetun) | VPN client for routing qBittorrent traffic | - |

## Prerequisites

- Linux server (Ubuntu, Debian, Fedora, etc.)
- [Docker](https://docs.docker.com/engine/install/)
- [Docker Compose](https://docs.docker.com/compose/install/) (v2.0+)

## Quick Start

### 1. Clone the repository

```bash
git clone https://github.com/yourusername/ArrStack.git
cd ArrStack
```

### 2. Create your media folder structure

Create a folder structure for your media. The recommended layout is:

```
/path/to/media/
├── movies/
├── television/
├── anime/
└── downloads/
    ├── complete/
    └── incomplete/
```

### 3. Create your appdata folder

Create a folder to store all container configuration files:

```bash
mkdir -p /path/to/appdata
```

### 4. Configure environment variables

Edit the `.env` file and fill in the required values:


**Required settings:**

| Variable | Description | Example |
|----------|-------------|---------|
| `PUID` | Your user ID (run `id -u`) | `1000` |
| `PGID` | Your group ID (run `id -g`) | `1000` |
| `TZ` | Your timezone | `America/New_York` |
| `MEDIA_SHARE` | Path to your media folder | `/mnt/media` |
| `APPDATA_FOLDER` | Path to your appdata folder | `/opt/appdata` |
| `PLEX_CLAIM` | Plex claim token (see below) | `claim-xxxx` |

**Getting your Plex claim token:**

1. Go to https://www.plex.tv/claim/
2. Sign in to your Plex account
3. Copy the claim token (valid for 4 minutes)

### 5. Start the stack

open a terminal in the folder where the compose and .env file are saved an run the following command:

```bash
docker compose up -d
```

### 6. Access the web interfaces

Once running, access each service at `http://your-server-ip:port`:



## Configuration

### Enabling Optional Services

Several services are commented out by default. To enable them, edit `compose.yml` and uncomment the relevant sections:

- **Jellyfin** - Alternative to Plex
- **qBittorrent + Gluetun** - Torrent client with VPN (requires ProtonVPN account)
- **Sonarr-Anime** - Separate Sonarr instance for anime
- **Notifiarr** - Discord notifications
- **Cloudflare Tunnel** - Remote access without port forwarding (Do not enable without setting up proper authentication rules and domains in cloudflare zero trust.)


### Changing Ports

All web UI ports can be customized in the `.env` file:

```bash
RADARR_WEBUI_PORT=7878        # default: 7878
SONARR_WEBUI_PORT=8989        # default: 8989
PROWLARR_WEBUI_PORT=9696      # default: 9696
# ... etc
```

### VPN Setup (for torrenting)

If you want to use qBittorrent with VPN protection:

1. Get a [ProtonVPN](https://protonvpn.com/) account
2. Uncomment the `qbittorrent` and `protonvpn` services in `compose.yml`
3. Set these variables in `.env`:
   ```
   OPENVPN_USER=your_username+pmp
   OPENVPN_PASSWORD=your_password
   ```
   Note: Add `+pmp` after your username to enable port forwarding

### Cloudflare Tunnel Setup

For secure remote access without opening ports:

1. Create a tunnel at https://one.dash.cloudflare.com/
2. Copy your tunnel token
3. Uncomment the `cloudflaretunnel` service in `compose.yml`
4. Add your token to `.env`:
   ```
   CLOUDFLARE_TUNNEL_TOKEN=your_token_here
   ```

## Recommended Setup Order

For the best experience, set up the services in this order:

1. **Plex** - Set up your media server and libraries
2. **Prowlarr** - Add your indexers (usenet and/or torrent)
3. **SABnzbd** (and/or qBittorrent) - Configure your download client
4. **Radarr** - Connect to Prowlarr and your download client, set up movie library
5. **Sonarr** - Connect to Prowlarr and your download client, set up TV library
6. **Bazarr** - Connect to Radarr and Sonarr for automatic subtitles
7. **Overseerr** - Connect to Plex, Radarr, and Sonarr for media requests
8. **Tautulli** - Connect to Plex for monitoring

## Common Commands

```bash
# Start all services
docker compose up -d

# Stop all services
docker compose down

# View logs
docker compose logs -f

# View logs for specific service
docker compose logs -f radarr

# Restart a specific service
docker compose restart radarr

# Update all containers
docker compose pull && docker compose up -d

# Update a specific container
docker compose pull radarr && docker compose up -d radarr
```

## Folder Structure

The stack uses a unified folder structure to enable hardlinks and atomic moves:

```
${MEDIA_SHARE}/
├── movies/           # Radarr movie library
├── television/       # Sonarr TV library
├── anime/            # Sonarr-Anime library (optional)
└── downloads/        # Download clients save here
    ├── complete/
    └── incomplete/
```

All containers mount `${MEDIA_SHARE}` to `/media` inside the container. This allows:
- **Hardlinks** - Completed downloads are linked instead of copied, saving disk space
- **Atomic moves** - Files move instantly within the same filesystem

## Troubleshooting

### Permission Issues

If containers can't read/write files, check your PUID and PGID:

```bash
id $USER
```

Make sure the values in `.env` match your user's UID and GID.

### Containers Can't Communicate

All containers should be on the `arrstack` network. Verify with:

```bash
docker network inspect arrstack
```

### Port Conflicts

If a port is already in use, change it in the `.env` file and restart:

```bash
docker compose down && docker compose up -d
```

## Additional Resources

### Official Documentation
- [Plex Support](https://support.plex.tv/)
- [Radarr Wiki](https://wiki.servarr.com/radarr)
- [Sonarr Wiki](https://wiki.servarr.com/sonarr)
- [Prowlarr Wiki](https://wiki.servarr.com/prowlarr)
- [Bazarr Wiki](https://wiki.bazarr.media/)
- [SABnzbd Wiki](https://sabnzbd.org/wiki/)
- [Overseerr Docs](https://docs.overseerr.dev/)

### Guides
- [TRaSH Guides](https://trash-guides.info/) - Quality profiles, custom formats, and best practices
- [Servarr Wiki](https://wiki.servarr.com/) - Documentation for all *arr apps
- [LinuxServer.io](https://docs.linuxserver.io/) - Container documentation

### Communities
- [r/Plex](https://www.reddit.com/r/PleX/)
- [r/Radarr](https://www.reddit.com/r/radarr/)
- [r/Sonarr](https://www.reddit.com/r/sonarr/)
- [r/usenet](https://www.reddit.com/r/usenet/)
