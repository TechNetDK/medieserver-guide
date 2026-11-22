# 08 – VPN til download-klienten

Med en VPN går al trafik fra download-klienten gennem en VPN-udbyder.
Din internetudbyder kan ikke se, hvad der hentes, og din hjemme-IP bliver ikke vist.

Jeg bruger **Gluetun**: en lille container, der laver VPN-forbindelsen. qBittorrent bruger
Gluetuns netværk, så **al** dens trafik går gennem VPN'en. Falder VPN'en ud, blokerer Gluetun
trafikken (kill switch), så intet slipper ud uden om.

Du skal have et abonnement hos en VPN-udbyder, som Gluetun understøtter (fx Mullvad, ProtonVPN eller AirVPN).

## 1. Giv containeren adgang til `/dev/net/tun` (på Proxmox-værten)
```bash
pct set 101 --dev0 /dev/net/tun
pct reboot 101
```
Bruger du allerede `dev0` til noget andet, så tag det næste ledige nummer, fx `--dev1`.

## 2. Gem VPN-oplysningerne i en fil (inde i containeren)
Hent en **WireGuard**-konfiguration hos din udbyder, og skriv værdierne ind:
```bash
cd /opt/medieserver
nano gluetun.env
```
```ini
VPN_SERVICE_PROVIDER=mullvad
VPN_TYPE=wireguard
WIREGUARD_PRIVATE_KEY=<din-private-nøgle>
WIREGUARD_ADDRESSES=<adresse-fra-udbyderen>
SERVER_COUNTRIES=Sweden
```
```bash
chmod 600 gluetun.env
```
Navnene på felterne afhænger af udbyderen – se Gluetuns wiki for din udbyder.

## 3. Ret docker-compose.yml
Tilføj `gluetun`, og **erstat** hele `qbittorrent`-blokken med denne:
```yaml
  gluetun:
    container_name: gluetun
    restart: unless-stopped
    networks: [media]
    image: qmcgaw/gluetun:latest
    cap_add:
      - NET_ADMIN
    devices:
      - /dev/net/tun:/dev/net/tun
    env_file: ./gluetun.env
    environment:
      - TZ=Europe/Copenhagen
    ports: ["8080:8080"]

  qbittorrent:
    container_name: qbittorrent
    restart: unless-stopped
    image: lscr.io/linuxserver/qbittorrent:latest
    network_mode: "service:gluetun"
    depends_on:
      gluetun:
        condition: service_healthy
        restart: true
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Copenhagen
      - WEBUI_PORT=8080
    volumes:
      - /mnt/media/config/qbittorrent:/config
      - /mnt/media:/data
```
```bash
docker compose up -d
```

| Ændring | Hvorfor |
|---|---|
| `network_mode: "service:gluetun"` | qBittorrent bruger Gluetuns netværk – al trafik går gennem VPN'en. |
| Ingen `<<: *common`, `networks` eller `ports` på qBittorrent | De må ikke bruges sammen med `network_mode`. Porten åbnes på Gluetun i stedet. |
| `depends_on ... service_healthy` | qBittorrent starter først, når VPN'en er oppe. |
| `restart: true` | qBittorrent genstarter automatisk, når Gluetun genstarter. |
| `cap_add: NET_ADMIN` og `/dev/net/tun` | Gluetun skal bruge dem for at lave VPN-forbindelsen. |

Port 6881 er fjernet. Bag en VPN virker den kun, hvis udbyderen understøtter port forwarding.

## 4. Ret adressen i Sonarr og Radarr
qBittorrent har nu Gluetuns adresse. Under **Settings → Download Clients** skifter du host
fra `qbittorrent` til **`gluetun`** (port `8080`). Webgrænsefladen er stadig `http://<ip>:8080`.

## 5. Test at VPN'en virker
```bash
docker logs gluetun | tail -n 20          # skal vise, at VPN'en er forbundet
docker exec qbittorrent curl -s ifconfig.me
```
Den sidste kommando skal vise VPN-udbyderens IP – **ikke** din egen.
