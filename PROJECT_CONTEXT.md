# god-mode-server — Project Context

**Title:** god-mode-server
**Type:** Infrastructure / DevOps
**Category:** god-mode sub-project
**Location:** `/mnt/d/Projects/personal/god-mode-server/`
**Status:** Live — Post-Migration Verified
**Created:** 2026-02-22
**Owner:** COB

---

## Objective

Convert a MacBook Air M1 (8GB RAM, 1TB external SSD) into a headless always-on server that serves two purposes:

1. **Remote Development Server** — All dev files, projects, god-mode/_bmad module, and toolchain run natively on macOS. Windows machine connects via SSH / VS Code Remote-SSH. Eliminates WSL2 I/O penalty entirely.
2. **Personal Media NAS** — Cloud storage for photos and videos from phone, accessible via SMB file sharing and optionally synced with SyncThing.

---

## Hardware

| Component | Spec | Notes |
|-----------|------|-------|
| Machine | MacBook Air M1 (2020/2021) | Fanless — thermal management required |
| RAM | **8GB** | Primary constraint — strict memory budget |
| Internal Storage | 256GB (assumed) | macOS + apps only |
| External Storage | 1TB SSD (USB-C) | Projects + Media — must auto-mount on reboot |
| Network | 5GHz WiFi | Ethernet adapter recommended for stability |
| Power | USB-C charger (always plugged in) | Battery acts as built-in UPS |

---

## Architecture

```
┌─────────────────────────────────────────────────┐
│  MacBook Air M1 (macOS — Headless Server)       │
│                                                   │
│  ┌─────────────┐  ┌──────────────────────────┐  │
│  │ macOS Native │  │ 1TB External SSD         │  │
│  │ ─ Node.js    │  │ ├── Dev/                 │  │
│  │ ─ Git        │  │ │   ├── _bmad/           │  │
│  │ ─ Claude Code│  │ │   └── god-mode/        │  │
│  │ ─ tmux       │  │ ├── Projects/            │  │
│  │ ─ SSH Server │  │ │   ├── aivate.net/      │  │
│  │              │  │ │   └── personal/         │  │
│  └─────────────┘  │ └── Media/                │  │
│                     │     ├── Photos/           │  │
│  ┌─────────────┐  │     └── Videos/           │  │
│  │ Docker (2GB) │  └──────────────────────────┘  │
│  │ ─ Postgres   │                                 │
│  │ ─ Supabase   │  ┌──────────────────────────┐  │
│  │ ─ Homepage   │  │ Tailscale VPN            │  │
│  └─────────────┘  │ (remote access anywhere)  │  │
│                     └──────────────────────────┘  │
└────────────────────────┬────────────────────────┘
                         │ SSH / SMB / HTTP
        ┌────────────────┼────────────────┐
        │                │                │
   ┌────▼─────┐   ┌─────▼──────┐  ┌─────▼──────┐
   │ Windows  │   │  Phone     │  │  Remote    │
   │ Desktop  │   │ (media     │  │  (laptop / │
   │ (SSH +   │   │  backup)   │  │  travel)   │
   │  VS Code │   │            │  │  via       │
   │  Remote) │   │            │  │  Tailscale │
   └──────────┘   └────────────┘  └────────────┘
```

---

## Implementation Phases

| Phase | Description | Status | Blocked By |
|-------|------------|--------|------------|
| 1 | Physical Setup & Power Management | SUCCESS | Verified |
| 2 | Automate Boot & Login (Headless) | SUCCESS | Verified |
| 3 | SSH & Remote Terminal Access | SUCCESS | Verified |
| 4 | Dev Toolchain Installation | SUCCESS | Verified |
| 5 | External SSD Config & Auto-Mount | SUCCESS | Verified |
| 6 | Development Environment (WSL Mount) | SUCCESS | Verified |
| 7 | Bulk Project Migration | SUCCESS | Verified |
| 8 | Docker (Resource Limits Applied) | SUCCESS | Verified |
| 9 | Media Migration (320GB Sync) | SUCCESS | Verified |
| 10 | Remote Access (Tailscale) | SUCCESS | Verified |
| 10 | Firewall & Security Hardening | Pending | Phase 3 |
| 11 | Dashboard & QoL (Optional) | Pending | Phase 8, 9 |

---

## Known Blockers

### B-001: 8GB RAM Constraint (CRITICAL)
- **Impact:** Limits concurrent workloads — only one dev server + Docker at a time
- **Risk Level:** High
- **Mitigation:** Strict memory budget documented in setup guide. Docker capped at 3GB. No GUI apps on server. Monitor with htop.
- **Worst Case:** If memory pressure causes swapping to SSD, performance degrades and SSD lifespan shortens.
- **Resolution:** Discipline in resource management. Cannot upgrade RAM (soldered on M1).

### B-002: Fanless Thermal Throttling (MEDIUM)
- **Impact:** Sustained CPU load (long builds, multiple containers) causes thermal throttling on fanless M1 Air
- **Risk Level:** Medium
- **Mitigation:** Cooling pad or vertical stand. Avoid running heavy parallel builds. Web dev workloads (Node.js) are generally light.
- **Worst Case:** Prolonged throttling during intensive tasks (Docker builds, large npm installs)
- **Resolution:** Monitor temps. If chronic, consider a clamshell stand with active cooling fan.

### B-003: WiFi Reliability for SSH (MEDIUM)
- **Impact:** WiFi drops = SSH disconnects = lost terminal sessions (if not using tmux)
- **Risk Level:** Medium
- **Mitigation:** Always use tmux. Ethernet adapter strongly recommended.
- **Worst Case:** Frequent disconnects during long operations (git push, npm install)
- **Resolution:** Purchase USB-C Ethernet adapter. Until then, tmux is mandatory.

### B-004: External SSD Auto-Mount After Reboot (MEDIUM)
- **Impact:** If SSD doesn't auto-mount, all projects and media are inaccessible until manual intervention
- **Risk Level:** Medium
- **Mitigation:** Configure fstab entry. Test reboot cycle before going headless.
- **Worst Case:** macOS update resets fstab or changes disk UUID
- **Resolution:** Create a Launch Agent that checks mount status and remounts if needed. Keep disk UUID documented.

### B-005: FileVault Disabled — Physical Security (LOW)
- **Impact:** Auto-login requires FileVault off. If MacBook is stolen, data is exposed.
- **Risk Level:** Low (home server, not mobile)
- **Mitigation:** Physical security of home. All code backed up on GitHub. No plaintext secrets on disk.
- **Worst Case:** Theft = data exposure
- **Resolution:** Acceptable risk for home server. Revisit if server location changes.

### B-006: macOS Updates May Break Headless Config (LOW)
- **Impact:** Major macOS updates can reset auto-login, SSH, or power management settings
- **Risk Level:** Low (infrequent)
- **Mitigation:** Delay major macOS updates. Test on minor updates first. Keep this guide updated.
- **Worst Case:** Update disables auto-login → stuck at login screen → need physical access
- **Resolution:** Keep auto-updates on for security patches only. Manually approve major version upgrades.

### B-007: USB-C Hub / Ethernet Adapter Not Yet Purchased (BLOCKING)
- **Impact:** Cannot achieve wired Ethernet without a USB-C dongle/hub
- **Risk Level:** Blocking for Phase 1
- **Mitigation:** Can start with WiFi, but stability will be lower
- **Resolution:** Purchase a USB-C hub with Ethernet + USB-A + passthrough charging

### B-008: GitHub SSH Keys on New Machine (LOW)
- **Impact:** Need to generate and register new SSH keys for both GitHub accounts (aivaterepositories, Cobb-Simple) on the Mac
- **Risk Level:** Low — straightforward task
- **Mitigation:** Documented in Phase 4 of setup guide
- **Resolution:** Execute during Phase 4

### B-009: _bmad Module Migration Strategy (MEDIUM)
- **Impact:** god-mode/_bmad is currently on Windows at `/mnt/d/Dev/_bmad`. Moving to Mac server means Windows-side agents lose access unless synced.
- **Risk Level:** Medium
- **Mitigation:** Git-based sync. Mac becomes the source of truth. Windows can pull if needed.
- **Worst Case:** Conflicting edits from both machines
- **Resolution:** Establish Mac as primary. Windows becomes thin client only. All agent invocations happen via SSH to Mac.

### B-010: Claude Code MCP Server Re-Configuration (LOW)
- **Impact:** MCP servers (Monday.com, GitHub) are configured in Windows-side Claude Code. Need to reconfigure on Mac.
- **Risk Level:** Low
- **Mitigation:** Documented tokens/endpoints in Mavis memory. Re-add via `claude mcp add` on Mac.
- **Resolution:** Execute during Phase 6

---

## Decisions Log

| Date | Decision | Rationale |
|------|----------|-----------|
| 2026-02-22 | Project created under god-mode umbrella | Infrastructure that supports all dev projects |
| 2026-02-22 | 8GB RAM — Docker capped at 3GB, native dev tools | Maximize available memory for active work |
| 2026-02-22 | Tailscale for remote access (not port forwarding) | Zero-config, secure, free for personal use |
| 2026-02-22 | SMB file sharing for media NAS | Simple, built into macOS, accessible from Windows + phone |
| 2026-02-22 | SyncThing for phone photo backup (not code) | Auto-sync media; Git handles code sync |
| 2026-02-22 | Skip Docker-based IDE (use native + VS Code Remote) | RAM conservation on 8GB machine |

---

## Files

| File | Purpose |
|------|---------|
| `PROJECT_CONTEXT.md` | This file — full project context, blockers, decisions |
| `cloud-server-setup-guide.md` | Step-by-step implementation guide (11 phases) |

---

## Continuous Improvement Notes

This project is iterative. As phases are completed, update this context file with:
- Actual hardware purchased (hub model, Ethernet adapter)
- Performance benchmarks (RAM usage under load, build times vs WSL)
- Any new blockers discovered during setup
- Config values (static IP, Tailscale IP, mount paths) once established
