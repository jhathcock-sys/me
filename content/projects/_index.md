---
title: "Projects"
date: 2026-01-08
draft: false
description: "Technical projects showcasing infrastructure, automation, and DevOps skills."
---

A collection of technical projects demonstrating hands-on experience with infrastructure, automation, and modern DevOps practices.

---

## Featured Projects

### [Home Lab](../home-lab/)
Self-hosted infrastructure running on Proxmox and Raspberry Pi. Includes monitoring, DNS, dashboards, and containerized services managed via Dockhand.

**Technologies:** Proxmox, Docker, Pi-hole, Grafana, Prometheus, Tailscale

---

### [GitOps Infrastructure](../gitops/)
Infrastructure as Code repository for managing all homelab Docker configurations. Version-controlled deployments with secrets management and remote stack control via Hawser.

**Technologies:** Git, Docker Compose, Dockhand, GitHub

---

### [Docker Security Review](../security-review/)
Comprehensive security audit of homelab Docker infrastructure. Identified critical vulnerabilities including privileged containers and unprotected Docker socket mounts, then applied hardening fixes following CIS Docker Benchmark guidelines.

**Technologies:** Docker, Security Auditing, Linux Capabilities, Container Hardening

---

### [Container Resource Management](../container-resource-management/)
Implemented comprehensive memory limits and fixed Prometheus alerting across 20 Docker containers. Fixed broken alerts showing +Inf%, applied resource limits based on usage analysis, and improved monitoring accuracy.

**Technologies:** Docker, Prometheus, Grafana, PromQL, GitOps

---

### [Claude Code Memory System](../claude-memory/)
Structured context system for AI-assisted infrastructure development. Converts markdown-based memory files to XML format for better hierarchy and queryability across development sessions.

**Technologies:** Claude Code, XML, Bash, Git

---
