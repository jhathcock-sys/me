---
title: "GitOps Infrastructure"
date: 2026-01-26
draft: false
---

# 🔄 GitOps Infrastructure

**Repository:** [github.com/jhathcock-sys/Dockers](https://github.com/jhathcock-sys/Dockers)

## Project Overview

Infrastructure as Code for my homelab environment. All Docker Compose configurations are version-controlled in Git, enabling reproducible deployments, change tracking, and easy rollbacks.

This is a stepping stone toward full Kubernetes orchestration, building GitOps practices while working with Docker Compose.

---

## 📁 Repository Structure

```
homelab-ops/
├── proxmox/                    # ProxMoxBox (192.168.1.4) stacks
│   ├── dockhand/               # Docker management UI
│   ├── homepage/               # Dashboard + config files
│   │   └── config/             # services.yaml, widgets.yaml, etc.
│   ├── homelab-tools/          # Homebox asset inventory
│   ├── minecraft/              # PaperMC + Geyser/Floodgate
│   ├── monitoring/             # Prometheus, Grafana, Node Exporter, cAdvisor
│   │   └── prometheus/         # prometheus.yml scrape configs
│   ├── nginx-proxy-manager/    # Reverse proxy
│   └── uptime-kuma/            # Service health monitoring
│
└── pi5/                        # Raspberry Pi 5 stacks (via Hawser)
    ├── infra/                  # Pi-hole + Tailscale
    ├── mealie/                 # Recipe management
    └── nebula-sync/            # Pi-hole sync
```

---

## 🚀 Deployment Workflow

### Local Stacks (ProxMoxBox)

```bash
# 1. Edit compose files locally
vim proxmox/homepage/docker-compose.yaml

# 2. Commit and push
git add . && git commit -m "Update homepage config"
git push

# 3. On server: pull and deploy
cd /opt/homepage
git pull
docker compose up -d
```

### Remote Stacks (Pi5 via Hawser)

Pi5 stacks are managed remotely through Dockhand's **Hawser** agent:
- Compose files live on ProxMoxBox at `/opt/pi5-stacks/`
- Hawser executes commands on the Pi5 Docker daemon
- No SSH required for routine deployments

---

## 🗺️ Path Mapping

| Git Path | Deploy Path | Server |
|----------|-------------|--------|
| `proxmox/<stack>/` | `/opt/<stack>/` | ProxMoxBox |
| `pi5/<stack>/` | `/opt/pi5-stacks/<stack>/` | ProxMoxBox (Hawser → Pi5) |

---

## 🔧 Stack Management with Dockhand

[Dockhand](https://github.com/fnsys/dockhand) provides a web UI for Docker management:

- **Stack Import:** Adopt existing containers by pointing to compose files
- **Remote Management:** Hawser agent enables control of remote Docker hosts
- **Compose Editing:** Edit and redeploy stacks from the UI

### Key Configuration

```yaml
services:
  dockhand:
    image: fnsys/dockhand:latest
    environment:
      - HOST_DATA_DIR=/opt          # Path resolution for stacks
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /opt:/opt                   # Access to all compose files
```

---

## 🔐 Secrets Management

Sensitive values are kept out of Git using `.env` files:

```bash
# Template provided in repo
cp .env.example .env

# Generate encryption keys
openssl rand -hex 32
```

**Pattern:**
- `.env.example` → Committed (templates with placeholders)
- `.env` → Gitignored (actual secrets)

---

## 📊 Current Stacks

### ProxMoxBox Services

| Stack | Ports | Purpose |
|-------|-------|---------|
| dockhand | 3000 | Docker management |
| homepage | 4000 | Dashboard |
| homelab-tools | 3100 | Homebox inventory |
| minecraft | 25565, 19132/udp | Game server |
| monitoring | 3030, 9090, 9100, 8081, 3101 | Grafana, Prometheus, Loki, exporters |
| nginx-proxy-manager | 80, 443, 81 | Reverse proxy |
| uptime-kuma | 3001 | Health monitoring |

### Pi5 Services (via Hawser)

| Stack | Services | Purpose |
|-------|----------|---------|
| infra | Pi-hole, Tailscale | DNS + VPN |
| mealie | Mealie | Recipe management |
| nebula-sync | Nebula-sync | Pi-hole replication |
| promtail | Promtail | Log collection to Loki |

---

## 📋 Future Plans

- [ ] Implement CI/CD pipeline for automated deployments
- [ ] Add pre-commit hooks for YAML linting
- [ ] Migrate to Kubernetes with ArgoCD
- [ ] Set up Renovate for automated image updates
