# GitOps Configs for ProxMox Docker-SRV

**Author:** James Hathcock
**Date:** January 26th, 2026

## Project Description

I am moving all of my Yamls to GIT for management by Dockhand on my servers.  At some point I would like to move to **Kubernettes** but for now I am working my way to that.  Working with a learning curve but documentation to what I am striving for.

### Uptime Kuma

```YAML
services:
  uptime-kuma:
    image: louislam/uptime-kuma:1
    container_name: uptime-kuma
    volumes:
      - ./data:/app/data
      - /var/run/docker.sock:/var/run/docker.sock # Optional: Allows it to monitor Docker containers directly
    ports:
      - "3001:3001"
    restart: unless-stopped
```

### Home Lab Tools Stack

```YAML
services:
  # Homarr - Your Dashboard
  homarr:
    container_name: homarr
    image: ghcr.io/ajnart/homarr:latest
    restart: unless-stopped
    volumes:
      - ./homarr/configs:/app/data/configs
      - ./homarr/icons:/app/public/icons
      - /var/run/docker.sock:/var/run/docker.sock # Optional: Lets Homarr control local containers
    ports:
      - "7575:7575"

  # HomeBox - Asset Inventory
  homebox:
    image: ghcr.io/sysadminsmedia/homebox:latest
    container_name: homebox
    restart: unless-stopped
    environment:
      - HBOX_LOG_LEVEL=info
      - HBOX_LOG_FORMAT=text
      - HBOX_WEB_PORT=3100
      - HBOX_OPTIONS_ALLOW_REGISTRATION=true # Set to false after you create your account!
    volumes:
      - ./homebox-data:/data
    ports:
      - "3100:3100"
```

### Minecraft Server for Daughter

```YAML
services:
  mc:
    image: itzg/minecraft-server
    container_name: mc-server
    tty: true
    stdin_open: true
    ports:
      - "25565:25565"       # Java Port
      - "19132:19132/udp"   # Bedrock Port (Mapped directly to this container)
    environment:
      EULA: "TRUE"
      TYPE: "PAPER"
      VERSION: "LATEST"
      MEMORY: "6G"
      MOTD: "Hosted on Dell R430"
      MAX_PLAYERS: 10
      # This variable tells the container to auto-download these plugins on boot
      PLUGINS: |
        https://download.geysermc.org/v2/projects/geyser/versions/latest/builds/latest/downloads/spigot
        https://download.geysermc.org/v2/projects/floodgate/versions/latest/builds/latest/downloads/spi>
    volumes:
      - ./data:/data
    restart: unless-stopped
```

### Homepage (Dashboard)

```YAML
services:
  homepage:
    image: ghcr.io/gethomepage/homepage:latest
    container_name: homepage
    ports:
      - 4000:4000
    volumes:
      - /opt/homepage/config:/app/config
      - /var/run/docker.sock:/var/run/docker.sock:ro # Optional for Docker Integration
    environment:
      HOMEPAGE_ALLOWED_HOSTS: gethomepage.dev,192.168.1.4:4000,dashboard.home.lab #required adding port may be needed was for me
```


