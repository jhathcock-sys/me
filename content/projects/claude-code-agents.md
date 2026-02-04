---
title: "Claude Code Custom Agent System"
date: 2026-02-04
draft: false
tags: ["AI", "Automation", "DevOps", "Security", "Infrastructure"]
categories: ["Automation"]
summary: "Built a specialized sub-agent system for automated homelab infrastructure validation, security auditing, and documentation synchronization using Claude Code's Task API."
---

## Overview

Created a custom agent framework leveraging Claude Code's sub-agent capabilities to automate security auditing, infrastructure validation, and documentation maintenance for a Docker-based homelab environment.

**Tech Stack**: Claude Code CLI, Markdown (agent definitions), Bash, Docker Compose, Obsidian

**Repository**: [ai-assistant-config](https://github.com/jhathcock-sys/ai-assistant-config)

---

## The Problem

Managing a homelab with 20+ Docker containers across multiple hosts requires constant vigilance:

- **Security drift** - Configurations degrade over time (privileged containers, weak passwords, exposed ports)
- **Port conflicts** - New services collide with existing allocations
- **Documentation rot** - Obsidian vault falls behind infrastructure changes
- **Convention violations** - Docker Compose files don't follow established patterns

**Manual audits are time-consuming and error-prone.** Each security review took 30+ minutes and missed edge cases.

---

## The Solution

Built four specialized AI agents that work in parallel to automate validation:

### 1. Security Reviewer Agent

**Purpose**: Audit Docker Compose stacks for vulnerabilities

**Checks**:
- Privileged mode abuse (`privileged: true`)
- Docker socket permissions (must be `:ro`)
- Hardcoded secrets / default passwords
- Image tag pinning (no `:latest` in production)
- Security options (`no-new-privileges:true`)
- Resource limits (DoS prevention)
- Network exposure + UFW firewall rules

**First Audit Results**: Found 3 critical issues in monitoring stack (9 containers):
- Missing `.env` file (deployment blocker)
- 7 services missing `security_opt: no-new-privileges:true`
- 8 exposed ports without firewall rules

**Output**: Generated ready-to-use UFW script restricting all monitoring ports to LAN-only access.

---

### 2. Infrastructure Validator Agent

**Purpose**: Pre-deployment conflict detection

**Checks**:
- Port conflicts (vs 15+ known allocations across 3 hosts)
- IP address conflicts (192.168.1.0/24 topology)
- Volume path existence
- Docker Compose syntax validation
- Resource allocation (prevents OOM on 8GB hosts)
- Network configuration issues

**Example**: Prevents deploying Jellyfin on port 3000 (already used by Dockhand).

---

### 3. Documentation Sync Agent

**Purpose**: Keep Obsidian vault synchronized with infrastructure

**Checks**:
- Missing service documentation pages
- Outdated port/IP references
- Broken WikiLinks (`[[Internal Links]]`)
- Missing README files in stacks
- Stale change dates (docs vs code)

**Example**: Detects when monitoring stack changes on 2026-02-04 but documentation last updated 2026-01-15.

---

### 4. Deployment Helper Agent

**Purpose**: Enforce Docker Compose conventions

**Checks**:
- Restart policies (`unless-stopped` required)
- Security options (privilege escalation prevention)
- Memory limits (all services must define)
- Environment variables (no default password fallbacks)
- Volume mount patterns (prefer local paths)
- Health checks (for dependency orchestration)

**Example**: Validates that all 20 containers follow the established pattern before git commit.

---

## Technical Implementation

### Agent Architecture

Each agent is a markdown file with YAML frontmatter:

```yaml
---
name: security-reviewer
description: Security audit for Docker Compose stacks
model: sonnet
subagent_type: general-purpose
---

[Detailed agent prompt with checks, context, and output format]
```

**Location**: `~/.claude/agents/`
**Backup**: Git-versioned in `ai-assistant-config` repository

### Parallel Execution

Agents can run simultaneously for fast validation:

```bash
# Single message launches 3 agents in parallel
"Review proxmox/jellyfin/ with security-reviewer, infra-validator, and deploy-helper in parallel"

# Results combined into single report
```

### Context Integration

Agents access homelab state from structured files:

| File | Contains |
|------|----------|
| `~/.claude/CLAUDE.md` | Global preferences, core directives |
| `~/homelab-ops/CLAUDE.md` | Port allocations, IP topology, resource limits |
| `~/Documents/HomeLab/HomeLab/_Index.md` | Documentation structure |

**Example**: Infrastructure validator knows that ProxMoxBox has 8GB RAM and 9.5GB already allocated (1.19x overcommit), so it warns when new services push allocation above safe thresholds.

---

## Key Features

### 1. Educational Security+ Integration

Security reviewer provides exam-relevant context:

- **Attack Frameworks** - Privilege escalation vectors
- **Configuration Management** - Image pinning, secrets management
- **Defense in Depth** - Multiple security layers
- **CIA Triad** - Resource limits ensure availability
- **Supply Chain Security** - Container image provenance

**Example output**:
> "This prevents a common privilege escalation vector where a compromised process tries to exploit setuid binaries (like `sudo`, `passwd`) to gain root. It's like removing the ladder that attackers use to climb out of the container."

---

### 2. Actionable Remediation

Agents don't just report problems - they provide **copy-paste fixes**:

**Security Issue**:
```
❌ Missing security_opt on 7 services
```

**Agent-Provided Fix**:
```yaml
prometheus:
  image: prom/prometheus:v2.49.1
  security_opt:
    - no-new-privileges:true
  # ... rest of config
```

**UFW Firewall Script** (generated):
```bash
#!/bin/bash
sudo ufw allow from 192.168.1.0/24 to any port 9090 proto tcp comment 'Prometheus - LAN only'
sudo ufw deny 9090/tcp comment 'Block Prometheus from internet'
# ... (16 rules total)
```

---

### 3. Risk Scoring

Each service gets a security score:

| Service | Image Pin | Security Opt | Resources | Network | Overall |
|---------|-----------|--------------|-----------|---------|---------|
| Prometheus | ⚠️ :latest | ⚠️ Missing | ✅ 768M | ⚠️ 9090 | **Medium** |
| cAdvisor | ⚠️ :latest | ✅ **Done** | ✅ 512M | ⚠️ 8081 | **Low** |

**Prioritization**: Fix criticals first, defer low-risk issues.

---

### 4. Fail-Secure Design

Agents validate that security controls are **required**, not optional:

**Bad (default fallback)**:
```yaml
environment:
  - ADMIN_PASSWORD=${ADMIN_PASSWORD:-admin}  # Fails open
```

**Good (required explicit value)**:
```yaml
environment:
  - ADMIN_PASSWORD=${ADMIN_PASSWORD}  # Fails closed
```

Security reviewer **validates absence of fallback operators** in password fields.

---

## Results & Impact

### Security Improvements

**Before Agents**:
- Manual audits every 2-3 weeks
- Inconsistent security configurations
- Port conflicts discovered at runtime
- Documentation lagged behind changes

**After Agents**:
- ✅ Automated pre-commit validation
- ✅ Consistent security patterns enforced
- ✅ Zero port conflicts (caught before deployment)
- ✅ Documentation audit in <30 seconds

### First Security Audit Findings

**Stack**: proxmox/monitoring/ (9 containers)
**Duration**: ~2 minutes (vs 30+ minutes manual)

**Findings**:
- 1 critical issue (missing `.env` file)
- 3 high-priority issues (image pinning, security options, firewall rules)
- 3 low-priority informational items (all acceptable by design)

**Generated Outputs**:
- 153KB comprehensive security report
- UFW firewall script (16 rules)
- Specific version recommendations for 9 images
- Security+ exam context for all findings

---

## Workflows

### New Service Deployment

```bash
# 1. Create docker-compose.yaml

# 2. Validate in parallel (single command)
"Review proxmox/jellyfin/ with security-reviewer, infra-validator, and deploy-helper in parallel"

# 3. Fix issues

# 4. Deploy
cd ~/homelab-ops/proxmox/jellyfin && docker-compose up -d

# 5. Update docs
"Use doc-sync to find what documentation needs creating"
```

### Pre-Commit Validation

```bash
# Run before every commit
"Run deploy-helper and security-reviewer on modified compose files in parallel"
```

### Monthly Security Audit

```bash
# Audit all stacks simultaneously
"Use security-reviewer to audit proxmox/dockhand, proxmox/monitoring, proxmox/homepage, and proxmox/minecraft in parallel"
```

---

## Technical Challenges

### 1. Agent Context Limits

**Problem**: Agents need access to infrastructure state (ports, IPs, resources) but context windows are limited.

**Solution**: Structured CLAUDE.md files in each repository with:
- Network topology (IP allocations)
- Port allocations by service
- Resource limits by host
- Known exceptions

Agents read these files first, then cross-reference during validation.

---

### 2. Parallel Agent Coordination

**Problem**: Running 3+ agents simultaneously could produce redundant or conflicting reports.

**Solution**: Each agent has a **distinct scope**:
- Security reviewer → Vulnerabilities only
- Infrastructure validator → Conflicts only
- Deployment helper → Convention compliance only

No overlap = clean combined reports.

---

### 3. Actionable vs Informational Findings

**Problem**: Early agent versions generated 50+ findings per stack, causing alert fatigue.

**Solution**: Severity-based prioritization:
- **CRITICAL** (red) → Deployment blockers
- **HIGH** (orange) → Before production
- **MEDIUM** (yellow) → Next maintenance
- **LOW** (blue) → Informational only

Only CRITICAL/HIGH require immediate action.

---

## Future Enhancements

### Planned Agents

- **Backup Validator** - Verify backup jobs, test restore procedures
- **Certificate Manager** - Track SSL certificate expiration
- **Log Analyzer** - Parse Loki logs for patterns, anomalies
- **Performance Profiler** - Identify resource bottlenecks

### Feature Ideas

- **Agent chaining** - Output of one agent feeds into another
- **Scheduled runs** - Cron-based daily security audits
- **Dashboard integration** - Display agent results in Homepage
- **Notification system** - Slack/Discord alerts for critical findings

---

## Lessons Learned

### 1. Structured Context is Critical

Agents need **machine-readable state files** to make decisions. Unstructured documentation doesn't work.

**Good**: CLAUDE.md with clear port allocation table
**Bad**: "Prometheus runs somewhere around port 9000-ish"

---

### 2. Agents Should Provide Fixes, Not Just Problems

"Port 3000 is in use" → Frustrating
"Port 3000 used by Dockhand. Next available: 3002" → Actionable

Include **copy-paste solutions** in every finding.

---

### 3. Educational Context Adds Value

Since I'm studying for Security+, agents that explain **why** a vulnerability matters are more valuable than simple checklists.

Example: "This blocks setuid exploitation" → Generic
"This prevents container escape by blocking exploitation of setuid binaries like sudo" → Educational

---

### 4. Fail-Secure is Better Than Fail-Open

When agents can't determine if something is secure, they should **flag it for review** rather than assume it's fine.

Unknown configuration → ⚠️ Warn for manual review
Don't silently pass uncertain states.

---

## Code Samples

### Agent Definition Structure

```markdown
---
name: security-reviewer
description: Security audit for Docker Compose stacks
model: sonnet
subagent_type: general-purpose
---

# Security Reviewer Agent

You are a security-focused agent specialized in auditing Docker Compose stacks.

## Context

**Network**: 192.168.1.0/24
**Target User**: System Administrator studying Security+

## Your Mission

Scan Docker Compose files for vulnerabilities. Provide actionable recommendations with educational context.

## Critical Security Checks

### 1. Privileged Mode (CRITICAL)
[Detailed check logic, examples, remediation]

### 2. Docker Socket Mounts (HIGH)
[Detailed check logic, examples, remediation]

[... 6 more checks ...]

## Output Format

[Structured report template with severity levels]
```

---

### UFW Firewall Script (Generated)

```bash
#!/bin/bash
# Generated by security-reviewer agent
# Purpose: Restrict monitoring services to LAN-only

set -euo pipefail

# Allow from LAN only
sudo ufw allow from 192.168.1.0/24 to any port 9090 proto tcp comment 'Prometheus - LAN only'
sudo ufw allow from 192.168.1.0/24 to any port 3030 proto tcp comment 'Grafana - LAN only'

# Explicit deny from internet
sudo ufw deny 9090/tcp comment 'Block Prometheus from internet'
sudo ufw deny 3030/tcp comment 'Block Grafana from internet'

echo "✓ UFW rules applied"
```

---

## Metrics

| Metric | Before Agents | After Agents |
|--------|--------------|--------------|
| Security audit time | 30+ minutes | ~2 minutes |
| Port conflicts detected | At runtime (fail) | Pre-deployment (prevent) |
| Documentation freshness | Manual checks | Automated sync |
| Convention compliance | Inconsistent | Enforced |
| Security+ study value | Separate research | Integrated into output |

---

## Related Projects

- [Container Resource Management](/projects/container-resource-management) - Memory limit implementation
- [Security Review](/projects/security-review) - Manual hardening process
- [Obsidian Vault Restructure](/projects/obsidian-vault-restructure) - Documentation system

---

## Conclusion

Building custom AI agents transformed homelab infrastructure management from reactive (fix issues after they occur) to proactive (prevent issues before deployment).

**Key Takeaway**: AI agents are most valuable when they:
1. Have structured context (not just documentation)
2. Provide actionable fixes (not just reports)
3. Run in parallel (fast feedback loops)
4. Integrate with existing workflows (pre-commit, pre-deploy)

The agent system has become part of my standard deployment workflow - every new service is validated by 3+ agents before going live.

---

**Source Code**: [GitHub - ai-assistant-config/agents](https://github.com/jhathcock-sys/ai-assistant-config)

**Documentation**: [Obsidian Vault - Claude Code Agents](https://github.com/jhathcock-sys/homelab-docs)
