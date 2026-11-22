# 03 – docker-compose.yml

Al opsætning ligger i én fil: `/opt/medieserver/docker-compose.yml` (inde i containeren).

```bash
mkdir -p /opt/medieserver && cd /opt/medieserver
nano docker-compose.yml
```

## Filen
```yaml
name: medieserver

networks:
  media:
    driver: bridge

x-common: &common
  restart: unless-stopped
  networks: [media]
  environment:
    - PUID=1000
    - PGID=1000
    - TZ=Europe/Copenhagen

services:
  jellyfin:
    <<: *common
    container_name: jellyfin
    image: lscr.io/linuxserver/jellyfin:latest
    ports: ["8096:8096"]
    volumes:
      - /mnt/media/config/jellyfin:/config
      - /mnt/media:/data
    healthcheck:
      test: ["CMD", "curl", "-fs", "http://localhost:8096/health"]
      interval: 1m

  sonarr:
    <<: *common
    container_name: sonarr
    image: lscr.io/linuxserver/sonarr:latest
    ports: ["8989:8989"]
    volumes:
      - /mnt/media/config/sonarr:/config
      - /mnt/media:/data

  radarr:
    <<: *common
    container_name: radarr
    image: lscr.io/linuxserver/radarr:latest
    ports: ["7878:7878"]
    volumes:
      - /mnt/media/config/radarr:/config
      - /mnt/media:/data

  prowlarr:
    <<: *common
    container_name: prowlarr
    image: lscr.io/linuxserver/prowlarr:latest
    ports: ["9696:9696"]
    volumes:
      - /mnt/media/config/prowlarr:/config

  bazarr:
    <<: *common
    container_name: bazarr
    image: lscr.io/linuxserver/bazarr:latest
    ports: ["6767:6767"]
    volumes:
      - /mnt/media/config/bazarr:/config
      - /mnt/media:/data

  qbittorrent:
    <<: *common
    container_name: qbittorrent
    image: lscr.io/linuxserver/qbittorrent:latest
    ports: ["8080:8080", "6881:6881", "6881:6881/udp"]
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Copenhagen
      - WEBUI_PORT=8080
    volumes:
      - /mnt/media/config/qbittorrent:/config
      - /mnt/media:/data

  seerr:
    container_name: seerr
    restart: unless-stopped
    networks: [media]
    image: ghcr.io/seerr-team/seerr:latest
    init: true
    environment:
      - TZ=Europe/Copenhagen
    ports: ["5055:5055"]
    volumes:
      - /mnt/media/config/seerr:/app/config

  cloudflared:
    container_name: cloudflared
    restart: unless-stopped
    networks: [media]
    image: cloudflare/cloudflared:latest
    command: tunnel --no-autoupdate run
    env_file: ./cloudflared.env
```

## Start det hele
```bash
touch cloudflared.env && chmod 600 cloudflared.env
mkdir -p /mnt/media/config/seerr && chown -R 1000:1000 /mnt/media/config/seerr
docker compose up -d
docker compose ps
```
cloudflared genstarter igen og igen, indtil du har sat tokenet i kapitel 05. Det er normalt.

## Forklaring
| Del | Hvorfor |
|---|---|
| `x-common` | Fælles indstillinger skrives kun én gang. |
| `networks: media` | Tjenesterne kan finde hinanden på navn, fx `http://sonarr:8989`. |
| `restart: unless-stopped` | Starter igen efter genstart, men ikke hvis du selv har stoppet den. |
| `/mnt/media:/data` | Samme sti overalt, så hardlinks virker (se kapitel 02). |
| `container_name` | Fast navn, så fx `docker logs qbittorrent` virker. |
| `init: true` (Seerr) | Seerr stopper og genstarter pænt. Seerr kører altid som bruger 1000, så `config/seerr` skal ejes af 1000 (det klares lige før `docker compose up`). |
| `env_file` (cloudflared) | Hemmeligheden ligger i en separat fil – ikke i compose-filen. |

> qBittorrent har sin egen `environment`, fordi en liste ikke kan flettes med `x-common`.
