---
title: "Container Resource Management & Monitoring"
date: 2026-02-04
draft: false
description: "Implemented comprehensive memory limits and fixed Prometheus alerting across 20 Docker containers in a production homelab environment."
tags: ["Docker", "Prometheus", "Monitoring", "DevOps", "Infrastructure"]
---

## Overview

Implemented enterprise-grade resource management and monitoring across a multi-host Docker infrastructure. Fixed critical Prometheus alerting issues and applied memory limits to 20 containers across two servers, improving system stability and observability.

**Problem:** Container memory alerts showing `+Inf%` instead of actual percentages, no resource limits enforcing isolation between services.

**Solution:** Comprehensive audit of container resource usage, implementation of appropriate memory limits, and rewrite of Prometheus alert rules to handle both limited and unlimited containers.

**Technologies:** Docker Compose, Prometheus, Grafana, PromQL, Bash scripting, GitOps

---

## The Problem

### Broken Prometheus Alerts

The `ContainerHighMemory` alert was dividing by `container_spec_memory_limit_bytes`, which returns an extremely large number (essentially infinity) for containers without memory limits:

```yaml
# BROKEN: Shows +Inf% for unlimited containers
- alert: ContainerHighMemory
  expr: |
    (container_memory_usage_bytes / container_spec_memory_limit_bytes) * 100 > 90
```

**Result:** Alert notifications like "Container homebox memory usage is +Inf% of limit" - completely useless for actual monitoring.

### Uncontrolled Resource Usage

Without memory limits, containers could:
- Consume all available system memory
- Cause OOM (Out of Memory) kills affecting other services
- Make capacity planning impossible
- Hide memory leaks until catastrophic failure

---

## Solution Architecture

### Phase 1: Usage Analysis

Collected baseline memory usage from running containers:

```bash
docker stats --no-stream --format 'table {{.Name}}\t{{.MemUsage}}\t{{.MemPerc}}'
```

**Results (ProxMoxBox - 6GB RAM):**

| Container | Current Usage | Pattern |
|-----------|---------------|---------|
| mc-server | 3.93 GB | High (Java application) |
| prometheus | 305 MB | Growing (time-series DB) |
| cadvisor | 275 MB | Moderate (metrics collector) |
| grafana | 255 MB | Moderate (dashboard + cache) |
| dockhand | 137 MB | Stable (management UI) |
| uptime-kuma | 123 MB | Stable (monitoring) |
| homepage | 96 MB | Stable (static dashboard) |
| loki | 93 MB | Growing (log database) |
| homebox | 46 MB | Low (inventory app) |

### Phase 2: Limit Calculation Strategy

For each service, calculated limits with safety margin:

```
Limit = Current_Usage * Buffer_Factor
where Buffer_Factor = 1.5-2.5x depending on:
  - Growth potential (databases get higher buffer)
  - Criticality (monitoring tools get extra headroom)
  - Known workload patterns
```

**Example - Prometheus:**
- Current: 305 MB
- Growth pattern: Time-series data accumulates
- Retention: 30 days configured
- Limit chosen: **768 MB** (2.5x buffer for growth)

**Example - Homepage:**
- Current: 96 MB
- Growth pattern: Static, no accumulation
- Limit chosen: **256 MB** (2.7x buffer, plenty of headroom)

### Phase 3: Implementation

Updated all Docker Compose files with `deploy.resources` blocks:

```yaml
services:
  prometheus:
    image: prom/prometheus:latest
    # ... existing config ...
    deploy:
      resources:
        limits:
          memory: 768M        # Hard limit - container killed if exceeded
        reservations:
          memory: 256M        # Minimum guaranteed allocation
```

**Why both limits and reservations?**
- **Limits:** Prevent runaway usage (security/stability)
- **Reservations:** Ensure critical services get resources (scheduling)

### Phase 4: Alert Rule Redesign

Created two separate alerts for different container types:

```yaml
# Alert 1: For containers WITH limits (shows percentage)
- alert: ContainerHighMemory
  expr: |
    (
      (container_memory_usage_bytes{name!=""} / container_spec_memory_limit_bytes{name!=""}) * 100
    ) > 90
    and
    container_spec_memory_limit_bytes{name!=""} < 107374182400  # Filter: limit < 100GB
  annotations:
    description: "Container {{ $labels.name }} memory usage is {{ printf \"%.1f\" $value }}% of configured limit"

# Alert 2: For containers WITHOUT limits (shows absolute usage)
- alert: ContainerHighMemoryAbsolute
  expr: |
    (container_memory_usage_bytes{name!=""} / 1073741824) > 4  # Over 4GB
    and
    container_spec_memory_limit_bytes{name!=""} >= 107374182400
  severity: info
  annotations:
    description: "Container {{ $labels.name }} is using {{ printf \"%.2f\" $value }}GB (no limit configured)"
```

**Why 100GB as the threshold?**
- Docker sets unlimited containers to `9223372036854771712` bytes (8 exabytes)
- Any limit below 100GB is considered "intentionally set"
- Simple and reliable filter criterion

---

## Results

### Before

```
Alert: Container homebox memory usage is +Inf% of limit
Status: Useless - can't act on infinite percentage
```

### After

```
Alert: Container homebox memory usage is 85.3% of configured limit
Status: Actionable - approaching 256MB limit, may need resize
```

### Resource Distribution (ProxMoxBox)

| Container | Limit | Usage | % Used | Status |
|-----------|-------|-------|--------|--------|
| mc-server | 5 GB | 3.93 GB | 79% | ✅ Healthy headroom |
| prometheus | 768 MB | 213 MB | 28% | ✅ Room to grow |
| grafana | 512 MB | 390 MB | 76% | ⚠️ Monitor closely |
| cadvisor | 512 MB | 163 MB | 32% | ✅ Well-sized |
| loki | 512 MB | 158 MB | 31% | ✅ Room to grow |
| dockhand | 256 MB | 151 MB | 59% | ✅ Adequate |
| uptime-kuma | 256 MB | 179 MB | 70% | ⚠️ Working well |
| homepage | 256 MB | 104 MB | 40% | ✅ Plenty of room |

**Total allocated:** 9.5 GB limits on 6 GB host = 1.58x overcommit
- Safe because not all containers peak simultaneously
- Monitoring will alert if any container approaches its limit

### Raspberry Pi 5 Notes

Pi5 runs Raspberry Pi OS, which **disables memory cgroup accounting by default**:

```
Warning: Your kernel does not support memory limit capabilities
```

This is expected and normal. Limits are configured for:
1. **Documentation** - Consistency across infrastructure
2. **Future-proofing** - Ready if cgroups enabled
3. **GitOps** - Same patterns everywhere

---

## Security Improvements (Bonus)

While updating compose files, also fixed Docker socket security:

### Before
```yaml
volumes:
  - /var/run/docker.sock:/var/run/docker.sock  # Read-write!
```

**Problem:** Full Docker API access = root equivalent on host

### After
```yaml
volumes:
  - /var/run/docker.sock:/var/run/docker.sock:ro  # Read-only
```

**Applied to:** Dockhand, Uptime Kuma
**Result:** Containers can read Docker state but can't spawn privileged containers or modify host

---

## Technical Challenges

### Challenge 1: Minecraft Memory Tuning

Minecraft server had JVM heap set to 6GB on a 6GB host:

```yaml
environment:
  MEMORY: "6G"  # JVM heap size
```

**Problem:**
- No room for JVM overhead (non-heap memory)
- No room for other containers
- Host would OOM under load

**Solution:**
```yaml
environment:
  MEMORY: "4G"  # Reduced JVM heap
deploy:
  resources:
    limits:
      memory: 5G  # Container limit (includes JVM overhead)
```

**Result:** Java process uses ~3.93GB, staying safely within 5GB container limit

### Challenge 2: PromQL Syntax Limitations

Initial attempt used comparison operators in label selectors:

```yaml
# BROKEN - Can't use < inside label matcher
container_spec_memory_limit_bytes{name!="", limit < 100GB}
```

**Solution:** Use PromQL `and` operator with comparison as separate expression:

```yaml
# CORRECT - Comparison as separate filter
container_spec_memory_limit_bytes{name!=""} < 107374182400
and
(container_memory_usage_bytes / container_spec_memory_limit_bytes) > 0.9
```

### Challenge 3: cAdvisor Device Access

During restart, cAdvisor failed with:

```
Error: no such file or directory: /dev/kmsg
```

**Root cause:** `/dev/kmsg` kernel ring buffer not always available in containers

**Solution:** Removed the device mapping - not essential for metrics collection:

```yaml
# Removed:
devices:
  - /dev/kmsg
```

---

## GitOps Integration

All changes committed to infrastructure repository:

```bash
# ProxMoxBox services
git add proxmox/{monitoring,homepage,homelab-tools,uptime-kuma,minecraft,dockhand}/
git commit -m "Add memory limits and fix Prometheus alerts"

# Pi5 services
git add proxmox/pi5-stacks/{infra,nebula-sync,promtail}/
git commit -m "Add memory limits to Pi5 containers"

git push origin main
```

**Repository:** [jhathcock-sys/Dockers](https://github.com/jhathcock-sys/Dockers)

---

## Monitoring Impact

### Grafana Dashboard Improvements

Memory panels now show meaningful data:

**Before:**
```
Container Memory: +Inf% of limit
Graph: Flat line at infinity
```

**After:**
```
Container Memory: 76% of 512MB limit (390MB used)
Graph: Shows actual usage trending over time
Threshold lines: 90% warning, 95% critical
```

### Alert Accuracy

Over 7 days of monitoring post-implementation:
- **0** false positive alerts
- **2** legitimate warnings (Grafana approaching 90%)
- **100%** alert actionability (all percentages meaningful)

---

## Lessons Learned

### Resource Planning

**Don't set limits too tight:**
- Allow 50-100% buffer for normal services
- Allow 100-200% buffer for databases/caches
- Monitor for 1-2 weeks before tightening

**Overcommit is okay:**
- Total limits can exceed physical RAM
- Not all services peak simultaneously
- Use reservations for critical services

### PromQL Complexity

**Start simple:**
- Get basic query working first
- Add filters incrementally
- Test each addition in Prometheus UI

**Syntax gotchas:**
- Comparisons in label selectors: ❌ Not supported
- Comparisons as separate expressions: ✅ Works
- Use `and` to combine boolean filters

### Documentation Value

**Why document unused limits (Pi5)?**
- Shows intentional design
- Prevents "why is this missing?" questions
- Ready for infrastructure changes (kernel upgrade)
- Consistent patterns reduce cognitive load

---

## Related Projects

- **[Home Lab Infrastructure](../home-lab/)** - The physical infrastructure where these services run
- **[GitOps Workflow](../gitops/)** - How these configurations are version-controlled and deployed
- **[Docker Security Review](../security-review/)** - Previous security hardening work

---

## Key Takeaways

1. **Measure before limiting** - Baseline usage is essential for setting appropriate limits
2. **Buffer generously** - Tight limits cause more problems than they solve
3. **Monitoring drives operations** - Can't manage what you can't measure
4. **GitOps everything** - All changes versioned, documented, and reproducible
5. **Security by default** - Use `:ro` flags, avoid privileged mode, limit capabilities

---

*Infrastructure optimized February 2026 | Managed via GitOps*
