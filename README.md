# qBittorrent RPi4 (Static & Optimized)

A highly optimized, fully static build of `qBittorrent-nox` tailored for Raspberry Pi 4 (ARM64/Cortex-A72), packaged in a minimal Google Distroless container.

## Build Features

This image is built from source with a focus on performance, minimal footprint, and security.

### 🏗 Components
- **Base Image:** `gcr.io/distroless/cc-debian13` (Debian Trixie based, no shell, no package manager).
- **Core:** qBittorrent-nox (No GUI).
- **Network:** Libtorrent v1.2 (Selected for stability and broad compatibility).
- **Framework:** Qt 6 (Minimal static build, stripped of GUI/Widgets/DBus/OpenGL).
- **Memory Manager:** Microsoft `mimalloc` (Statically linked and overriding glibc malloc).
- **Libraries:** Boost (Minimal), OpenSSL 3.

### ⚡ Optimization & Hardening
- **CPU Tuning:** `-mcpu=cortex-a72 -mtune=cortex-a72` (Native RPi4 optimization).
- **Compiler Level:** `-O3` (Aggressive optimization) + `LTO` (Link Time Optimization).
- **Security:**
  - `-fstack-protector-strong`
  - `_FORTIFY_SOURCE=2`
  - `-Werror=format-security`
- **ARM64 Fixes:** `-mno-outline-atomics` (Prevents potential atomic locking issues).
- **Static Linking:** All dependencies are statically linked into a single binary, preventing "dependency hell" and reducing attack surface.

## 🔄 Rebuild Policy

The repository runs a **Daily Check** (cron) that queries upstream git tags for all components (qBittorrent, Libtorrent, Qt, Boost, OpenSSL, Mimalloc) and the Base Image digest.
- **Trigger:** A rebuild is triggered **only** if a new version is detected for any component.
- **Result:** Always provides the absolute latest stable versions of all pieces.

## ⚙️ Internal Config

- **User:** `qbt` (UID: `1000`, GID: `1000`)
- **Home/Config:** `/home/qbt`
- **Environment:**
  - `XDG_CONFIG_HOME=/home/qbt`
  - `XDG_DATA_HOME=/home/qbt`

All configuration and data reside in a single directory structure under `/home/qbt`.

## 🐳 Usage

### Docker Compose

```yaml
services:
  qbittorrent:
    image: ghcr.io/joan-morera/qbittorrent-rpi4:latest
    container_name: qbittorrent
    restart: unless-stopped
    ports:
      - "8080:8080"      # WebUI
      - "6881:6881"      # TCP Peer Port
      - "6881:6881/udp"  # UDP Peer Port
    volumes:
      # Persist config and data (torrents, .config, .local)
      - ./qbt-data:/home/qbt
      # Optional: Mount external storage for downloads
      - /mnt/storage:/downloads
    environment:
      - PUID=1000
      - PGID=1000
    # Recommended for high-performance networking
    sysctls:
      - net.core.rmem_max=4194304
      - net.core.wmem_max=1048576
```

### Initial Login
- **WebUI:** `http://localhost:8080`
- **Default Credentials:** Printed in the logs on first start (qBittorrent v5+ generates a random password). Checks logs with `docker logs qbittorrent`.
