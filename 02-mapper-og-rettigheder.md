# 02 – Mapper og rettigheder

## Mappestruktur
Læg det hele under **én** mappe på **samme disk**:
```
/mnt/media/
├── film/          # færdige film
├── serier/        # færdige serier
├── downloads/     # download-klientens mappe
└── config/        # opsætning for hver tjeneste
    ├── jellyfin/
    ├── sonarr/
    └── ...
```
```bash
mkdir -p /mnt/media/{film,serier,downloads,config}
```

**Hvorfor samme disk?** Så kan Sonarr og Radarr bruge **hardlinks**: filen kan ligge både i
`downloads/` og i `film/` med det samme – uden at blive kopieret og uden at fylde dobbelt.

## Rettigheder
Alle tjenester kører som bruger **1000** inde i Docker (`PUID=1000`, `PGID=1000`).
I en unprivileged container er bruger 1000 = **101000** på Proxmox-værten.

Kør på **værten**:
```bash
chown -R 101000:101000 /mnt/data/media
```

Eller inde i **containeren**:
```bash
chown -R 1000:1000 /mnt/media
```

## Samme sti i alle tjenester
Giv alle tjenester mappen under det **samme navn** (fx `/data`).
Så ser Sonarr, Radarr og download-klienten de samme stier, og hardlinks virker:
```yaml
volumes:
  - /mnt/media:/data
```
