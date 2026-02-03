---
title: "Claude Code Memory System"
date: 2026-02-02
draft: false
---

# 🧠 Claude Code Memory System

Building a structured context system for AI-assisted infrastructure development.

## Project Overview

Claude Code is Anthropic's CLI tool for AI-assisted development. While powerful out of the box, it lacks persistent memory between sessions. This project documents how I built a structured memory file (`CLAUDE.md`) to provide consistent context, enforce operational standards, and create a personalized AI engineering assistant.

**Key Insight:** The quality of AI assistance is directly proportional to the quality of context you provide. A well-structured memory file transforms a general-purpose AI into a specialized team member who understands your infrastructure, preferences, and standards.

---

## 🎯 Problem Statement

Without persistent context, each Claude Code session starts fresh:
- No knowledge of my infrastructure or IP assignments
- No understanding of my preferred tools (Docker Compose vs docker run)
- No awareness of security requirements or coding standards
- Repeated explanations of the same architecture
- Generic suggestions that don't fit my environment

**The Frustration Cycle:**
```
Session 1: "My Docker host is at 192.168.1.4, I use Dockhand for management..."
Session 2: "Remember, my Docker host is at 192.168.1.4..."
Session 3: "As I mentioned before, 192.168.1.4 is my Docker host..."
```

**Goal:** Create a memory system that makes Claude Code act as a Senior Systems Architect who already knows my homelab inside and out—every session, from the first prompt.

---

## 📐 Why XML Over Markdown Tables?

My initial `CLAUDE.md` used markdown tables—the standard approach most people use. But I discovered significant limitations when dealing with complex infrastructure documentation:

### The Limitations of Flat Tables

| Markdown Tables | XML Structure |
|-----------------|---------------|
| Flat, hard to nest | Hierarchical, supports relationships |
| Ambiguous parsing | Explicit element boundaries |
| Limited metadata | Attributes for properties |
| Repetitive headers | Semantic grouping |
| Context spread across multiple tables | Related data grouped together |

### Example 1: Service Definitions

*Markdown approach (what most people use):*
```markdown
| Server | Service | Port | Notes |
|--------|---------|------|-------|
| ProxMoxBox | Grafana | 3030 | Monitoring dashboards |
| ProxMoxBox | Prometheus | 9090 | Metrics collection |
| ProxMoxBox | Loki | 3101 | Log aggregation |
| Pi5 | Pi-hole | 53, 8080 | Secondary DNS |
| Pi5 | Mealie | 9925 | Recipe management |
```

*XML approach (what I implemented):*
```xml
<services>
    <server name="ProxMoxBox" ip="192.168.1.4">
        <service name="Grafana" port="3030" note="Monitoring dashboards" />
        <service name="Prometheus" port="9090" note="Metrics collection" />
        <service name="Loki" port="3101" note="Log aggregation" />
    </server>
    <server name="Pi5" ip="192.168.1.234">
        <service name="Pi-hole" ports="53, 8080" note="Secondary DNS" />
        <service name="Mealie" port="9925" note="Recipe management" />
    </server>
</services>
```

**Why XML wins here:**
- Services are grouped by server—no scanning through rows to find what's on ProxMoxBox
- The IP address is attached to the server, not repeated for every service
- Relationships are explicit: Grafana *belongs to* ProxMoxBox
- Easy to add server-level attributes (IP, role, OS) without new columns

### Example 2: Access Credentials and Endpoints

*Markdown approach:*
```markdown
## SSH Access
- ProxMoxBox: `ssh root@192.168.1.4`
- Pi5: `ssh cib@192.168.1.234`

## Web Interfaces
| Service | URL | Credentials |
|---------|-----|-------------|
| Grafana | http://192.168.1.4:3030 | admin / ******** |
| Wazuh | https://192.168.1.7 | admin / ************ |
```

*XML approach:*
```xml
<access_points>
    <ssh target="ProxMoxBox">ssh root@192.168.1.4</ssh>
    <ssh target="Pi5">ssh cib@192.168.1.234</ssh>
    <ssh target="Wazuh">ssh root@192.168.1.7</ssh>

    <web_interface name="Grafana" url="http://192.168.1.4:3030" creds="admin / ********" />
    <web_interface name="Wazuh Dashboard" url="https://192.168.1.7" creds="admin / ************************" />
    <web_interface name="Prometheus" url="http://192.168.1.4:9090" />
    <web_interface name="Alertmanager" url="http://192.168.1.4:9093" note="Discord notifications" />
</access_points>
```

**Why XML wins here:**
- All access information in one semantic block
- Optional attributes (creds, note) only appear when relevant
- Consistent structure for both SSH and web interfaces
- The AI can query "how do I access Wazuh?" and find both SSH and web access together

### Example 3: Project Path Mapping

*Markdown approach:*
```markdown
| Server | Git Path | Deploy Path |
|--------|----------|-------------|
| ProxMoxBox | homelab-ops/proxmox/<stack>/ | /opt/<stack>/ |
| Pi5 | homelab-ops/pi5/<stack>/ | /opt/pi5-stacks/<stack>/ |
```

*XML approach:*
```xml
<stack_mapping>
    <server name="ProxMoxBox">
        <git_path>homelab-ops/proxmox/&lt;stack&gt;/</git_path>
        <deploy_path>/opt/&lt;stack&gt;/</deploy_path>
    </server>
    <server name="Pi5">
        <git_path>homelab-ops/pi5/&lt;stack&gt;/</git_path>
        <deploy_path>/opt/pi5-stacks/&lt;stack&gt;/</deploy_path>
        <note>Managed via Hawser agent</note>
    </server>
</stack_mapping>
```

**Why XML wins here:**
- The Pi5-specific note about Hawser management is attached to Pi5, not in a separate section
- When the AI helps deploy to Pi5, it immediately sees the Hawser context

---

## 🏗️ Memory Architecture

The memory file is organized into logical sections, each serving a specific purpose:

### 1. Metadata Section

Basic profile information that personalizes interactions:

```xml
<meta_data>
    <user_name>James</user_name>
    <location>Delaware, EST (UTC-5)</location>
    <status>Job Hunting (Target: SysAdmin, IT Director, Network/Sec)</status>
    <years_experience>30+</years_experience>
    <key_strengths>Networking, Windows, Project Management</key_strengths>
</meta_data>
```

**Why it matters:** The AI knows:
- My experience level → Don't over-explain networking basics, but do explain newer concepts like GitOps
- My career focus → Emphasize enterprise patterns and security best practices
- My timezone → Relevant for scheduling, cron jobs, log timestamps
- My strengths → Leverage my networking knowledge, help me grow in coding/automation

**Real-world impact:**
```
Without metadata: "A subnet is a logical division of an IP network..."
With metadata: "Since you're familiar with networking, I'll skip the subnet basics.
               For your 192.168.1.0/24 network, here's the VLAN design..."
```

### 2. Persona Definition

This is where the magic happens—defining *how* the AI should behave:

```xml
<persona>
    <role>Senior Systems Architect and DevOps Engineer</role>
    <mission>Help build a secure, enterprise-grade homelab</mission>
    <core_directives>
        <directive name="Idempotency">
            Always prefer solutions that can be run multiple times without breaking things
            (e.g., check if a directory exists before creating it, use Ansible/Terraform
            logic where possible).
        </directive>
        <directive name="Security First">
            Since James is studying for Security+, prioritize least-privilege access.
            Do not suggest chmod 777. Always suggest firewall rules (UFW) for new services.
        </directive>
        <directive name="Network Awareness">
            Network is 192.168.1.0/24. Check infrastructure topology for used static IPs
            (.3, .4, .5, .6, .7, .234, .253) before suggesting a new static IP.
        </directive>
        <directive name="Documentation">
            When writing scripts, include comments explaining why specific flags or
            settings are used (educational value for learning).
        </directive>
        <directive name="Docker Compose Only">
            Never provide docker run commands. Always provide a docker-compose.yml file.
            For system config changes, provide a bash script or Ansible playbook snippet.
        </directive>
    </core_directives>
</persona>
```

**Deep Dive: Why Each Directive Matters**

#### Idempotency Directive

**The Problem:** Scripts that work once but fail on re-run:
```bash
# BAD: Fails if directory exists
mkdir /opt/myapp
```

**What the AI now provides:**
```bash
# Create directory if it doesn't exist (idempotent)
# -p flag: creates parent directories and doesn't error if dir exists
mkdir -p /opt/myapp
```

**Real example from our work:**
```bash
#!/bin/bash
# Install node-exporter (idempotent installation script)

# Check if already installed to avoid redundant work
if systemctl is-active --quiet node_exporter; then
    echo "node_exporter already running, skipping installation"
    exit 0
fi

# Download only if binary doesn't exist
if [ ! -f /usr/local/bin/node_exporter ]; then
    wget -q https://github.com/.../node_exporter-1.7.0.linux-amd64.tar.gz
    tar xzf node_exporter-*.tar.gz
    mv node_exporter-*/node_exporter /usr/local/bin/
fi
```

#### Security First Directive

**The Problem:** Lazy permissions that create vulnerabilities:
```bash
# BAD: World-writable, massive security hole
chmod 777 /opt/myapp
chmod 777 /var/log/myapp.log
```

**What the AI now provides:**
```bash
# Set ownership to the service account (principle of least privilege)
# Only the app user needs write access, others get read-only
chown -R myapp:myapp /opt/myapp
chmod 750 /opt/myapp           # Owner: rwx, Group: r-x, Others: ---
chmod 640 /opt/myapp/config    # Owner: rw-, Group: r--, Others: ---

# Add UFW rule for the new service
# Only allow traffic on the specific port needed
ufw allow 8080/tcp comment 'MyApp web interface'
```

**Why this matters for Security+ studies:**
- Every suggestion reinforces least-privilege concepts
- Firewall rules are included by default, not as an afterthought
- I'm learning security best practices through practical application

#### Network Awareness Directive

**The Problem:** IP conflicts from blind suggestions:
```
Me: "I need a static IP for my new VM"
AI (without context): "Use 192.168.1.10"
Me: "That's already my NAS!"
```

**What the AI now provides:**
```
Me: "I need a static IP for my new VM"
AI (with context): "Looking at your topology, these IPs are in use:
    - .3 (Pi-hole), .4 (ProxMoxBox), .5 (NAS), .6 (NPM), .7 (Wazuh),
    - .234 (Pi5), .253 (Proxmox)

    I'd suggest 192.168.1.8 for your new VM. It's:
    - Sequential with your server range (.3-.7)
    - Below .234 which you're using for IoT/secondary devices
    - Easy to remember in your naming scheme"
```

#### Docker Compose Only Directive

**The Problem:** One-off docker run commands that aren't reproducible:
```bash
# BAD: How do I remember these flags next time? What if I need to modify it?
docker run -d --name grafana -p 3030:3000 -v grafana-data:/var/lib/grafana \
  -e GF_SECURITY_ADMIN_PASSWORD=secret grafana/grafana:latest
```

**What the AI now provides:**
```yaml
# docker-compose.yml - Version controlled, reproducible, self-documenting
services:
  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    restart: unless-stopped
    ports:
      - "3030:3000"    # External:Internal - access on port 3030
    volumes:
      - grafana-data:/var/lib/grafana    # Persist dashboards and settings
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=${GRAFANA_ADMIN_PASSWORD}  # From .env file

volumes:
  grafana-data:        # Named volume for data persistence
```

**Benefits:**
- Checked into Git → version history, rollback capability
- Self-documenting with comments
- Secrets in `.env` file, not command history
- Consistent with my GitOps workflow

### 3. Interaction Rules

Beyond technical context, I defined *how* I want to work:

```xml
<interaction_rules>
    <rule>Provide detailed explanations with thorough code comments.</rule>
    <rule>Always ASK before making changes to infrastructure.</rule>
    <rule>Preferred workflow: Focused bursts followed by breaks.</rule>
    <rule>Environment preference: CLI for Linux, GUI for Windows.</rule>
</interaction_rules>
```

**Real-world impact of "Always ASK before making changes":**

```
Without rule:
AI: "I've updated your prometheus.yml and restarted the container."
Me: "Wait, I wasn't ready! That just broke my monitoring during an incident!"

With rule:
AI: "I've prepared the prometheus.yml changes. Here's the diff:
     [shows changes]

     Should I apply these changes? Note: This will require restarting
     Prometheus, which will cause a brief gap in metrics collection."
Me: "Let me wait until after hours to apply this."
```

### 4. Infrastructure Topology

Complete network documentation in a queryable format:

```xml
<infrastructure>
    <topology>
        <device ip="192.168.1.3" name="Primary Pi-hole" role="Main DNS (ns1.home.lab)" />
        <device ip="192.168.1.4" name="ProxMoxBox (Dell R430)" role="Main Docker Host, Dockhand" />
        <device ip="192.168.1.5" name="Synology NAS (DS220j)" role="Network Storage (DSM)" />
        <device ip="192.168.1.6" name="Nginx Proxy Manager" role="Reverse Proxy" />
        <device ip="192.168.1.7" name="Wazuh VM (Debian 12)" role="SIEM v4.14.2" />
        <device ip="192.168.1.234" name="Pi5 (Raspberry Pi 5)" role="Secondary DNS, Tailscale, Mealie" />
        <device ip="192.168.1.253" name="Proxmox" role="Hypervisor" />
    </topology>

    <access_points>
        <ssh target="ProxMoxBox">ssh root@192.168.1.4</ssh>
        <ssh target="Pi5">ssh cib@192.168.1.234</ssh>
        <ssh target="Wazuh">ssh root@192.168.1.7</ssh>
        <web_interface name="Dockhand" url="http://192.168.1.4:3000" />
        <web_interface name="Grafana" url="http://192.168.1.4:3030" creds="admin / ********" />
        <web_interface name="Wazuh Dashboard" url="https://192.168.1.7" creds="admin / ************" />
    </access_points>

    <services>
        <server name="ProxMoxBox" ip="192.168.1.4">
            <service name="Dockhand" port="3000" />
            <service name="Homepage" port="4000" />
            <service name="Homebox" port="3100" />
            <service name="Grafana" port="3030" />
            <service name="Prometheus" port="9090" />
            <service name="Alertmanager" port="9093" />
            <service name="Loki" port="3101" />
            <service name="Node Exporter" port="9100" />
            <service name="cAdvisor" port="8081" />
        </server>
        <server name="Pi5" ip="192.168.1.234">
            <service name="Pi-hole" ports="53, 8080" />
            <service name="Mealie" port="9925" />
            <service name="Node Exporter" port="9100" />
            <service name="Promtail" />
        </server>
    </services>
</infrastructure>
```

**Why this level of detail matters:**

When I say "add node-exporter to the Pi-hole LXC," the AI knows:
1. Pi-hole is at 192.168.1.3
2. It's an LXC container (not a full VM or Docker)
3. Node Exporter should use port 9100 (consistent with other hosts)
4. Prometheus at 192.168.1.4:9090 needs a new scrape target
5. The Grafana dashboards are at 192.168.1.4:3030

One sentence from me triggers a complete, context-aware response.

### 5. Monitoring Configuration

Alert thresholds and agent inventory ensure consistency:

```xml
<monitoring_config>
    <dashboards>
        <dashboard name="Homelab Overview" path="/d/homelab-overview" note="Single pane of glass" />
        <dashboard name="Docker Containers" path="/d/docker-containers" note="Container metrics" />
        <dashboard name="Loki Logs" />
        <dashboard name="Node Exporter Full" id="1860" />
        <dashboard name="cAdvisor" id="14282" />
    </dashboards>

    <alerts>
        <threshold metric="Disk Warning" value=">80%" />
        <threshold metric="Disk Critical" value=">90%" />
        <threshold metric="Memory" value=">90% for 5m" />
        <threshold metric="CPU" value=">90% for 5m" />
        <threshold metric="Host Down" value="unreachable 2m" severity="critical" />
        <threshold metric="Container Down" value="missing 2m" />
        <threshold metric="Target Down" value="scrape fail 2m" severity="critical" />
    </alerts>

    <wazuh_agents>
        <agent id="001" name="SRV-DOCKER01" host="ProxMoxBox (192.168.1.4)" />
        <agent id="002" name="pi-infra" host="Pi5 (192.168.1.234)" />
        <agent id="003" name="SRV-DNS01" host="Pi-hole LXC (192.168.1.3)" />
        <agent id="004" name="SRV-NPM01" host="NPM LXC (192.168.1.6)" />
    </wazuh_agents>
</monitoring_config>
```

**Why this matters:**

When adding a new host, the AI automatically suggests:
- Prometheus alert rules matching my existing thresholds (not arbitrary values)
- Wazuh agent registration with consistent naming conventions (SRV-*, descriptive names)
- Dashboard updates to include the new host

### 6. Project Definitions

Links repositories to their purposes and deployment paths:

```xml
<projects>
    <project name="Homelab Ops">
        <path_local>/home/cib/homelab-ops</path_local>
        <repo>github.com:jhathcock-sys/Dockers.git</repo>
        <stack_mapping>
            <server name="ProxMoxBox">
                <git_path>homelab-ops/proxmox/&lt;stack&gt;/</git_path>
                <deploy_path>/opt/&lt;stack&gt;/</deploy_path>
            </server>
            <server name="Pi5">
                <git_path>homelab-ops/pi5/&lt;stack&gt;/</git_path>
                <deploy_path>/opt/pi5-stacks/&lt;stack&gt;/</deploy_path>
                <note>Managed via Hawser agent</note>
            </server>
        </stack_mapping>
    </project>

    <project name="Portfolio Site">
        <path_local>/home/cib/my-portfolio</path_local>
        <repo>github.com:jhathcock-sys/me.git</repo>
        <live_url>https://jhathcock-sys.github.io/me/</live_url>
        <tech_stack>Hugo, PaperMod Theme</tech_stack>
        <commands>
            <cmd description="Local Dev">hugo server -D</cmd>
            <cmd description="Build">hugo</cmd>
            <cmd description="Deploy">git push (auto GitHub Pages)</cmd>
        </commands>
    </project>
</projects>
```

---

## 📊 Before and After: Real Examples

### Example 1: Deploying a New Service

**Before (no memory):**
```
Me: "I want to deploy Uptime Kuma"

AI: "Here's how to run Uptime Kuma:
     docker run -d --name uptime-kuma -p 3001:3001 \
       -v uptime-kuma:/app/data louislam/uptime-kuma:1

     Access it at http://localhost:3001"
```

**After (with memory):**
```
Me: "I want to deploy Uptime Kuma"

AI: "I'll create a docker-compose.yml for Uptime Kuma on ProxMoxBox.

     Based on your infrastructure:
     - Port 3001 is available on 192.168.1.4
     - I'll place it at /opt/uptime-kuma/ following your stack structure
     - I'll add comments explaining each configuration option

     Here's the compose file:

     # /opt/uptime-kuma/docker-compose.yml
     services:
       uptime-kuma:
         image: louislam/uptime-kuma:1
         container_name: uptime-kuma
         restart: unless-stopped    # Auto-restart on failure or reboot
         ports:
           - "3001:3001"            # Web UI - internal and external same
         volumes:
           - ./data:/app/data       # Persist monitors, settings, history

     Should I also:
     1. Add a UFW rule: ufw allow 3001/tcp comment 'Uptime Kuma'
     2. Add it to your Homepage dashboard?
     3. Create a Prometheus scrape target for /metrics?"
```

### Example 2: Troubleshooting

**Before (no memory):**
```
Me: "Grafana can't reach Prometheus"

AI: "Check if Prometheus is running with 'docker ps'.
     Verify the datasource URL in Grafana settings.
     Check firewall rules between the containers."
```

**After (with memory):**
```
Me: "Grafana can't reach Prometheus"

AI: "Let me help troubleshoot. In your setup:
     - Grafana: 192.168.1.4:3030
     - Prometheus: 192.168.1.4:9090
     - Both are in the monitoring stack at /opt/monitoring/

     Since they're on the same host and Docker network, let's check:

     1. Verify Prometheus is healthy:
        curl http://192.168.1.4:9090/-/healthy

     2. Check if they're on the same Docker network:
        docker network inspect monitoring_default

     3. In Grafana, the datasource URL should be:
        http://prometheus:9090 (using container name, not IP)

     4. Check your docker-compose.yml - both services should be
        in the same compose file or explicitly share a network.

     Want me to SSH to ProxMoxBox and run these diagnostics?"
```

### Example 3: Adding Monitoring to a New Host

**Before (no memory):**
```
Me: "Add monitoring to my Wazuh VM"

AI: "Install node_exporter on the VM and add it to your Prometheus config."
```

**After (with memory):**
```
Me: "Add monitoring to my Wazuh VM"

AI: "I'll add full monitoring to your Wazuh VM (192.168.1.7, Debian 12).
     Here's the plan:

     1. **Node Exporter** (system metrics):
        # Idempotent installation script with educational comments
        [provides full bash script]

        UFW rule: ufw allow from 192.168.1.4 to any port 9100 proto tcp \
                  comment 'Prometheus scraping'

     2. **Prometheus scrape target**:
        # Add to /opt/monitoring/prometheus/prometheus.yml
        - job_name: 'wazuh'
          static_configs:
            - targets: ['192.168.1.7:9100']
              labels:
                instance: 'wazuh-vm'
                environment: 'production'

     3. **Promtail** (logs to Loki):
        [provides docker-compose.yml for Promtail]

     4. **Wazuh agent** - Already installed (ID: not yet assigned)
        Should I register the VM as a Wazuh agent monitoring itself?

     5. **Grafana dashboard** - The Node Exporter Full dashboard (ID: 1860)
        will automatically show the new host.

     Should I proceed with step 1 (Node Exporter installation)?"
```

---

## 🎓 Lessons Learned

### 1. Structure Matters More Than Content Volume

A small, well-organized memory file outperforms a large, unstructured one. The AI can find and use `<device ip="192.168.1.4">` faster than parsing paragraphs of prose.

### 2. Directives Prevent Mistakes Before They Happen

Instead of fixing bad suggestions, the directives prevent them entirely:
- "No chmod 777" → Never have to reject insecure permissions
- "Docker Compose only" → Never have to ask for a compose file
- "Always ask before changes" → Never have accidental modifications

### 3. Context Compounds Over Time

Each piece of information builds on others:
- Knowing the IP → Knowing the services → Knowing the ports → Knowing the access method

The AI can make increasingly sophisticated suggestions as the context grows.

### 4. Personas Shape Behavior Dramatically

Defining a role ("Senior Systems Architect") changes:
- **Vocabulary:** Uses proper terminology, not oversimplified explanations
- **Assumptions:** Expects familiarity with enterprise concepts
- **Suggestions:** Recommends production-grade solutions, not quick hacks
- **Depth:** Explains the "why" behind recommendations

### 5. Educational Value is Built-In

The "Documentation" directive means every script teaches something:
```bash
# -p flag: creates parent directories AND doesn't error if directory exists
# This makes the script idempotent (safe to run multiple times)
mkdir -p /opt/monitoring/prometheus
```

I'm learning Linux administration, security practices, and DevOps patterns through practical application, not just documentation reading.

---

## 📁 File Locations

Claude Code supports multiple memory files with different scopes:

| File | Scope | Purpose |
|------|-------|---------|
| `~/.claude/CLAUDE.md` | Global | Loaded for ALL projects - personal preferences, infrastructure |
| `<project>/CLAUDE.md` | Project | Checked into git - shared with team, project-specific context |
| `<project>/CLAUDE.local.md` | Session | Gitignored - private notes, temporary context, session history |

**My Setup:**
- **Global (`~/.claude/CLAUDE.md`):** Infrastructure topology, personas, directives—everything in this article
- **Project (`homelab-ops/CLAUDE.md`):** Repository structure, deployment conventions, service ports
- **Local (`homelab-ops/CLAUDE.local.md`):** Session notes, work-in-progress, troubleshooting history

---

## 🔧 Implementation Tips

### Start Small, Iterate Often

Don't try to document everything at once. Start with:
1. Basic infrastructure (IPs, hostnames)
2. One or two critical directives
3. Expand based on what you find yourself repeating

### Use Real Examples

When adding directives, include examples of what you want and don't want:
```xml
<directive name="Security First">
    Do not suggest: chmod 777, running as root unnecessarily
    Do suggest: Specific user permissions, UFW rules, least privilege
</directive>
```

### Keep It Current

Update the memory file when infrastructure changes. An outdated memory file is worse than none—it causes confident but wrong suggestions.

### Test Your Directives

After adding a directive, test it:
- Add "Docker Compose Only" → Ask for a way to run nginx → Should get compose file, not docker run

---

## 🔗 Related Projects

- [HomeLab Infrastructure](../home-lab/) - The infrastructure this memory system documents
- [GitOps Workflow](../gitops/) - Docker Compose management with version control

---

## 📋 Future Enhancements

- [ ] Add `<runbooks>` section for common procedures (backup restore, certificate renewal)
- [ ] Include `<troubleshooting>` patterns for known issues
- [ ] Create project-specific CLAUDE.md files for each stack
- [ ] Document MCP (Model Context Protocol) server integrations for tool access
- [ ] Add `<maintenance_windows>` to inform AI about acceptable change times
- [ ] Include `<dependencies>` mapping between services for impact analysis
