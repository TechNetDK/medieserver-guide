# 01 – Container og Docker

**Regel:** Kør aldrig Docker direkte på Proxmox-værten. Kør det i en LXC-container.
Så holdes værten ren, og du kan tage snapshot og backup af hele medieserveren.

## 1. Opret containeren (på Proxmox-værten)
Jeg bruger ID **101**. Brug et ledigt ID, og hold Proxmox opdateret (9.1 eller nyere).
```bash
pct create 101 local:vztmpl/ubuntu-<version>-standard_<version>_amd64.tar.zst \
  --hostname medieserver --ostype ubuntu --unprivileged 1 \
  --cores 2 --memory 4096 --swap 2048 --rootfs local-lvm:32 \
  --net0 name=eth0,bridge=vmbr0,ip=10.0.0.13/24,gw=10.0.0.1 \
  --nameserver 1.1.1.1 --features nesting=1,keyctl=1 \
  --onboot 1
```

| Valg | Hvorfor |
|---|---|
| `--unprivileged 1` | Root i containeren er ikke root på værten. |
| `--features nesting=1,keyctl=1` | Kræves, for at Docker kan køre i containeren. |
| `--memory 4096` | Jellyfin bruger meget RAM og CPU, når den omkoder video. |

## 2. Giv containeren mediemappen
Medierne ligger på datadisken på værten (se kapitel 04 i Proxmox-guiden):
```bash
mkdir -p /mnt/data/media
pct set 101 -mp0 /mnt/data/media,mp=/mnt/media
pct start 101
```
På værten hedder mappen `/mnt/data/media`. Inde i containeren hedder den `/mnt/media`.

## 3. Installer Docker (inde i containeren)
```bash
pct enter 101
apt update && apt upgrade -y
apt install -y curl
curl -fsSL https://get.docker.com | sh
docker --version && docker compose version
```

## 4. Test
```bash
docker run --rm hello-world
```
Ser du "Hello from Docker!", er alt klar.
