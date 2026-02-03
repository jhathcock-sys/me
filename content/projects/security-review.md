---
title: "Docker Security Review"
date: 2026-02-03
draft: false
---

# Docker Security Audit & Hardening

**Repository:** [github.com/jhathcock-sys/Dockers](https://github.com/jhathcock-sys/Dockers)
**Commit:** `d403912` - Security hardening fixes

## Project Overview

A comprehensive security audit of my homelab Docker infrastructure, identifying critical vulnerabilities and applying hardening measures. This project directly supports my Security+ certification preparation by applying real-world container security concepts.

The audit reviewed 11 Docker Compose files across two servers (ProxMoxBox and Pi5), examining configurations for common security misconfigurations that could lead to container escape, privilege escalation, or data exposure.

---

## Audit Methodology

The security review followed a structured approach based on Docker security best practices and CIS Docker Benchmark guidelines:

| Category | What We Checked |
|----------|-----------------|
| **Privileged Containers** | Containers with `privileged: true` or excessive capabilities |
| **Docker Socket Exposure** | Mounts of `/var/run/docker.sock` without read-only flag |
| **Secrets Management** | Hardcoded passwords, API keys, or insecure defaults |
| **Volume Mounts** | Dangerous mounts (/, /etc, docker.sock) without `:ro` |
| **Image Versioning** | Use of `:latest` tags vs pinned versions |
| **Resource Limits** | Missing memory/CPU constraints |
| **Network Exposure** | Unnecessary port bindings, lack of network isolation |
| **User Permissions** | Containers running as root unnecessarily |

---

## Critical Findings

### 1. cAdvisor Running in Privileged Mode

**Severity:** HIGH
**File:** `proxmox/monitoring/docker-compose.yaml`

**Issue:**
```yaml
cadvisor:
  privileged: true    # Full host access!
  devices:
    - /dev/kmsg
  volumes:
    - /:/rootfs:ro
    - /var/run:/var/run:ro
```

**Risk:** `privileged: true` disables all container isolation. The container has:
- Full access to all host devices
- All Linux capabilities enabled
- Ability to load kernel modules
- Complete container escape vector

**Fix Applied:**
```yaml
cadvisor:
  cap_add:
    - SYS_PTRACE              # Only what's needed for process metrics
  security_opt:
    - no-new-privileges:true  # Prevent privilege escalation
  devices:
    - /dev/kmsg
```

**Why SYS_PTRACE?** cAdvisor needs to inspect process information for container metrics. This single capability is sufficient without granting full privileged access.

---

### 2. Docker Socket Exposure Without Read-Only

**Severity:** HIGH
**Files:** `proxmox/uptime-kuma/docker-compose.yaml`, `proxmox/dockhand/docker-compose.yaml`

**Issue:**
```yaml
volumes:
  - /var/run/docker.sock:/var/run/docker.sock  # Read-write access!
```

**Risk:** The Docker socket is equivalent to root access on the host. A container with write access can:
- Spawn new privileged containers
- Access secrets from any container
- Mount any host path
- Execute commands on the host via container creation

**Fix Applied:**
```yaml
volumes:
  - /var/run/docker.sock:/var/run/docker.sock:ro  # Read-only
```

**Trade-off:** Dockhand may lose some container management features with read-only access. If critical functionality breaks, this specific service can be reverted while maintaining the fix for Uptime Kuma.

---

### 3. Default Password Fallback in Grafana

**Severity:** MEDIUM
**File:** `proxmox/monitoring/docker-compose.yaml`

**Issue:**
```yaml
environment:
  - GF_SECURITY_ADMIN_PASSWORD=${GRAFANA_PASSWORD:-admin}
```

**Risk:** If the `.env` file is missing or `GRAFANA_PASSWORD` is unset, the container starts with `admin` as the password. This creates a security hole that might go unnoticed.

**Fix Applied:**
```yaml
environment:
  - GF_SECURITY_ADMIN_PASSWORD=${GRAFANA_PASSWORD}
```

**Behavior Change:** Container will now fail to start if `GRAFANA_PASSWORD` is not set, making misconfigurations obvious rather than silent.

---

## Additional Findings (Lower Priority)

### Image Version Pinning

**Severity:** MEDIUM
**Affected:** 14 of 15 containers using `:latest`

```yaml
# Current (risky)
image: grafana/grafana:latest

# Recommended
image: grafana/grafana:10.2.3
```

**Risk:** Automatic updates can introduce breaking changes or, in worst case, supply chain attacks. Pinned versions provide:
- Reproducible deployments
- Controlled update process
- Audit trail of version changes

---

### Missing Resource Limits

**Severity:** MEDIUM
**Affected:** Most services lack memory/CPU constraints

```yaml
# Add to services
deploy:
  resources:
    limits:
      memory: 512M
      cpus: '1.0'
```

**Risk:** A single container can consume all host resources, causing denial-of-service to other services.

---

### Network Isolation

**Severity:** LOW
**Status:** All containers on default bridge network

**Recommendation:** Create isolated networks for different service tiers:
- `monitoring` network for Prometheus stack
- `apps` network for user-facing services
- `management` network for Dockhand, Portainer

---

## Security Concepts Applied

This audit reinforced several Security+ exam objectives:

| Domain | Concept | Application |
|--------|---------|-------------|
| **3.0 Architecture** | Least privilege | Replacing `privileged: true` with specific capabilities |
| **3.0 Architecture** | Defense in depth | Multiple layers: read-only mounts, no-new-privileges, resource limits |
| **4.0 Operations** | Hardening | Removing default credentials, pinning versions |
| **4.0 Operations** | Change management | Git-tracked infrastructure changes |
| **5.0 Governance** | Security policies | Documented conventions in CLAUDE.md |

---

## Changes Summary

| Service | Before | After |
|---------|--------|-------|
| **cAdvisor** | `privileged: true` | `cap_add: SYS_PTRACE` + `no-new-privileges` |
| **Uptime Kuma** | `docker.sock` (rw) | `docker.sock:ro` |
| **Dockhand** | `docker.sock` (rw) | `docker.sock:ro` |
| **Grafana** | Password fallback `:-admin` | No fallback (fail if unset) |

---

## Documentation Updates

The security hardening was documented in multiple locations:

1. **homelab-ops/CLAUDE.md** - Added `<security_hardening>` section with:
   - New security conventions
   - Detailed fix documentation
   - Remaining recommendations

2. **Obsidian Homelab Log** - Added Security Hardening section explaining:
   - Why each vulnerability matters
   - Docker socket risks
   - Privileged mode dangers

---

## Remaining Recommendations

| Priority | Task | Status |
|----------|------|--------|
| **HIGH** | Pin all container images to specific versions | Planned |
| **MEDIUM** | Add memory/CPU limits to all services | Planned |
| **MEDIUM** | Add health checks to critical services | Planned |
| **LOW** | Add `no-new-privileges` to all containers | Planned |
| **LOW** | Create isolated Docker networks | Planned |

---

## Lessons Learned

1. **Privileged mode is rarely necessary.** Most containers requesting it can function with specific capabilities instead.

2. **Docker socket is a root-equivalent.** Any container with socket access should be treated as having full host control.

3. **Default credentials are silent failures.** Configurations should fail explicitly rather than fall back to insecure defaults.

4. **Security is iterative.** This audit identified immediate fixes and created a backlog of improvements to implement over time.

---

## Related Projects

- [HomeLab Infrastructure](../home-lab/) - Full homelab documentation
- [GitOps Infrastructure](../gitops/) - How Docker configs are managed via Git
- [Claude Code Memory System](../claude-memory/) - AI-assisted development workflow
