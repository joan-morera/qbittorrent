# qBittorrent RPi4 (Static)

This is a personal Docker image for `qBittorrent-nox`, built statically.

I wanted a highly optimized, lightweight headless build for my Raspberry Pi 4, ergo this repo.

## Features & Improvements
- **Size**: Minimal Distroless base, stripped binaries.
- **Arch**: Multi-arch (AMD64, ARM64) plus a specifically optimized build for RPi4 (Cortex-A72).
- **Security**: Runs as a dedicated non-root user (`qbt`) inside the container.
- **Optimization**: Built with `LTO`, `-O3`, and `mimalloc` for performance. RPi4 variant tuned with `-mcpu=cortex-a72`.

## Build Policy & Versions
The GitHub Actions workflow runs daily to check for:
1. New **qBittorrent/Libtorrent/Qt** releases (tags) from upstream repositories.
2. Updates to the underlying **Distroless** base image.

If any change is detected, the image is automatically rebuilt and published.

## Usage
The container is configured to store all data and configuration in `/home/qbt` (which you should mount as a volume).

### Basic Usage
```bash
docker run -d \
  --name qbittorrent \
  -p 8080:8080 \
  -p 6881:6881 \
  -p 6881:6881/udp \
  -v qbt_data:/home/qbt \
  ghcr.io/joan-morera/qbittorrent-rpi4:rpi4
```

### Docker Compose
```yaml
services:
  qbittorrent:
#   image: ghcr.io/joan-morera/qbittorrent-rpi4:latest # Generic optimized (AMD64/ARM64)
    image: ghcr.io/joan-morera/qbittorrent-rpi4:rpi4   # RPi4 optimized (ARM64)
    container_name: qbittorrent
    restart: unless-stopped
    ports:
      - "8080:8080"
      - "6881:6881"
      - "6881:6881/udp"
    volumes:
      - qbt_data:/home/qbt
      - /path/to/downloads:/downloads
    environment:
      - PUID=1000
      - PGID=1000

volumes:
  qbt_data:
```

> Note: Adjust volume paths and PUID/PGID to match your host permissions.
