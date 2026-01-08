---
title: "Infrastructure & Security Home Lab"
date: 2026-01-08
tags: ["Homelab", "Proxmox", "High Availability", "Network Security", "Linux"]
summary: "A continuous learning sandbox utilizing Proxmox VE, Redundant DNS (Pi-hole), Reverse Proxying, and Zero Trust networking."
weight: 1
---

## Project Overview
As a Systems Engineer, I believe in "learning by doing." This lab serves as a testbed for simulating enterprise architectures. The environment is currently focused on **High Availability (HA)**, **Zero Trust Networking**, and **Secure Service Delivery**.

## 🖥️ Current Stack

| Component | Specification | Purpose |
| :--- | :--- | :--- |
| **Hypervisor** | Proxmox VE | Type-1 Hypervisor managing Windows Server & Linux LXC containers. |
| **DNS / Security** | Pi-hole (Cluster) | **Redundant** DNS sinkholes for network-wide telemetry blocking. |
| **Reverse Proxy** | NGINX Proxy Manager | SSL termination and sub-domain routing for internal services. |
| **Remote Access** | Tailscale | Mesh VPN implementation for secure, zero-trust remote management. |
| **Data Sync** | Syncthing | Decentralized, real-time documentation and config synchronization. |

## 🛡️ Key Configurations

### 1. High Availability DNS (Redundant Pi-hole)
Deployed dual Pi-hole instances (Primary & Secondary) to eliminate DNS as a Single Point of Failure (SPOF).
* **Resilience:** Configured network DHCP to distribute both IP addresses; if one node goes down for maintenance, network resolution remains 100% active.
* **Security:** Blocks ads and tracking telemetry at the network level, reducing bandwidth usage and improving privacy across all VLANs.

### 2. Secure Service Delivery (NGINX Proxy Manager)
Implemented NGINX as a Reverse Proxy to manage internal traffic flow.
* **SSL/TLS Termination:** Centralized certificate management (Let's Encrypt) to ensure all internal web services are accessed via HTTPS, even if the native application doesn't support it.
* **Routing:** Maps internal IP:Port combinations to friendly DNS names (e.g., `proxmox.lab.local`), simplifying access and management.

### 3. Virtualization Core (Proxmox VE)
Migrated infrastructure to **Proxmox Virtual Environment** to decouple services from hardware.
* **Workloads:** Hosting virtualized Windows Server 2022 (Active Directory labs) and lightweight Linux LXC containers.
* **Operations:** Utilizing Proxmox's native snapshotting to test patch management strategies and rollback procedures safely.

### 4. Zero Trust & Data Continuity
* **Tailscale:** Establishes a secure overlay network, allowing remote management without opening firewall ports to the WAN.
* **Syncthing:** Creates a private, peer-to-peer synchronization mesh between the workstation (Pop!_OS) and servers, ensuring documentation is resilient and cloud-independent.

---
> *This project is active and updates as I study for my Security+ certification.*
