# 06 – Backup og drift

## Backup af hele containeren
Brug Proxmox' egen backup af container 101 (se kapitel 07 i Proxmox-guiden).
Den tager **ikke** bind-mountet `/mnt/media` med – hverken mediefilerne eller `config/`.
Derfor skal indstillinger og databaser tages med herunder.

## Indbygget backup i Sonarr, Radarr og Prowlarr
Den nemmeste vej: **System → Backup**. De laver selv en backup hver uge,
som gemmes i deres `config`-mappe.

## Backup af databaserne
Tjenesterne gemmer alt i **SQLite-databaser** under `/mnt/media/config`.
Kopier aldrig en database, mens den kører – brug SQLite's egen backup:
```bash
apt install -y sqlite3
mkdir -p /mnt/media/backup
sqlite3 /mnt/media/config/sonarr/sonarr.db ".backup '/mnt/media/backup/sonarr-$(date +%F).db'"
sqlite3 /mnt/media/backup/sonarr-$(date +%F).db "PRAGMA integrity_check;"   # skal svare "ok"
```
Gør det samme for `radarr/radarr.db`, `prowlarr/prowlarr.db`, `bazarr/db/bazarr.db`,
`seerr/db/db.sqlite3` og Jellyfins `jellyfin.db` (find den med `find /mnt/media/config/jellyfin -name "*.db"`).
Læg det i et script og kør det hver nat med `cron`.

**Kopier derefter `/mnt/media/backup` til en anden disk** – eller ud af huset.
En backup på samme disk hjælper ikke, hvis disken dør.

## Opdatering
```bash
cd /opt/medieserver
docker compose pull
docker compose up -d
docker image prune -f        # ryd gamle images op
```
- Tag først en **backup af databaserne** (se ovenfor). Et snapshot af containeren
  (`pct snapshot 101 foer-opdatering`) tager ikke `/mnt/media/config` med.
- **Lås Jellyfins version** (fx `image: lscr.io/linuxserver/jellyfin:<version>`), hvis en opdatering
  kan ændre databasen. Så opdaterer du kun, når du selv vil.

## Hold øje
```bash
docker compose ps            # kører alt, og er det "healthy"?
docker logs -f sonarr        # se loggen for én tjeneste
df -h /mnt/media             # er disken ved at blive fuld?
```
