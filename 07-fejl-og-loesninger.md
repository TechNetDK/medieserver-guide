# 07 – Fejl jeg har ramt, og løsningerne

| Problem | Årsag | Løsning |
|---|---|---|
| Docker vil ikke starte i containeren | Mangler `nesting`/`keyctl` | På værten: `pct set 101 -features nesting=1,keyctl=1` og genstart containeren |
| "Permission denied" på `/mnt/media` | Forkert ejer | `chown -R 101000:101000 /mnt/data/media` på værten |
| Filer kopieres i stedet for at flyttes | Downloads og medier har forskellige stier eller diske | Brug `/mnt/media:/data` i alle tjenester (kapitel 02) |
| Sonarr kan ikke finde qBittorrent | Bruger IP eller `localhost` | Brug navnet `qbittorrent` – de er på samme Docker-netværk |
| Seerr starter ikke | `config/seerr` ejes ikke af bruger 1000 | `chown -R 1000:1000 /mnt/media/config/seerr` i containeren |
| Jellyfin kan ikke starte efter opdatering | Ny version ændrede databasen | Gendan databasen fra backup (kapitel 06), og brug den gamle version. En ældre version kan ikke læse den nye database |
| To Jellyfin kører mod samme data | En test-container blev startet manuelt | Kør kun én: `docker rm -f <navn>` |
| Video hakker | Jellyfin omkoder på CPU | Afspil i originalkvalitet, eller giv containeren et grafikkort (kapitel 05 i Proxmox-guiden – brug ID 101 i stedet for 100) og installer **NVIDIA Container Toolkit**, så Jellyfin i Docker kan bruge det |
| Docker fejler med `ip_unprivileged_port_start ... permission denied` | Fejl i ældre Proxmox efter en opdatering af containerd | Opdater Proxmox til 9.1 eller nyere |

## Nyttige kommandoer
```bash
docker compose ps                 # status
docker compose logs -f <tjeneste> # log
docker compose restart <tjeneste> # genstart én tjeneste
docker compose down && docker compose up -d   # genstart det hele
```
