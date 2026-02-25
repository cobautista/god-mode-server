# Phase 16: Mobile Media Cloud Sync

This phase transforms the MacBook Air M1 into a private cloud storage solution for mobile devices, enabling automated photo and video backups from anywhere in the world.

## Proposed Changes

### 1. Remote Connectivity (Tailscale)
- **Action**: Direct the user to install Tailscale on mobile devices.
- **Goal**: Establish a secure, zero-config VPN tunnel between mobile and the Mac server.

### 2. Background Sync (SyncThing)
#### [NEW] [syncthing-service](file:///opt/homebrew/bin/syncthing)
- **Action**: Install and enable SyncThing via Homebrew.
- **Goal**: Provide a continuous, low-latency background sync engine that doesn't rely on cloud intermediaries.

### 3. Folder Infrastructure
- **Action**: Ensure `/Volumes/Cob-SSD/Media/Photos/Mobile-Backup` is prepared with proper permissions for the SyncThing service.

## Verification Plan

### Automated Tests
- **Connectivity**: Verify `ping 100.x.x.x` (Mobile IP) from the Mac server once Tailscale is active.
- **Service Status**: Verify `brew services list` shows syncthing as `started`.

### Manual Verification
- **Mobile Handshake**: Open the SyncThing Web UI (port 8384) and pair with a mobile device.
- **File Transfer**: Take a photo on the mobile device and verify its arrival in `/Volumes/Cob-SSD/Media/Photos/Mobile-Backup` within 60 seconds.
