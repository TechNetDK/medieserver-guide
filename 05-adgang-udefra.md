# 05 – Adgang udefra

## Åbn ikke porte i routeren
Med en **Cloudflare Tunnel** går forbindelsen **ud** fra serveren til Cloudflare.
Du skal ikke åbne nogen porte, og din hjemme-IP bliver ikke vist.

## 1. Lav tunnelen
1. Log ind på Cloudflare-dashboardet og opret en tunnel (under **Networks → Tunnels** i skrivende stund).
2. Vælg **Cloudflared** og kopier **tokenet**.
3. Gem det i en fil ved siden af compose-filen:
```bash
cd /opt/medieserver
echo "TUNNEL_TOKEN=<dit-token>" > cloudflared.env
chmod 600 cloudflared.env
docker compose up -d cloudflared
```

## 2. Peg dine adresser på tjenesterne
Under tunnelens **Public Hostnames / Published application routes**:

| Adresse | Service |
|---|---|
| `film.ditdomæne.dk` | `http://jellyfin:8096` |
| `oensker.ditdomæne.dk` | `http://seerr:5055` |

> Cloudflares vilkår kan begrænse streaming af meget video gennem deres netværk på gratis-planen.
> Læs vilkårene, før du bruger tunnelen til Jellyfin.

Gør **kun** Jellyfin og Seerr tilgængelige udefra. Sonarr, Radarr, Prowlarr,
Bazarr og qBittorrent skal kun kunne nås hjemmefra.

## 3. Sikkerhed
- Stærke passwords på **alle** tjenester – også dem, der kun er på hjemmenettet.
- Slå login til i Sonarr, Radarr, Prowlarr og Bazarr (**Settings → General → Authentication**).
- Hemmeligheder (`cloudflared.env`) skal have `chmod 600` og må aldrig i Git.
