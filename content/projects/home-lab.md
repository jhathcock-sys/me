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

4. **Monitoring & Alerting Stack** 📊
   * **Grafana:** Dashboards and visualization (custom Homelab Overview + Docker Containers dashboards)
   * **Prometheus:** Metrics collection and storage
   * **Alertmanager:** Alert routing with Discord notifications
   * **Loki:** Centralized log aggregation
   * **Promtail:** Log collector agent
   * **Node Exporter:** System metrics
   * **cAdvisor:** Container metrics

   **Alerts Configured:** Disk space >80%/90%, High CPU/Memory >90%, Host down, Container down, Target scrape failures

### 🍓 Node 2: Raspberry Pi 5 (192.168.1.234)

High-availability node for DNS redundancy and isolated services, managed remotely via **Hawser** agent.

* **Pi-hole (Secondary):** Redundant DNS (ns2) for uptime during R430 reboots
* **Tailscale:** Secure zero-config VPN access
* **Mealie:** Recipe and meal planning manager
* **Nebula-Sync:** Syncs DNS records and blocklists between Pi-hole instances
* **Node Exporter:** System metrics fed to Prometheus
* **Promtail:** Log collector sending to Loki

---

## 🌐 Networking & DNS

| IP | Device | Role |
|----|--------|------|
| 192.168.1.3 | Primary Pi-hole | Main DNS (ns1.home.lab) |
| 192.168.1.4 | ProxMoxBox | Main Docker host |
| 192.168.1.5 | Synology NAS | Network storage |
| 192.168.1.6 | Nginx Proxy Manager | Reverse proxy |
| 192.168.1.7 | Wazuh VM | SIEM (security monitoring) |
| 192.168.1.234 | Pi5 | Secondary DNS, Tailscale |
| 192.168.1.253 | Proxmox | Hypervisor management |

### DNS Flow
1. Client requests `dashboard.home.lab`
2. **Pi-hole** resolves domain to the **NPM** IP address
3. **NPM** routes the request to the correct **Docker container** port

---

## 🔐 Security & SIEM

### Wazuh SIEM (192.168.1.7)

Dedicated VM running the full Wazuh security stack for centralized security monitoring across all homelab hosts.

**Components:**
* **Wazuh Manager:** Core SIEM engine - processes security events, file integrity monitoring, vulnerability detection
* **Wazuh Indexer:** OpenSearch-based backend for storing and searching security data
* **Wazuh Dashboard:** Web UI for security analysis, threat hunting, and compliance reporting
* **Filebeat:** Ships alerts from manager to indexer

**Agents Deployed:**
| Host | Agent Name | Monitors |
|------|------------|----------|
| ProxMoxBox (192.168.1.4) | SRV-DOCKER01 | Docker host, containers, system logs |
| Pi5 (192.168.1.234) | pi-infra | DNS server, Tailscale, system logs |

**Capabilities:**
* Real-time log analysis and correlation
* File integrity monitoring (FIM)
* Vulnerability detection
* Security Configuration Assessment (SCA)
* Rootkit detection
* Ready for Suricata IDS integration (planned with OPNsense)

---

## 📊 Dashboard & Management

* **Docker Management:** [Dockhand](http://192.168.1.4:3000) - UI for managing all stacks locally and remotely via Hawser
* **Application Dashboard:** [Homepage](http://192.168.1.4:4000) - Central hub for all services
* **Monitoring:** [Grafana](http://192.168.1.4:3030) - Custom dashboards for homelab overview and container metrics
* **Alerting:** [Alertmanager](http://192.168.1.4:9093) - Alert routing with Discord notifications
* **Log Aggregation:** [Loki](http://192.168.1.4:3101) - Centralized logs from all hosts
* **Health Checks:** [Uptime Kuma](http://192.168.1.4:3001) - Service availability monitoring
* **Inventory:** [Homebox](http://192.168.1.4:3100) - Physical IT gear tracking
* **SIEM:** [Wazuh Dashboard](http://192.168.1.7:443) - Security event analysis and threat detection

---

## 🚀 Infrastructure as Code

All Docker compose files are managed via GitOps. See my [GitOps Project](../gitops/) for details on the repository structure and deployment workflow.

**Repository:** [github.com/jhathcock-sys/Dockers](https://github.com/jhathcock-sys/Dockers)

---

## 📋 Future Plans

- [ ] **OPNsense + Managed Switch** - Enterprise networking with VLANs and IDS/IPS
- [ ] Add media stack (Jellyfin/Plex, Sonarr, Radarr)
- [ ] Implement Home Assistant for home automation
- [ ] Migrate to Kubernetes for orchestration
- [ ] Set up offsite backups to cloud storage
- [x] ~~Add alerting via Grafana/Prometheus~~ (Completed - Discord notifications active)
- [x] ~~Wazuh SIEM~~ (Completed - agents on ProxMoxBox and Pi5, ready for Suricata integration)
