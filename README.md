# Medieserver-guide: din egen "Netflix" derhjemme

En dansk trin-for-trin-guide til at bygge en automatisk medieserver med Docker
i en LXC-container på Proxmox. Den bygger på mit eget homelab, men alle navne og
IP-adresser er eksempler.

**Forudsætning:** En kørende Proxmox-server. Se [proxmox-guide](https://github.com/TechNetDK/proxmox-guide).

## Sådan virker det

```
En bruger ønsker en film eller serie i Seerr
            │
Radarr (film) / Sonarr (serier) finder den via Prowlarr
            │
Download-klienten (qBittorrent) henter den
            │
Filen flyttes og omdøbes automatisk
            │
Bazarr henter undertekster
            │
Jellyfin viser den på TV, mobil og PC
```

## Tjenesterne

| Tjeneste | Opgave | Port |
|---|---|---|
| Jellyfin | Afspiller film og serier | 8096 |
| Seerr | Brugerne ønsker film og serier | 5055 |
| Sonarr | Holder styr på serier | 8989 |
| Radarr | Holder styr på film | 7878 |
| Prowlarr | Samler kilder til Sonarr og Radarr | 9696 |
| Bazarr | Undertekster | 6767 |
| qBittorrent | Download-klient | 8080 |
| cloudflared | Adgang udefra uden åbne porte | – |
| Gluetun | VPN til download-klienten (valgfri) | 8080 (qBittorrents webgrænseflade, når VPN bruges) |

## Indhold

| # | Kapitel |
|---|---|
| 01 | [Container og Docker](01-container-og-docker.md) |
| 02 | [Mapper og rettigheder](02-mapper-og-rettigheder.md) |
| 03 | [docker-compose.yml](03-docker-compose.md) |
| 04 | [Sæt tjenesterne sammen](04-saet-tjenesterne-sammen.md) |
| 05 | [Adgang udefra](05-adgang-udefra.md) |
| 06 | [Backup og drift](06-backup-og-drift.md) |
| 07 | [Fejl jeg har ramt, og løsningerne](07-fejl-og-loesninger.md) |
| 08 | [VPN til download-klienten](08-vpn.md) |

> Brug kun indhold, du har ret til – fx dine egne film og serier eller lovlige kilder.
