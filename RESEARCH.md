# 🔬 Homelab Research — What We've Learned

_Compiled from 30+ AI-assisted research sessions covering every aspect of this build. This is the distilled, non-repetitive knowledge base._

---

## 🏗️ Proxmox Cluster Architecture

### The Right Approach for This Hardware

Three Dell OptiPlex Micros (3060 + 3050×2) form a Proxmox VE cluster. These are low-power micro desktops with single NVMe slots and no ECC RAM — the architecture needs to reflect that reality.

**Storage: ZFS Local + Synology NFS**
- **Skip Ceph entirely.** Single drive per node, no 10GbE, and Ceph's OSD daemons eat 2-4GB RAM each. With 32GB per node max, that's unacceptable overhead.
- Local NVMe on each node runs VM boot disks (fast I/O where it matters).
- Synology DS1019+ provides shared NFS storage for ISOs, templates, backups, and any VMs that need live migration.
- ZFS replication (every 15 minutes) between nodes provides pseudo-HA without the complexity of distributed storage.

**Networking**
- Corosync cluster heartbeat gets its own dedicated VLAN — sharing it with VM traffic risks split-brain during high load.
- VLAN-aware Linux bridges on each node let VMs tag into the right network segment.
- HA groups with node priorities ensure critical VMs (Home Assistant, DNS) auto-migrate if a node fails.

### VM Placement Strategy

| Node | CPU | Best For | Why |
|------|-----|----------|-----|
| 3060 Micro | i5-8500T (6C/6T) | HA, Jellyfin, OPNsense | Most cores, best iGPU (UHD 630 QuickSync) |
| 3050 Micro #1 | i5-7500T (4C/4T) | Docker host, monitoring | General workloads |
| 3050 Micro #2 | i5-7500T (4C/4T) | PBS, k3s, overflow | Backup + experimental |

**LXC containers over VMs whenever possible** — 10-20% less overhead. Only use full VMs for: USB passthrough (Home Assistant), different OS kernels (Windows), or strict isolation requirements.

**Don't exceed 80% RAM allocation** on any node — ZFS ARC cache needs headroom for NFS performance.

---

## 🗄️ NAS Philosophy: Storage, Not Compute

The DS1019+ has a Celeron J3455 — every OptiPlex in the rack outperforms it. The research consistently reinforced one principle: **the NAS stores files, Proxmox runs services.**

### What the NAS Should Do
- Serve SMB/NFS shares to the network
- Run Hyper Backup for offsite backups
- Provide NFS datastore for Proxmox
- Run Synology-native packages (Drive, Photos, Surveillance Station)

### What the NAS Should NOT Do
- Run Docker containers (move to Proxmox)
- Host Jellyfin/Plex (no useful transcoding on J3455)
- Run databases (I/O profile is wrong for random writes)

### Storage Configuration
- **SHR-1 on Btrfs** across all 5 bays — single parity maximizes capacity
- Real protection comes from offsite backup, not double parity
- Btrfs gives snapshots, checksums, and deduplication for free
- **Single `/media` share** for the entire *arr stack — hardlinks only work within one share, which means instant imports instead of slow copy operations

### Highest-ROI NAS Upgrade
RAM. The DS1019+ unofficially supports 16GB DDR3L (2×8GB). More RAM = bigger file cache = dramatically faster operations. Costs ~£30 on eBay. Do this before NVMe cache, 2.5G adapters, or anything else.

---

## 🏠 Home Assistant + Zigbee Strategy

### Zigbee2MQTT Over ZHA
Every research session agreed: Z2M wins for a setup this size. It runs as a standalone service (separate LXC on Proxmox), survives HA restarts, has broader device support, and the MQTT backbone feeds data to Grafana/Node-RED/anything else.

### The Pet Problem
With 2 cats and 2 dogs, standard PIR motion sensors fire constantly on pets. The solution is a multi-sensor approach:
- **mmWave presence sensors** (Aqara FP300) at 1.8m+ height — detect stationary humans, ignore floor-level pets
- **PIR sensors** for fast triggering (hallways, stairs)
- **Door contact sensors** as ground truth for room entry/exit
- Pattern: PIR fires → mmWave confirms human presence → automation runs

### House State Machine
The single most impactful automation pattern: define house modes (`home_awake`, `home_sleeping`, `home_away`, `guest_mode`) via an `input_select` entity. Every automation checks the current mode before acting. This eliminates 80% of "why did the lights turn on at 3AM" problems.

### Heating Automation (UK-Specific)
Zigbee TRVs on individual radiators provide room-by-room heating control. Only heat occupied rooms, schedule pre-heat before wake times, pause heating when windows are open. At UK energy prices, this saves 15-25% on gas bills — the hardware pays for itself in one winter.

### Critical Design Rule
**Direct Zigbee binding** for essential switches (bathroom, hallway). If HA crashes or Z2M restarts, these switches still work. Maddy shouldn't notice your infrastructure. WAF (Partner Acceptance Factor) is non-negotiable.

---

## 🌐 Network Segmentation

### VLAN Architecture
Five VLANs on the GoodTop managed switch:

| VLAN | Purpose | Key Rule |
|------|---------|----------|
| 10 — Management | Proxmox UI, switch admin | Only accessible from trusted + Tailscale |
| 20 — Trusted | Personal devices, JANET | Full access to servers |
| 30 — Servers | VMs, containers, NAS | Workload traffic |
| 40 — IoT | Zigbee bridge, smart devices | Jailed: internet for NTP/DNS only |
| 50 — Guest | Visitor WiFi | Internet only, zero local access |

### The Router Question
VLANs without inter-VLAN routing and firewall rules are cosmetic. **OPNsense as a VM on the 3060** handles DHCP, DNS, firewall, and routing. It's free and the 3060 has the CPU headroom. Risk: if that node dies, the network dies. Mitigation: keep an emergency flat-network fallback on the switch's native VLAN.

### Common Breakage When Adding VLANs
- **mDNS stops working** (Chromecast, AirPlay) → enable Avahi reflector in OPNsense
- **NAS access from multiple VLANs** → explicit firewall rules for SMB/NFS ports
- **Tailscale subnet routing** → update advertised routes to include all new subnets

---

## 📊 Monitoring Stack

### Architecture
Prometheus + Grafana + Loki + Uptime Kuma on a single VM/LXC (~2 cores, 2-4GB RAM). Run monitoring **outside** what it monitors — if k3s dies, dashboards should still work.

### Exporters by Host
- **Linux:** node_exporter (all Proxmox nodes, Pi)
- **Windows:** windows_exporter (JANET, HADES)
- **Synology:** snmp_exporter (RAID, temps, volumes)
- **Proxmox:** proxmox-pve-exporter (API-based, cluster-wide)
- **Services:** blackbox_exporter (HTTP/ICMP probes)

### Alerts That Matter
Only alert on things that need human action:
- **Critical (instant):** Node down >2min, disk >90%, RAID degraded, temp >80°C
- **Warning (daily digest):** CPU >80% sustained, cert expiry <14d, backup failure
- **Everything else:** Grafana annotations, not notifications. Alert fatigue kills monitoring.

### The Bar LCD Dashboard
Grafana kiosk mode on the Pi 4B driving the Wisecoco 1280×400 display. At 3.2:1 aspect ratio, use single-row stat panels and gauges — no tall charts. Auto-rotate between cluster health, NAS capacity, network throughput, and service status.

---

## 🔒 Security Model

### Tailscale Is Your Perimeter
Starlink CGNAT means nothing reaches you from the internet. Tailscale provides encrypted, authenticated mesh networking. The gap most people ignore: **the default "allow all" ACL policy**. Write explicit tag-based rules immediately.

### Hardening Checklist (By Priority)
1. **Tailscale ACLs** — tag devices, write explicit rules (20 min, biggest impact)
2. **SSH key-only auth** + fail2ban on all Linux hosts
3. **Synology:** disable default admin, enable 2FA + Auto Block, kill QuickConnect
4. **Proxmox firewall:** enable built-in iptables, default DROP, whitelist Tailscale subnet
5. **CrowdSec** over fail2ban — community threat intel, proactive blocking
6. **Container hygiene:** pin versions, scan with Trivy, audit Docker socket mounts

### Secrets Management
At minimum: `.env` files with `chmod 600`, never committed to Git. Better: Docker secrets or SOPS+age for encrypted secrets in Git repos. Best: Vaultwarden for human credentials + Infisical for service credentials.

---

## ☸️ k3s Kubernetes

### When k3s Makes Sense
k3s sits inside Proxmox as a workload layer for apps that benefit from container orchestration — rolling updates, health checks, secrets management, service discovery. It doesn't replace Docker Compose for simple setups; it complements Proxmox for microservice-style deployments.

### Topology
3 Ubuntu VMs (1 server + 2 agents), ~8 vCPU / 20GB RAM total. Foundation: MetalLB (real LAN IPs), Traefik (ingress), cert-manager (auto TLS), NFS CSI driver (Synology storage).

### What Runs Well on k3s
Stateless/easily-replaceable services: Vaultwarden, Uptime Kuma, Linkding, IT-Tools, Ntfy, Stirling-PDF, Paperless-ngx, n8n, Changedetection.io

### What Doesn't
Home Assistant (USB passthrough), databases (StatefulSet headaches), *arr stack (heavy disk I/O), anything needing GPU passthrough.

### GitOps
Store all manifests in Git, use Flux CD or ArgoCD to auto-reconcile. Push a commit, cluster updates. Infrastructure as code for the house.

---

## 🤖 AI Inference on HADES

384GB RAM is genuinely rare in homelabs. This enables running 70B parameter models entirely in memory on CPU — slow (2-4 tok/s) but functional and fully private.

**Stack:** Ollama + Open WebUI + LiteLLM (unified API proxy)

**GPU Upgrade Path:** A used Tesla P40 (24GB VRAM, ~£120 on eBay) fits the Precision 7910's PCIe 3.0 slots and transforms inference speed to 20+ tok/s on 70B models. Needs a blower cooler mod (~£15).

**Practical Uses:** Private document Q&A via RAG, code review, email triage, meeting transcription (Whisper), automated workflows via n8n.

---

## ⚡ Power & Thermal

### Budget
The entire rack idles at ~84W (~£19/month UK). HADES is separate at ~150W idle. Total homelab: ~£49/month.

### UPS
APC BX750MI (750VA) gives 25 minutes at 100W — enough for graceful shutdown. NUT orchestrates shutdown across all nodes; Synology has native NUT client support. Shutdown order: VMs first → Proxmox hosts → NAS last (flush writes).

### Thermal
Exhaust fans at the top of the rack (Noctua NF-A8 5V), temperature sensor inside rack feeding HA alerts. Layout: heaviest/coolest at bottom, hottest devices toward top, exhaust above everything.

---

## 🎨 Rack Aesthetics

- **Velcro cable ties** (never zip ties) — reusable, won't damage cables
- **Color-coded cables** — blue for LAN, yellow for management, red for WAN
- **Brother P-Touch label maker** — label every cable at both ends, every device
- **WLED on ESP32** for rack lighting — integrates with Home Assistant, automate colors based on service health
- **Blank panels** in every empty U — looks clean, improves airflow direction

---

## 🎮 Fun Projects Worth Doing

1. **ADS-B aircraft tracking** — RTL-SDR dongle (~£30) on the Pi, track RAF Mildenhall/Lakenheath traffic in real-time
2. **Local AI chat** — Ollama on HADES, private ChatGPT with 70B models
3. **Retro gaming** — EmulatorJS for browser-based play, ROMs on Synology
4. **Speedtest tracker** — continuous Starlink performance monitoring with historical graphs
5. **Immich** — self-hosted Google Photos for the household

---

_This document represents the distilled output of ~30 research sessions. The raw research is archived; this is the reference going forward._
