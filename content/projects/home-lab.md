---
title: "Home Lab"
date: 2026-01-08
draft: false
---

# 🏠 My Homelab Setup

Welcome to my homelab documentation. This project tracks the infrastructure, services, and networking configuration of my self-hosted environment. The goal is to create a resilient, automated, and organized system for home services, monitoring, and development.

---

## 📚 Documentation Wiki

**[View Full Homelab Documentation →](https://jhathcock-sys.github.io/homelab-wiki/)**

Complete technical documentation is available on my public wiki, including:
- Detailed service configurations and architecture diagrams
- Step-by-step setup guides and troubleshooting
- GitOps workflow and deployment procedures
- Security best practices and monitoring setup
- Project changelogs and infrastructure evolution

The wiki is built with Quartz v4 and features full-text search, graph view, and WikiLinks for easy navigation.

---

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
   * **Prometheus:** Metrics collection from 7 targets (ProxMoxBox, Pi5, Pi-hole LXC, NPM LXC, Wazuh VM, cAdvisor)
   * **Alertmanager:** Alert routing with Discord notifications
   * **Loki:** Centralized log aggregation
   * **Promtail:** Log collector agent
   * **Node Exporter:** Installed on all hosts for system metrics
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
| 192.168.1.5 | Synology NAS (DS220j) | Network storage (7.2TB, SNMP monitored) |
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

Dedicated VM (Debian 12) running Wazuh v4.14.2 for centralized security monitoring across all homelab hosts.

**Components:**
* **Wazuh Manager:** Core SIEM engine - processes security events, file integrity monitoring, vulnerability detection
* **Wazuh Indexer:** OpenSearch-based backend for storing and searching security data
* **Wazuh Dashboard:** Web UI for security analysis, threat hunting, and compliance reporting (https://192.168.1.7)
* **Filebeat:** Ships alerts from manager to indexer

**Agents Deployed:**
| Host | Agent Name | Monitors |
|------|------------|----------|
| ProxMoxBox (192.168.1.4) | SRV-DOCKER01 | Docker host, containers, system logs |
| Pi5 (192.168.1.234) | pi-infra | Secondary DNS, Tailscale, system logs |
| Pi-hole LXC (192.168.1.3) | SRV-DNS01 | Primary DNS server, Pi-hole logs |
| NPM LXC (192.168.1.6) | SRV-NPM01 | Reverse proxy, SSL certificates, access logs |

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
* **SIEM:** [Wazuh Dashboard](https://192.168.1.7) - Security event analysis and threat detection

---

## 🚀 Infrastructure as Code

All Docker compose files are managed via GitOps. See my [GitOps Project](../gitops/) for details on the repository structure and deployment workflow.

**Repository:** [github.com/jhathcock-sys/Dockers](https://github.com/jhathcock-sys/Dockers)

---

## 🛡️ Security Hardening

The Docker infrastructure has undergone a comprehensive security audit. Critical vulnerabilities were identified and fixed, including:

- Removed privileged mode from cAdvisor (replaced with specific capabilities)
- Added read-only flags to Docker socket mounts
- Eliminated default password fallbacks

See my [Docker Security Review](../security-review/) for the full audit report, methodology, and lessons learned.

---

## 🚀 Recent Projects

### Podcast Studio (2026-02-03)
Self-hosted video podcast recording platform with 4K multi-track support. Built for D&D sessions with up to 6 participants.

**Stack:** LiveKit (WebRTC), React + TypeScript, Node.js/Express, MinIO (S3 storage), FFmpeg (post-processing), Coturn (TURN server)

**Features:**
- Hybrid LiveKit + double-ended recording for true 4K quality
- Client-side MediaRecorder with resumable uploads (Uppy.js)
- Live streaming to YouTube/Twitch via RTMP
- Automated audio normalization and multi-track sync with FFmpeg
- Scene switching with custom layouts

**Storage:** ~70GB per 1-hour 6-person 4K session

**Repository:** [podcast-studio](https://github.com/jhathcock-sys/podcast-studio)

---

## 🗄️ NAS Integration (2026-02-04)

Integrated Synology DS220j (7.2TB) with homelab infrastructure for centralized storage and monitoring.

**SNMP Monitoring:**
- Prometheus scrapes NAS metrics via snmp-exporter
- Monitors: network interfaces, traffic counters, interface status
- Grafana dashboards available for visualization

**SMB Storage Shares:**
- Mounted on ProxMoxBox (via Proxmox host bind mount)
- Mounted on Pi5 with full read/write access
- Directories: `/mnt/nas/homelab/docker-backups/` and `/mnt/nas/homelab/media/`
- Docker containers have full read/write access for backups and media
- Persistent mounts configured in /etc/fstab on both systems

**Syslog Forwarding (Partial):**
- Synology forwards logs to Promtail on ProxMoxBox:1514
- Messages arriving but needs RFC 3164 → RFC 5424 format conversion
- Future enhancement: add syslog relay for proper parsing

**Storage Capacity:**
- Total: 7.2TB
- Used: 4.9TB
- Available: 2.4TB (ready for media library and backups)

---

## 📋 Future Plans

- [ ] **Podcast Studio Deployment** - Deploy and test on 192.168.1.8
- [ ] **OPNsense + Managed Switch** - Enterprise networking with VLANs and IDS/IPS
- [ ] Add media stack (Jellyfin/Plex, Sonarr, Radarr) - **NAS storage ready**
- [ ] Implement Home Assistant for home automation
- [ ] Migrate to Kubernetes for orchestration
- [ ] Set up offsite backups to cloud storage
- [ ] Fix syslog format conversion for NAS logs
- [x] ~~NAS integration~~ (Completed - SNMP monitoring + SMB shares)
- [x] ~~Container resource management~~ (Completed - memory limits on all 20 containers)
- [x] ~~Add alerting via Grafana/Prometheus~~ (Completed - Discord notifications active)
- [x] ~~Wazuh SIEM~~ (Completed - agents on ProxMoxBox and Pi5, ready for Suricata integration)
