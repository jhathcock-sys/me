---
title: "Self-Hosted AI Inference Stack & Agent Team"
date: 2026-02-23
draft: false
tags: ["ai", "infrastructure", "automation", "homelab"]
summary: "A production-grade, self-hosted AI inference platform with local-first LLM routing, Redis caching, cloud fallback, and a team of specialized AI agents — all running on homelab hardware."
---

# 🤖 Self-Hosted AI Inference Stack & Agent Team

A production-grade, local-first AI platform built on homelab hardware. Local models handle the majority of requests for free, with cloud fallback only when needed. A team of specialized agents handles research, monitoring, backend, frontend, and creative work — all orchestrated through a central LiteLLM proxy.

---

## 🎯 The Goal

Most AI setups are cloud-only: every request hits an API, every token costs money, and nothing runs when the internet is down. This project builds the opposite — a **local-first inference stack** where:

- **~95% of requests** are served by local hardware at zero cost
- **Cloud (Anthropic Claude)** is available as a reliable fallback, not the primary
- **Redis caching** eliminates redundant API calls across sessions
- A **team of specialized agents** handles different domains autonomously

---

## 🏗️ Architecture

### Hardware

| Machine | Role | Key Specs |
| :--- | :--- | :--- |
| **Mac Studio** | Primary inference node | Apple Silicon, runs MLX models |
| **Dell Linux Server** | Proxy + secondary inference | x86, runs LiteLLM + Ollama |

### Routing Diagram

```
User / Agent Request
        ↓
LiteLLM Proxy (192.168.1.60:4000)
        ↓
[1] MLX API (192.168.1.226:8000)  ← Primary — Apple Silicon, fastest quality
        ↓ timeout / fail
[2] Ollama (localhost:11434)       ← Backup — x86 Linux, always-on
        ↓ timeout / fail
[3] Anthropic Claude (API)         ← Fallback — 99.9% uptime guarantee
```

Traffic flows down the chain only on timeout or failure. In practice, MLX handles the vast majority of requests.

---

## ⚡ Inference Nodes

### Node 1: MLX on Mac Studio (Primary)

[MLX](https://github.com/ml-explore/mlx) is Apple's machine learning framework optimized for Apple Silicon's unified memory architecture. Running models locally on the Mac Studio delivers significantly better throughput than equivalent x86 hardware.

**Models Running:**
| Model | Size | Use Case |
| :--- | :--- | :--- |
| Qwen2.5-14B | 16GB | General reasoning, complex tasks |
| Phi-4-14B | 15GB | Fast responses, coding assistance |

**Service:** Managed by `launchd` — auto-starts on login, auto-restarts on crash.
**Logs:** `~/.openclaw/workspace/logs/mlx-api.log`
**Response time:** ~4.7s average

---

### Node 2: Ollama on Linux Server (Backup)

[Ollama](https://ollama.com) runs on the always-on Dell server, providing a reliable backup when the Mac is unavailable or overloaded.

**Models Running:**
| Model | Size | Latency | Use Case |
| :--- | :--- | :--- | :--- |
| Qwen2.5-14B | 9GB | 2.7s | General reasoning |
| Llama3.2 | 2GB | 0.8s | Fast/lightweight tasks |
| Qwen2.5-Coder | 9GB | 2.7s | Code generation |

**Storage:** 19GB total (optimized from 51.7GB after removing unused models).

---

### Node 3: Anthropic Claude (Cloud Fallback)

When both local nodes are unavailable or a request exceeds local model capabilities, traffic falls through to Claude. The 30-second timeout on local models ensures fast fallback without long waits.

**Models available via fallback:**
- Claude Haiku — Fast, low cost (~$0.08/1M tokens), 657ms response
- Claude Sonnet — Balanced quality/speed
- Claude Opus — Maximum capability for complex tasks

**Cost impact:** With local models handling ~95% of traffic, Claude costs run at roughly **$13/month** compared to hundreds for cloud-only setups.

---

## 🔀 LiteLLM Proxy

[LiteLLM](https://github.com/BerriAI/litellm) acts as the unified gateway — a single API endpoint that abstracts all three inference backends behind one interface. Every agent and every tool talks to LiteLLM; the routing, fallback logic, and caching are all handled transparently.

**Key features in use:**
- **Unified API:** OpenAI-compatible endpoint for all models
- **Automatic fallback:** Configurable order with timeout-based failover
- **Redis caching:** 10-minute TTL on repeated queries — eliminates redundant calls
- **Cost tracking:** Per-model spend logging to PostgreSQL
- **Model aliases:** Named routes per agent role

### Agent-Specific Routes

Each agent has a dedicated LiteLLM route tuned for its workload:

| Agent | Route | Primary Model | Backup |
| :--- | :--- | :--- | :--- |
| Skip (main) | `smart-agent` | MLX Qwen2.5 | Ollama → Claude |
| Nellie (research) | `nellie-research` | Ollama Qwen | Kimi K2.5 |
| Chief (monitoring) | `chief-monitor` | MLX Phi-4 | Claude Haiku |
| Gonzo (backend) | `gonzo-backend` | Ollama Qwen-Coder | Claude Sonnet |
| Cleo (frontend) | `cleo-frontend` | MLX Phi-4 | Claude Haiku |
| Jax (creative) | `jax-creative` | Ollama Qwen | Claude Sonnet |

---

## 🧠 Redis Caching Layer

Repeated queries — common in monitoring checks, status polls, and iterative development — are served from [Redis](https://redis.io) without hitting any inference node.

- **TTL:** 600 seconds (10 minutes)
- **Result:** Eliminates duplicate API calls across sessions
- **Location:** Co-located on the Linux server

---

## 👥 The Agent Team

The inference stack powers a team of six specialized AI agents, each with a defined role, domain authority, and personality. Agents are spawned on-demand by the orchestrator and run in isolated sessions.

---

### Skip — Director of Operations (Orchestrator)

> *"Orchestrate, don't just do."*

The primary agent and orchestrator. Routes tasks to specialists, manages cross-agent coordination, and serves as the main point of contact. Enforces completion standards — nothing ships until it's built, deployed, verified, and documented.

**Voice:** Warm Southern drawl. Direct, confident, analytical.
**Authority:** Routes all tasks, resolves conflicts between divisions.
**Model:** `smart-agent` route (MLX Qwen2.5 → Ollama → Claude)

---

### Nellie — Research Specialist (Operations Division)

> *"Ultimate authority on facts."*

Deep research and information synthesis. Deployed for multi-source research, technical accuracy reviews, and knowledge archiving. Jax's scripts must pass Nellie's technical accuracy review before publishing.

**Specialty:** Web research, citation synthesis, structured reports.
**Authority:** Final say on factual accuracy across all agents.
**Model:** `nellie-research` route (Ollama Qwen → Kimi K2.5)

---

### Chief — Operations Monitor (Operations Division)

> *"RED ALERT authority."*

System monitoring, health checks, and infrastructure alerting. Runs periodic checks against the Skip Dashboard, LiteLLM services, and Docker containers. Has absolute veto power over any deployment that threatens server stability.

**Specialty:** Prometheus metrics, Docker health, alerting, anomaly detection.
**Authority:** Can halt all Production tasks to stabilize infrastructure.
**Model:** `chief-monitor` route (MLX Phi-4 → Claude Haiku)

---

### Gonzo — Database Architect (Production Division)

> *"PROTECT THE DATA. OPTIMIZE THE QUERY."*

Backend engineer and database architect. Owns the data layer end to end — schemas, API performance, query optimization. Treats every user input as a potential SQL injection attempt. SELECT * is a sin.

**Specialty:** PostgreSQL, SQLite, Express APIs, schema design.
**Authority:** Owns data layer, API contracts, and database schema.
**Model:** `gonzo-backend` route (Ollama Qwen-Coder → Claude Sonnet)

---

### Cleo — UI/UX Engineer (Production Division)

> *"Pixel perfection. Zero clutter."*

Lead frontend engineer. Owns the user interface, Hugo layouts, and CSS. No inline styles. Semantic HTML only. Div soup is a crime. Worships white space and audibly sighs when asked to make things "pop."

**Specialty:** React, Hugo, Tailwind, accessibility, responsive design.
**Authority:** Owns UI, Hugo layouts, CSS — full veto on visual decisions.
**Model:** `cleo-frontend` route (MLX Phi-4 → Claude Haiku)

---

### Jax — Creative Director (Production Division)

> *"Make technical content actually entertaining."*

Video scripts, narrative structure, and visual storytelling. Fast-talking, highly visual, obsessed with "the hook." Uses film terminology. Every piece of content is structured in three acts: Hook → Conflict → Resolution.

**Specialty:** Video scripts, storyboards, narrative framing.
**Authority:** Owns narrative voice and all video content.
**Model:** `jax-creative` route (Ollama Qwen → Claude Sonnet)

---

## 🏢 Division Structure

```
James (The Architect)
    └── Skip (COO / Orchestrator)
            ├── OPERATIONS DIVISION
            │       ├── Nellie — Research & Facts
            │       └── Chief — Monitoring & Stability
            └── PRODUCTION DIVISION
                    ├── Gonzo — Backend & Data
                    ├── Cleo — Frontend & UI
                    └── Jax — Creative & Scripts
```

**Key rules of engagement:**
- **API Contract:** Gonzo and Cleo agree on JSON structure before writing code
- **Reality Check:** Jax's scripts require Nellie's technical accuracy sign-off
- **Deployment Gate:** Chief has absolute veto on anything threatening server resources
- **80% Rule:** Nothing ships until backend is secure, frontend is responsive, docs are filed, and health checks pass

---

## 📊 Performance Summary

| Route | Model | Latency | Quality | Cost |
| :--- | :--- | :--- | :--- | :--- |
| MLX (Mac) | Qwen2.5-14B | 4.7s | ⭐⭐⭐⭐ | Free |
| Ollama (Linux) | Qwen2.5-14B | 2.7s | ⭐⭐⭐⭐ | Free |
| Ollama (Linux) | Llama3.2 | 0.8s | ⭐⭐⭐ | Free |
| Claude | Haiku | 0.657s | ⭐⭐⭐⭐ | ~$0.08/1M tokens |
| Claude | Sonnet | ~2s | ⭐⭐⭐⭐⭐ | ~$3/1M tokens |

**Cost comparison:**
- Cloud-only equivalent: ~$150–300/month
- This stack: ~$13/month (cloud fallback only)
- **Savings: ~95%**

---

## 🔧 Operational Status

| Component | Status |
| :--- | :--- |
| MLX API (Mac Studio) | ✅ Running (launchd auto-restart) |
| Ollama (Linux Server) | ✅ Running (3 models, 19GB) |
| LiteLLM Proxy | ✅ Running (port 4000) |
| Redis Cache | ✅ Active (10min TTL) |
| Anthropic Fallback | ✅ Configured |
| Agent Team | ✅ All 6 agents operational |

---

## 🔗 Related Projects

- [Home Lab Infrastructure](../home-lab/) — The hardware this stack runs on
- [Skip Dashboard](../home-lab/#-dashboard--management) — Mission control for the whole operation
- [Semantic Memory: Claude Memory System 3.0](../semantic-memory-or-claude-memory-3/) — How agents share and retrieve knowledge
