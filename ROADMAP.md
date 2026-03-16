# 🗺️ Roadmap — What To Do Next

_Prioritized action plan for the 10-inch rack build. Ordered by dependency and impact._

---

## Phase 1: Foundation (Week 1-2)
_Get the cluster running. Everything else builds on this._

### 1.1 — Proxmox Cluster Setup
- [ ] Max RAM on all 3 OptiPlexes (32GB each — 2×16GB DDR4 SO-DIMM, ~£40-50/node)
- [ ] Flash Proxmox VE 8.x on all three nodes (NVMe as boot, ZFS single-disk)
- [ ] Create cluster on 3060, join both 3050s
- [ ] Configure Corosync on dedicated network (separate VLAN or subnet)
- [ ] Add Synology NFS storage to all nodes (ISOs, backups, templates)
- [ ] Set up vzdump backup schedule (daily snapshot, 7 daily / 4 weekly / 3 monthly retention)
- [ ] Enable Proxmox firewall (default DROP, whitelist 8006 + SSH + Corosync)

**Cost:** ~£120-150 (RAM upgrades)  
**Time:** 4-6 hours

### 1.2 — NAS Optimization
- [ ] Upgrade DS1019+ RAM to 16GB (2×8GB DDR3L SO-DIMM)
- [ ] Restructure shares: /media, /backups, /photos, /documents, /shared
- [ ] Enable Btrfs snapshots on all shared folders
- [ ] Set up NFS exports for Proxmox (nfsvers=4.1, noatime, hard, intr)
- [ ] Disable Record File Access Time
- [ ] Disable QuickConnect
- [ ] Enable Auto Block + 2FA on DSM

**Cost:** ~£30 (RAM)  
**Time:** 2-3 hours

### 1.3 — Deploy Home Assistant
- [ ] Create HA OS VM on 3060 (2 vCPU, 4GB RAM, local NVMe)
- [ ] USB passthrough for Zigbee coordinator (SONOFF ZBDongle-E if needed, ~£20)
- [ ] Install Zigbee2MQTT + Mosquitto (standalone LXC on Proxmox)
- [ ] Pair all 17 existing Zigbee devices
- [ ] Build first automations: motion lighting, presence detection, goodnight scene
- [ ] Install HACS + Adaptive Lighting + Mushroom cards

**Cost:** £0-20 (coordinator if needed)  
**Time:** 3-4 hours + ongoing iteration

---

## Phase 2: Visibility & Security (Week 3-4)
_See everything, lock everything down._

### 2.1 — Monitoring Stack
- [ ] Deploy Prometheus + Grafana + Loki in LXC (2 vCPU, 2GB RAM)
- [ ] Install node_exporter on all Proxmox nodes
- [ ] Install windows_exporter on JANET + HADES
- [ ] Enable SNMP on Synology, deploy snmp_exporter
- [ ] Deploy Uptime Kuma (separate LXC)
- [ ] Set up Alertmanager → Discord webhook to #logs
- [ ] Import Grafana dashboards: node overview (1860), Proxmox (10347), Synology (14284)
- [ ] Build "Rack Overview" dashboard for the bar LCD

**Cost:** £0  
**Time:** 3-4 hours

### 2.2 — Security Hardening
- [ ] Write Tailscale ACLs — tag all devices, explicit rules per group
- [ ] SSH hardening on all Linux hosts (key-only, fail2ban)
- [ ] Synology: disable default admin, firewall rules, SMB3 minimum
- [ ] Enable 2FA on Proxmox web UI
- [ ] Deploy CrowdSec LAPI + bouncers on each node
- [ ] Audit any Docker socket mounts

**Cost:** £0 (optional: YubiKey ~£45)  
**Time:** 2-3 hours

### 2.3 — Backup Strategy
- [ ] Deploy Proxmox Backup Server on 3050 #2 (VM, datastore on Synology NFS)
- [ ] Configure PBS backup jobs for all VMs/CTs
- [ ] Set up Hyper Backup → Backblaze B2 for photos + documents
- [ ] Set up Hyper Backup → HADES for nightly local backup
- [ ] Buy 2× external USB drives for monthly offsite rotation (~£240)
- [ ] Schedule first test restore

**Cost:** ~£240 (USB drives) + ~£5/month (B2)  
**Time:** 3-4 hours

---

## Phase 3: Services & Network (Week 5-8)
_Start running things and segment the network._

### 3.1 — Core Services
- [ ] Deploy reverse proxy (Traefik or Nginx Proxy Manager) — LXC on 3060
- [ ] Deploy Vaultwarden — LXC, <50MB RAM
- [ ] Deploy AdGuard Home ×2 (one per node for DNS failover)
- [ ] Deploy Jellyfin on 3060 LXC (QuickSync HW transcoding via /dev/dri passthrough)
- [ ] Deploy *arr stack (Sonarr, Radarr, Prowlarr, Bazarr, qBittorrent) — Docker VM on 3050 #1
- [ ] Deploy Overseerr for media requests
- [ ] Deploy Paperless-ngx for document management
- [ ] Deploy Immich for photo backup

**Cost:** £0  
**Time:** 6-8 hours across multiple sessions

### 3.2 — VLAN Segmentation
- [ ] Configure VLANs on GoodTop switch (10, 20, 30, 40, 50)
- [ ] Deploy OPNsense VM on 3060 (2 vCPU, 2GB RAM)
- [ ] Configure DHCP per VLAN on OPNsense
- [ ] Set up inter-VLAN firewall rules (IoT jailed, guest internet-only)
- [ ] Update Proxmox bridges to VLAN-aware
- [ ] Move devices to correct VLANs one at a time
- [ ] Enable Avahi mDNS reflector for Chromecast/AirPlay
- [ ] Install Tailscale on OPNsense as subnet router

**Cost:** £0  
**Time:** Half a day (fiddly, test thoroughly)

### 3.3 — Home Automation Expansion
- [ ] Buy mmWave sensors (Aqara FP300 ×3) for pet-aware presence (~£90)
- [ ] Buy door/window contact sensors ×4 (~£28)
- [ ] Buy power monitoring plugs ×2 (washing machine, rack) (~£30)
- [ ] Buy Zigbee TRVs ×4 for room-by-room heating (~£100)
- [ ] Implement house state machine (home_awake / sleeping / away / guest)
- [ ] Build heating automations with TRVs
- [ ] Set up security notifications (door sensors + night mode)
- [ ] Direct-bind critical light switches for HA-independent operation

**Cost:** ~£250  
**Time:** Ongoing over weeks

---

## Phase 4: Advanced & Fun (Month 2-3)
_The cool stuff. Only after the foundation is solid._

### 4.1 — k3s Kubernetes (Optional)
- [ ] Provision 3 Ubuntu VMs on Proxmox (1 server + 2 agents)
- [ ] Install k3s with MetalLB + cert-manager
- [ ] Bootstrap Flux CD or ArgoCD with a Git repo
- [ ] Deploy first workloads: Uptime Kuma, Linkding, IT-Tools
- [ ] Configure NFS CSI driver for Synology storage
- [ ] Migrate suitable services from Docker Compose to k3s

**Cost:** £0  
**Time:** Weekend project

### 4.2 — Local AI on HADES
- [ ] Create Ubuntu VM on HADES (256GB RAM, 32 vCPU)
- [ ] Install Ollama + Open WebUI
- [ ] Pull models: qwen2.5:32b, codestral:22b, nomic-embed-text
- [ ] Set up RAG pipeline for document Q&A (ChromaDB)
- [ ] Optional: Buy Tesla P40 GPU (~£120) for 10× inference speed
- [ ] Expose via Tailscale for access from anywhere

**Cost:** £0-135  
**Time:** 2-3 hours

### 4.3 — Pi Status Display
- [ ] Flash Raspberry Pi OS Lite on Pi 4B
- [ ] Configure display (`hdmi_cvt=1280 400 60 6 0 0 0`)
- [ ] Set up Chromium kiosk mode → Grafana dashboard
- [ ] Build rack-specific dashboard (single-row stat panels for 1280×400)
- [ ] Configure playlist rotation (cluster → NAS → network → HA)
- [ ] Add motion-activated display wake (PIR sensor, ~£3)
- [ ] Night mode auto-dimming via cron

**Cost:** ~£3 (PIR sensor)  
**Time:** 2-3 hours

### 4.4 — Fun Projects
- [ ] ADS-B aircraft tracking — RTL-SDR dongle on Pi (~£30)
- [ ] Speedtest tracker — log Starlink performance
- [ ] Mealie — recipe manager for the household
- [ ] EmulatorJS — browser-based retro gaming, ROMs on NAS
- [ ] WLED on ESP32 — rack lighting integrated with HA

---

## Phase 5: Polish (Month 3+)
_Make it bulletproof and beautiful._

### 5.1 — UPS & Power Management
- [ ] Buy APC BX750MI UPS (~£80)
- [ ] Install NUT on primary Proxmox node
- [ ] Configure Synology as NUT client
- [ ] Set up HA automation: alert on battery, flash lights red on power outage
- [ ] Smart plug on rack for energy monitoring → Grafana dashboard
- [ ] Configure WoL for HADES (sleep when idle, wake on demand)

**Cost:** ~£95  
**Time:** 2 hours

### 5.2 — Rack Aesthetics
- [ ] Brother P-Touch label maker + TZe-FX flexible tape (~£40)
- [ ] Label every cable at both ends
- [ ] Install WLED + ESP32 in rack (~£10)
- [ ] Fill empty U spaces with blank panels
- [ ] Route power left, network right
- [ ] Final cable management pass with Velcro ties
- [ ] Take the "after" photo for r/homelab

**Cost:** ~£50  
**Time:** 2-3 hours

---

## 💰 Total Budget Summary

| Phase | Estimated Cost |
|-------|---------------|
| Phase 1: Foundation | ~£170-200 |
| Phase 2: Visibility & Security | ~£240-285 |
| Phase 3: Services & Network | ~£250 |
| Phase 4: Advanced & Fun | ~£3-165 |
| Phase 5: Polish | ~£145 |
| **Total (all phases)** | **~£810-1,045** |
| **Monthly recurring** | ~£5/mo (Backblaze B2) |

Most of this is hardware (RAM, UPS, sensors, USB backup drives). All software is free and open source.

---

## 📋 Quick Reference: Weekend Sprint

If you had one weekend to get the most value:

**Saturday Morning:** Flash Proxmox on all 3 nodes, create cluster, add NFS storage  
**Saturday Afternoon:** Deploy HA VM, pair Zigbee devices, first automations  
**Saturday Evening:** Deploy monitoring (Prometheus + Grafana + Uptime Kuma)  
**Sunday Morning:** NAS optimization + Hyper Backup to B2  
**Sunday Afternoon:** Deploy Jellyfin + *arr stack  
**Sunday Evening:** Security hardening pass (Tailscale ACLs, SSH, Synology)

That gives you a working, monitored, backed-up, media-serving homelab in 48 hours.

---

_Last updated: 16 March 2026 — Janet 🐟_
