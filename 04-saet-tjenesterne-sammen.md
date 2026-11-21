# 04 – Sæt tjenesterne sammen

Gør det i denne rækkefølge. Inde i Docker-netværket bruger du **navnet** på tjenesten
i stedet for en IP, fx `http://sonarr:8989`.

## 1. qBittorrent – `http://<ip>:8080`
- Find det midlertidige password i loggen: `docker logs qbittorrent`
- Skift brugernavn og password med det samme (**Tools → Options → Web UI**).
- **Downloads:** Standardmappe `/data/downloads`.

## 2. Prowlarr – `http://<ip>:9696`
- Lav en login-bruger.
- Tilføj dine kilder under **Indexers**.
- **Settings → Apps:** Tilføj Sonarr (`http://sonarr:8989`) og Radarr (`http://radarr:7878`)
  med deres **API-nøgle** (findes i Sonarr/Radarr under **Settings → General**).

## 3. Sonarr og Radarr – `:8989` og `:7878`
- **Media Management → Root Folders:** `/data/serier` (Sonarr) og `/data/film` (Radarr).
- Slå **Use Hardlinks instead of Copy** til.
- **Download Clients:** Tilføj qBittorrent – host `qbittorrent`, port `8080`.

## 4. Bazarr – `:6767`
- **Settings → Sonarr/Radarr:** host `sonarr` / `radarr` + API-nøgle.
- **Languages:** Vælg fx dansk og engelsk.
- Tilføj udbydere af undertekster under **Providers**.

## 5. Jellyfin – `:8096`
- Lav en admin-bruger i opsætningsguiden.
- Tilføj biblioteker: **Film** → `/data/film` og **Serier** → `/data/serier`.
- Lav en bruger til hvert familiemedlem – ikke admin.

## 6. Seerr – `:5055`
- Log ind med din Jellyfin-konto. Jellyfin-adressen er `http://jellyfin:8096`.
- **Services:** Tilføj Sonarr og Radarr (host `sonarr` / `radarr` + API-nøgle).
- Nu kan familien ønske film og serier, og resten sker automatisk.
