# god-mode-server Setup Plan

This plan outlines the steps to convert your MacBook Air M1 into a headless development server and media NAS. Since this is a manual setup on physical hardware, I will guide you phase by phase.

## Proposed Steps

### Phase 1: Physical Setup & Power Management
- [x] Position MacBook in a ventilated spot (elevated or vertical stand).
- [x] Connect to power (USB-C).
- [x] Configure Battery/Power settings to prevent sleep and enable wake for network access.

### Phase 2: Automate Boot & Login
- [x] Enable "Start up automatically after power failure".
- [x] Enable automatic login for your user account.
- [x] Disable FileVault (Required for auto-login).
- [x] Add Docker/Tailscale to Login Items.

### Phase 3: SSH & Remote Terminal Access
- [x] Verify Tailscale connection between Windows and Mac (Check Admin Console).
- [x] Enable Remote Login (SSH) on the Mac (Username: `cob`).
- [x] Set up a static IP or DHCP reservation (Optional if using Tailscale).
- [x] Configure SSH key-based authentication from your Windows machine.
- [x] Create an SSH alias (`macserver`) on Windows using Tailscale IP.

### Phase 4: Dev Toolchain & god-mode Migration
- [x] Verify existing toolchain (Xcode, Homebrew, Git, Node.js installed).
- [x] Install missing component: **tmux**.
- [x] Claude Code installed.

### Phase 5: Storage Optimization & god-mode Migration
- [x] Identify External SSD (`Cob-SSD`).
- [x] Resolve "Operation not permitted" (Full Disk Access).
- [x] Rebuild SSD structure on clean slate (`Projects`, `Media`).
- [x] Force recreate standard symlinks (`~/Projects`, `~/Media`).
- [x] Migrate `_bmad` to internal Mac storage: `~/God-Mode-Module/_bmad`.
- [x] Sync project repos (`fine-dining-landing`) to SSD.
- [x] Establish `_bmad` symlinks within projects.

### Phase 6: Remote Development (WSL-Native Mounting - SUCCESS)
- [x] Install `sshfs` in WSL (Ubuntu): `sudo apt install sshfs`.
- [x] Create mount point: `mkdir ~/macserver`.
- [x] Mount Mac Projects: `sshfs cob@100.72.109.37:/Volumes/Cob-SSD/Projects ~/macserver`.
- [x] Open `~/macserver/fine-dining-landing` in Antigravity.

### Phase 7: Bulk Project Migration (Windows to Mac SSD - SUCCESS)
- [x] Define migration batches (`aivate.net`, `personal`, `simple.biz`).
- [x] Execute `rsync` transfers from WSL to Mac Server.
- [x] Verify directory integrity on `Cob-SSD`.
- [x] Re-establish `_bmad` engine symlinks on the Mac filesystem.

> [!TIP]
> All 19 projects are now natively hosted on the Mac's Silicon core, eliminating network overhead for processing tasks.

## Verification Plan

### Manual Verification
- **Reboot Test:** After Phase 2, we will reboot the Mac to ensure it logs in automatically without a keyboard/monitor.
- **SSH Connectivity:** After Phase 3, we will verify that `ssh macserver` works from your Windows terminal without a password.
- **Remote Edit:** After Phase 6, we will verify that VS Code Remote-SSH can open and edit files on the Mac.
