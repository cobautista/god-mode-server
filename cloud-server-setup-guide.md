# Cloud-Server Setup Guide

**Project:** MacBook Air M1 → Headless Development Server + Media NAS
**Hardware:** MacBook Air M1 (8GB RAM) + 1TB External SSD (USB-C)
**Network:** 5GHz WiFi (Ethernet adapter recommended)
**Prepared by:** Mavis — Project Manager
**Date:** 2026-02-22
**Status:** ✅ ALL CORE PHASES COMPLETE (1-7)

> [!NOTE]
> All projects have been successfully migrated to the Mac SSD. Native `_bmad` symlinks are active. The bridge is established via WSL SSHFS.

---

## Table of Contents

1. [Phase 1 — Physical Setup & Power Management](#phase-1--physical-setup--power-management)
2. [Phase 2 — Automate Boot & Login (Headless Operation)](#phase-2--automate-boot--login-headless-operation)
3. [Phase 3 — SSH & Remote Terminal Access](#phase-3--ssh--remote-terminal-access)
4. [Phase 4 — Dev Toolchain Installation](#phase-4--dev-toolchain-installation)
5. [Phase 5 — External SSD Configuration & Auto-Mount](#phase-5--external-ssd-configuration--auto-mount)
6. [Phase 6 — Development Environment Setup](#phase-6--development-environment-setup)
7. [Phase 7 — Media NAS (File Sharing for Photos & Videos)](#phase-7--media-nas-file-sharing-for-photos--videos)
8. [Phase 8 — Docker (Lightweight, Project-Specific Only)](#phase-8--docker-lightweight-project-specific-only)
9. [Phase 9 — Remote Access (Local & External)](#phase-9--remote-access-local--external)
10. [Phase 10 — Firewall & Security Hardening](#phase-10--firewall--security-hardening)
11. [Phase 11 — Dashboard & Quality of Life (Optional)](#phase-11--dashboard--quality-of-life-optional)
12. [8GB RAM Strategy](#8gb-ram-strategy)
13. [Maintenance & Troubleshooting](#maintenance--troubleshooting)

---

## Phase 1 — Physical Setup & Power Management

The MacBook Air's battery acts as a built-in UPS, keeping the server alive during short power outages.

### Steps

1. **Position the MacBook** in a well-ventilated spot (elevated, not on fabric). The M1 Air is fanless — airflow matters.
   - Use a vertical laptop stand or elevate on a cooling pad.
   - Clamshell mode (lid closed) is fine but runs slightly warmer. Open-lid with display off is cooler.

2. **Keep it plugged into power** at all times via USB-C charger.

3. **Get a USB-C hub/dongle** with:
   - USB-C/USB-A port for the external SSD
   - Ethernet port (RJ45) — wired connection is significantly more stable than WiFi for a server
   - USB-C passthrough charging

4. **Disable sleep:**
   - `System Settings → Battery → Options`
   - Set "Turn display off on battery when inactive" → **Never** (or a short time like 5 min — display off is fine)
   - Set "Prevent automatic sleeping when the display is off" → **ON**
   - Plug in the charger, then under `Battery → Power Adapter` set "Turn display off after" → **5 minutes**

5. **Enable Wake for Network Access:**
   - `System Settings → Battery → Options → Wake for network access` → **ON**

---

## Phase 2 — Automate Boot & Login (Headless Operation)

Without automation, a power outage or reboot will leave the Mac stuck at the login screen with no remote access until someone physically logs in.

### Steps

1. **Auto power-on after power failure:**
   - `System Settings → Battery → Options`
   - Enable **"Start up automatically after a power failure"**
   - If this option is not visible in GUI, run in Terminal:
     ```bash
     sudo pmset -a autorestart 1
     ```

2. **Enable automatic login:**
   - `System Settings → Users & Groups → Automatic Login`
   - Select your user account
   - Enter your password to confirm

3. **Security trade-off — Disable FileVault:**
   - Auto-login requires FileVault (disk encryption) to be **OFF**
   - `System Settings → Privacy & Security → FileVault → Turn Off`
   - **Risk:** If someone physically steals the MacBook, they can access the drive without a password
   - **Mitigation:** This is acceptable for a home server in a secure location. Your dev code is also on GitHub (backed up). Sensitive credentials should use macOS Keychain or environment variables, not plain files.

4. **Set login items (apps that start on boot):**
   - `System Settings → General → Login Items`
   - Add all server-critical apps here (Docker Desktop, Tailscale, etc.)
   - For CLI tools and scripts, use a **Launch Agent** (covered in Phase 6)

5. **Verify the full reboot cycle:**
   - Shut down the Mac completely
   - Unplug power, wait 10 seconds, plug back in
   - Confirm it powers on, logs in, and all login items start automatically

---

## Phase 3 — SSH & Remote Terminal Access

**This is your primary connection method.** The YouTube guide missed this entirely — SSH is the backbone of remote development.

### Steps

1. **Enable Remote Login (SSH):**
   - `System Settings → General → Sharing → Remote Login` → **ON**
   - Set "Allow access for" → **Only these users** → add your account (more secure than "All users")

2. **Find your Mac's local IP:**
   ```bash
   ipconfig getifaddr en0
   ```
   (Use `en0` for WiFi, `en1` or similar for Ethernet — check with `ifconfig`)

3. **Assign a static IP on your router:**
   - Log into your router admin panel
   - Find DHCP reservation / static lease settings
   - Assign a fixed IP (e.g., `192.168.1.100`) to the Mac's MAC address
   - This ensures the IP never changes

4. **Test SSH from Windows:**
   - Open Windows Terminal or PowerShell:
     ```bash
     ssh yourusername@192.168.1.100
     ```
   - Accept the fingerprint on first connection
   - You should land in a macOS terminal

5. **Set up SSH key authentication (strongly recommended):**
   - On your **Windows machine**, generate a key pair:
     ```bash
     ssh-keygen -t ed25519 -C "windows-desktop"
     ```
   - Copy the public key to the Mac:
     ```bash
     ssh-copy-id yourusername@192.168.1.100
     ```
   - Test passwordless login:
     ```bash
     ssh yourusername@192.168.1.100
     ```

6. **Create an SSH config alias** on Windows (`~/.ssh/config`):
   ```
   Host macserver
       HostName 192.168.1.100
       User yourusername
       IdentityFile ~/.ssh/id_ed25519
   ```
   Now you can simply type: `ssh macserver`

7. **Disable password authentication** (after confirming key auth works):
   - On the Mac, edit `/etc/ssh/sshd_config`:
     ```
     PasswordAuthentication no
     PubkeyAuthentication yes
     ```
   - Restart SSH: `sudo launchctl stop com.openssh.sshd && sudo launchctl start com.openssh.sshd`

---

## Phase 4 — Dev Toolchain Installation

Run all of these **on the Mac** (via SSH or directly).

### Steps

1. **Install Xcode Command Line Tools:**
   ```bash
   xcode-select --install
   ```

2. **Install Homebrew:**
   ```bash
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
   ```
   Follow the post-install instructions to add Homebrew to your PATH.

3. **Install core dev tools:**
   ```bash
   brew install git node tmux
   ```

4. **Install Claude Code:**
   ```bash
   npm install -g @anthropic-ai/claude-code
   ```

5. **Install tmux (session persistence):**
   - tmux keeps your terminal sessions alive even if SSH disconnects
   - Basic usage:
     ```bash
     tmux new -s dev          # Create a named session
     tmux attach -t dev       # Reattach after disconnect
     tmux ls                  # List sessions
     ```
   - **Critical:** Always work inside a tmux session when SSH'd in. If your WiFi drops, your running processes (dev servers, builds) survive.

6. **Configure Git:**
   ```bash
   git config --global user.name "Your Name"
   git config --global user.email "your@email.com"
   ```

7. **Set up GitHub SSH key on the Mac:**
   ```bash
   ssh-keygen -t ed25519 -C "macserver"
   cat ~/.ssh/id_ed25519.pub
   ```
   Add this public key to your GitHub account(s) (`aivaterepositories`, `Cobb-Simple`).

---

## Phase 5 — External SSD Configuration & Auto-Mount

The external SSD is your primary storage for projects and media. It **must auto-mount on reboot** — otherwise a power outage leaves your projects inaccessible.

### Steps

1. **Format the SSD** (if not already):
   - Use **APFS** (Apple File System) for best macOS performance
   - Disk Utility → Erase → Name: `ServerSSD` → Format: APFS

2. **Find the disk UUID:**
   ```bash
   diskutil info /Volumes/ServerSSD | grep "Volume UUID"
   ```

3. **Create a mount point:**
   ```bash
   sudo mkdir -p /Volumes/ServerSSD
   ```

4. **Configure auto-mount via fstab:**
   ```bash
   sudo vifs
   ```
   Add this line (replace UUID with your actual UUID):
   ```
   UUID=YOUR-UUID-HERE /Volumes/ServerSSD apfs rw,auto 0 0
   ```

5. **Create your directory structure on the SSD:**
   ```bash
   mkdir -p /Volumes/ServerSSD/Dev
   mkdir -p /Volumes/ServerSSD/Projects
   mkdir -p /Volumes/ServerSSD/Media/Photos
   mkdir -p /Volumes/ServerSSD/Media/Videos
   ```

6. **Symlink for convenience:**
   ```bash
   ln -s /Volumes/ServerSSD/Dev ~/Dev
   ln -s /Volumes/ServerSSD/Projects ~/Projects
   ln -s /Volumes/ServerSSD/Media ~/Media
   ```

7. **Test the reboot cycle:**
   - Restart the Mac, verify the SSD auto-mounts and symlinks work

---

## Phase 6 — Development Environment Setup

Migrate your BMAD god-mode and project workflow to the Mac server.

### Steps

1. **Clone god-mode / _bmad module:**
   ```bash
   cd ~/Dev
   git clone git@github.com:your-repo/_bmad.git
   ```
   Or rsync from Windows:
   ```bash
   rsync -avz /mnt/d/Dev/_bmad/ yourusername@192.168.1.100:~/Dev/_bmad/
   ```

2. **Clone your projects:**
   ```bash
   cd ~/Projects
   mkdir -p aivate.net personal
   git clone git@github.com:aivaterepositories/aivate-catalogue.git ~/Projects/aivate.net/aivate-catalogue
   # ... repeat for other projects
   ```

3. **Set up _bmad symlinks** per the Symlink Protocol:
   ```bash
   ln -s ~/Dev/_bmad ~/Projects/aivate.net/aivate-catalogue/_bmad
   # ... repeat for each project
   ```

4. **Configure Claude Code on the Mac:**
   - Run `claude` once to authenticate
   - Add MCP servers (Monday.com, GitHub) as needed
   - Your `/mavis` skill and agent configurations will work via the _bmad symlink

5. **VS Code Remote-SSH (on your Windows machine):**
   - Install the **Remote - SSH** extension in VS Code
   - Press `Ctrl+Shift+P` → "Remote-SSH: Connect to Host" → `macserver`
   - VS Code installs its server component on the Mac automatically
   - You now edit files on the Mac with full IntelliSense, terminal, extensions — all running on macOS natively

6. **Create a Launch Agent for dev services** (optional — auto-start scripts on login):
   - Create `~/Library/LaunchAgents/com.user.devsetup.plist`:
     ```xml
     <?xml version="1.0" encoding="UTF-8"?>
     <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
       "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
     <plist version="1.0">
     <dict>
         <key>Label</key>
         <string>com.user.devsetup</string>
         <key>ProgramArguments</key>
         <array>
             <string>/bin/bash</string>
             <string>/Users/yourusername/scripts/startup.sh</string>
         </array>
         <key>RunAtLoad</key>
         <true/>
     </dict>
     </plist>
     ```
   - Create `~/scripts/startup.sh`:
     ```bash
     #!/bin/bash
     # Start a tmux session with dev servers
     /opt/homebrew/bin/tmux new-session -d -s dev
     ```
   - Load it:
     ```bash
     launchctl load ~/Library/LaunchAgents/com.user.devsetup.plist
     ```

---

## Phase 7 — Media NAS (File Sharing for Photos & Videos)

This enables your Mac server to act as a **personal cloud for media files** — accessible from your phone, Windows PC, and other devices on the network.

### Steps

1. **Enable File Sharing:**
   - `System Settings → General → Sharing → File Sharing` → **ON**

2. **Add shared folders:**
   - Click the `+` button under "Shared Folders"
   - Add `/Volumes/ServerSSD/Media/Photos`
   - Add `/Volumes/ServerSSD/Media/Videos`
   - Optionally add `/Volumes/ServerSSD/Media` as one share

3. **Configure user access:**
   - Click "Options" → ensure **"Share files and folders using SMB"** is checked
   - Under "Users", set permissions:
     - Your account: **Read & Write**
     - Everyone / Guest: **No Access** (unless you want open LAN access)

4. **Access from Windows:**
   - Open File Explorer → address bar: `\\192.168.1.100`
   - Enter Mac username and password when prompted
   - Map as a network drive: Right-click the shared folder → "Map network drive" → assign a letter (e.g., `M:`)

5. **Access from iPhone/Android (photo backup):**
   - Use an app like **Owlfiles** (iOS) or **Cx File Explorer** (Android)
   - Connect via SMB: `smb://192.168.1.100/Media`
   - For **automatic phone photo backup**, consider:
     - **Syncthing** (free, cross-platform) — install on phone + Mac, configure one-way sync from phone Camera folder → `/Volumes/ServerSSD/Media/Photos`
     - This replaces iCloud/Google Photos with your own private cloud

6. **Optional — Organize with folders:**
   ```
   /Volumes/ServerSSD/Media/
   ├── Photos/
   │   ├── 2025/
   │   ├── 2026/
   │   └── Phone-Auto-Backup/
   └── Videos/
       ├── Projects/
       └── Personal/
   ```

---

## Phase 8 — Docker (Lightweight, Project-Specific Only) - SUCCESS

**Configuration for 8GB RAM Management:**
1.  **Memory:** `3.00 GB`
2.  **CPUs:** `4`
3.  **Swap:** `1 GB`

### Steps
1.  Open **Docker Desktop** on the Mac.
2.  Navigate to **Settings → Resources**.
3.  Apply the limits above and click **Apply & Restart**.
4.  Verify performance with `htop`.

---

## Phase 9 — Remote Access (Local & External)

### Local Access (Same Network)

- **SSH:** `ssh macserver` (via config alias)
- **VS Code:** Remote-SSH → `macserver`
- **Dev servers:** Open `http://192.168.1.100:3000` (or any port) in Windows browser
- **NAS/Media:** `\\192.168.1.100` in File Explorer

### Remote Access (Outside Home Network)

1. **Install Tailscale on the Mac:**
   ```bash
   brew install --cask tailscale
   ```
   - Open Tailscale, sign in with your account
   - Add to Login Items so it starts on boot
   - Note the Tailscale IP assigned (e.g., `100.x.y.z`)

2. **Install Tailscale on your Windows machine** and other devices.

3. **Access from anywhere:**
   - `ssh yourusername@100.x.y.z` (Tailscale IP)
   - VS Code Remote-SSH works over Tailscale too
   - Dev servers: `http://100.x.y.z:3000`
   - NAS: `\\100.x.y.z` from Windows Explorer

4. **Tailscale is free** for personal use (up to 100 devices) and requires zero port forwarding or firewall configuration.

### Remote Desktop (Optional)

- **Free option:** macOS built-in Screen Sharing (`System Settings → General → Sharing → Screen Sharing`)
  - Access via VNC from Windows using **RealVNC Viewer** (free) or **TightVNC**
  - Over Tailscale: `vnc://100.x.y.z`
- **Paid option:** Jump Desktop (one-time purchase, smoother experience)
- **Note:** For dev work, SSH + VS Code Remote is faster and lighter than screen sharing. Remote desktop is mainly useful for GUI tasks (Docker Desktop settings, Disk Utility, etc.)

---

## Phase 10 — Firewall & Security Hardening

### Steps

1. **Enable macOS Firewall:**
   - `System Settings → Network → Firewall` → **ON**
   - Click "Options":
     - Allow incoming connections for: SSH, File Sharing, Docker, Tailscale
     - Enable **"Block all incoming connections"** is OFF (would block SSH)
     - Enable **"Stealth mode"** → ON (Mac won't respond to ping from unknown sources)

2. **SSH hardening** (already covered in Phase 3):
   - Key-only authentication (no passwords)
   - Limit to specific users

3. **Keep macOS updated:**
   - `System Settings → General → Software Update → Automatic Updates` → ON
   - Schedule a weekly check: updates may require reboot, but auto power-on + auto-login handles recovery

4. **Sensitive files:**
   - Never store API keys, tokens, or credentials in plain text files
   - Use macOS Keychain or `.env` files excluded from Git
   - The `_bmad` module and god-mode stay local — never pushed to public repos (per existing rules)

---

## Phase 11 — Dashboard & Quality of Life (Optional)

Once everything is running, these are nice-to-haves.

### Homepage Dashboard

A self-hosted web dashboard to manage all your server services from one page.

```bash
docker run -d --name homepage \
  -p 3010:3000 \
  -v /Users/yourusername/.config/homepage:/app/config \
  ghcr.io/gethomepage/homepage:latest
```

Access at `http://192.168.1.100:3010` — configure links to:
- VS Code workspace
- NAS folders
- Docker containers
- Project dev servers
- Tailscale admin

### SyncThing (Selective Use)

**For media files only** — not for code (use Git for code).

If you want phone photos to auto-sync to the Mac:
```bash
brew install syncthing
brew services start syncthing
```
Access SyncThing UI at `http://localhost:8384` on the Mac, pair with your phone.

---

## 8GB RAM Strategy (Refined)

To keep the "Muscle" running at peak performance:
- **Docker VM:** Max 3GB.
- **Native Dev:** Always prefer `npm run dev` natively on Mac over `docker compose up` when possible.
- **MCP Servers:** Run in Docker or natively on Mac (3GB budget is enough for the suite).
- **Toolchain:** Keep `htop` running in a tmux pane to monitor swap usage.

---

## Maintenance & Troubleshooting

### Weekly

- Check disk usage: `df -h /Volumes/ServerSSD`
- Check Docker disk usage: `docker system df`
- Clean unused Docker images: `docker system prune -f`

### If Mac Becomes Unreachable

1. **Check if it's powered on** (physical check or try pinging: `ping 192.168.1.100`)
2. **If powered off after outage:** It should auto-restart. If not, check "Start up automatically after power failure" setting.
3. **If stuck at login:** Auto-login may have been disabled by a macOS update. Need physical access to re-enable.
4. **If SSH fails but Mac is on:** Try Tailscale IP. If both fail, need physical access or remote desktop (if configured).
5. **SSD not mounted:** SSH in, run `diskutil mount /Volumes/ServerSSD` or check `diskutil list`.

### Backup Strategy

- **Code:** GitHub (already covered by existing workflow)
- **Media:** Consider a second backup (external HDD periodically, or a cloud service for critical photos)
- **Server config:** Document everything (this guide) so you can rebuild from scratch if needed

---

## Quick Reference Card

| Action | Command / Location |
|--------|-------------------|
| SSH into Mac | `ssh macserver` |
| VS Code Remote | `Ctrl+Shift+P → Remote-SSH → macserver` |
| Start tmux session | `tmux new -s dev` |
| Reattach tmux | `tmux attach -t dev` |
| Dev server (browser) | `http://192.168.1.100:PORT` |
| NAS from Windows | `\\192.168.1.100` in File Explorer |
| Tailscale (remote) | `ssh user@100.x.y.z` |
| Check RAM | `htop` |
| Check disk | `df -h` |
| Stop all containers | `docker stop $(docker ps -q)` |
| Restart SSH | `sudo launchctl stop com.openssh.sshd && sudo launchctl start com.openssh.sshd` |
