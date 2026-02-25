---
title: "Projects"
date: 2026-01-08
draft: false
description: "Technical projects showcasing infrastructure, automation, and DevOps skills."
---

A collection of technical projects demonstrating hands-on experience with infrastructure, automation, and modern DevOps practices.

---

## Featured Projects

### [Self-Hosted AI Inference Stack & Agent Team](../ai-inference-stack/)
Production-grade, local-first AI platform combining MLX on Apple Silicon, Ollama on Linux, and Claude as cloud fallback — all routed through LiteLLM with Redis caching. Powers a team of six specialized AI agents handling research, monitoring, backend, frontend, and creative work.

**Technologies:** MLX, Ollama, LiteLLM, Redis, Anthropic Claude, OpenClaw, Apple Silicon

---

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

### [Semantic Memory: Claude Memory System 3.0](../semantic-memory-or-claude-memory-3/)
Evolution from file-based context to semantic vector search using ChromaDB. On-demand retrieval via MCP integration replaces pre-loaded context — AI memory that scales with your infrastructure.

**Technologies:** ChromaDB, MCP, Python, Vector Embeddings, Claude Code

---
