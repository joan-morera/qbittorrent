# qBittorrent RPi4 (Static)

Static `qbittorrent-nox` build for Raspberry Pi 4 (ARM64/Cortex-A72) on Google Distroless.

## Build Specs

- **Base:** `gcr.io/distroless/cc-debian13`
- **Components:** qBittorrent-nox, Libtorrent v1.2, Qt 6 (Minimal/No-GUI), Boost, OpenSSL 3, Mimalloc.
- **Flags:** `-O3`, `LTO`, `-mcpu=cortex-a72`, `-fstack-protector-strong`.
- **Linking:** Fully static.

## Automation

Daily cron checks upstream git tags. Rebuilds only when a new version is detected.

## Configuration

- **User:** `qbt` (1000:1000)
- **Root:** `/home/qbt` (includes `XDG_CONFIG_HOME`, `XDG_DATA_HOME`)

## Usage

```yaml
services:
  qbittorrent:
    image: ghcr.io/joan-morera/qbittorrent-rpi4:latest
    container_name: qbittorrent
    restart: unless-stopped
    ports:
      - "8080:8080"
      - "6881:6881"
      - "6881:6881/udp"
    volumes:
      - ./qbt-data:/home/qbt
      - /mnt/storage:/downloads
    environment:
      - PUID=1000
      - PGID=1000
```
