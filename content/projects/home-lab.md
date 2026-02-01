---
title: "Home Lab"
date: 2026-01-08
draft: false
---

# 🏠 My Homelab Setup

Welcome to my homelab documentation. This project tracks the infrastructure, services, and networking configuration of my self-hosted environment. The goal is to create a resilient, automated, and organized system for home services, monitoring, and development.

## 🛠 Hardware Infrastructure

| Device | Role | OS / Hypervisor | Specs/Notes |
| :--- | :--- | :--- | :--- |
| **Dell R430** | Primary Server | **Proxmox VE** | The heavy lifter. Runs LXC containers and manages core network services. |
| **Raspberry Pi 5** | Secondary Node | **Debian (Docker)** | Low-power node for DNS redundancy and lightweight services. |
| **Synology NAS** | Network Storage | **DSM** | Centralized file storage and backups. |

---

## ☁️ Virtualization & Software Stack

### 🖥️ Node 1: Dell R430 (ProxMoxBox - 192.168.1.4)

The Proxmox host runs **LXC Containers** to separate concerns:

1. **Nginx Proxy Manager** 🛡️
   * Reverse proxy with SSL termination
   * Routes all `*.home.lab` traffic

2. **Pi-hole (Primary)** 🛑
   * Network-wide ad blocking and local DNS (ns1.home.lab)

3. **Docker Host** 🐳
   * Core application stack managed by **Dockhand**
   * **Services Running:**
     * **Homepage:** Primary dashboard for the lab
     * **Homebox:** Asset inventory and tracking
     * **Uptime Kuma:** Service health monitoring
     * **Minecraft Server:** PaperMC with Geyser/Floodgate (Java + Bedrock)
     * **Syncthing:** File synchronization across devices

4. **Monitoring Stack** 📊
   * **Grafana:** Dashboards and visualization
   * **Prometheus:** Metrics collection and storage
   * **Node Exporter:** System metrics
   * **cAdvisor:** Container metrics

### 🍓 Node 2: Raspberry Pi 5 (192.168.1.234)

High-availability node for DNS redundancy and isolated services, managed remotely via **Hawser** agent.

* **Pi-hole (Secondary):** Redundant DNS (ns2) for uptime during R430 reboots
* **Tailscale:** Secure zero-config VPN access
* **Mealie:** Recipe and meal planning manager
* **Nebula-Sync:** Syncs DNS records and blocklists between Pi-hole instances
* **Node Exporter:** System metrics fed to Prometheus

---

## 🌐 Networking & DNS

| IP | Device | Role |
|----|--------|------|
| 192.168.1.3 | Primary Pi-hole | Main DNS (ns1.home.lab) |
| 192.168.1.4 | ProxMoxBox | Main Docker host |
| 192.168.1.5 | Synology NAS | Network storage |
| 192.168.1.6 | Nginx Proxy Manager | Reverse proxy |
| 192.168.1.234 | Pi5 | Secondary DNS, Tailscale |
| 192.168.1.253 | Proxmox | Hypervisor management |

### DNS Flow
1. Client requests `dashboard.home.lab`
2. **Pi-hole** resolves domain to the **NPM** IP address
3. **NPM** routes the request to the correct **Docker container** port

---

## 📊 Dashboard & Management

* **Docker Management:** [Dockhand](http://192.168.1.4:3000) - UI for managing all stacks locally and remotely via Hawser
* **Application Dashboard:** [Homepage](http://192.168.1.4:4000) - Central hub for all services
* **Monitoring:** [Grafana](http://192.168.1.4:3030) - System and container metrics
* **Health Checks:** [Uptime Kuma](http://192.168.1.4:3001) - Service availability monitoring
* **Inventory:** [Homebox](http://192.168.1.4:3100) - Physical IT gear tracking

---

## 🚀 Infrastructure as Code

All Docker compose files are managed via GitOps. See my [GitOps Project](/projects/gitops/) for details on the repository structure and deployment workflow.

**Repository:** [github.com/jhathcock-sys/Dockers](https://github.com/jhathcock-sys/Dockers)

---

## 📋 Future Plans

- [ ] Add media stack (Jellyfin/Plex, Sonarr, Radarr)
- [ ] Implement Home Assistant for home automation
- [ ] Migrate to Kubernetes for orchestration
- [ ] Set up offsite backups to cloud storage
- [ ] Add alerting via Grafana/Prometheus
