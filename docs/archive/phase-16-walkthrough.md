# Walkthrough: Phase 15 & 16 — Infrastructure & Cloud Activation

I have successfully completed the operational transition of the MacBook Air M1 server. The system is now healthy, monitored, and acts as a private cloud for mobile devices.

## Accomplishments

### 1. Operations & Maintenance (Phase 15)
- **Automated Pruning**: Weekly `docker system prune` scheduled via cron (Sundays at 3:00 AM) to maintain the 1TB SSD.
- **Health Monitoring**: Deployed `~/scripts/health-check.sh` on the Mac server.
- **Commander's Deck**: The Dashboard is live at `http://100.72.109.37:3010`.

### 2. Mobile Media Cloud (Phase 16)
- **Sync Engine**: Deployed **SyncThing** via Docker to bypass macOS Firewall restrictions.
- **Verified Connectivity**: GUI is live and reachable via Tailscale at `http://100.72.109.37:8384`.
- **Target Storage**: Prepared `/Volumes/Cob-SSD/Media/Photos/Mobile-Backup` and `/Volumes/Cob-SSD/Media/Documents` for incoming mobile syncs.
- **Premium Client**: Director has activated **Möbius Sync Pro**, unlocking unlimited background synchronization and file-system level access on iOS.

## Resolution of Connectivity Issue
> [!IMPORTANT]
> **Issue**: Native SyncThing was blocked by the macOS Application Firewall despite being configured to listen on all interfaces.
> **Fix**: Migrated SyncThing to Docker. Since Docker Desktop for Mac manages its own network bridge, it circumvents the application-level firewall blockade, allowing immediate access via Tailscale.

## 🧹 Safe Deletion & Future Syncing

### How to liberate space on your iPhone:
1.  **Mass Delete**: Open the **Photos** app → **Library** → **All Photos** → **Select** → **Select All** → **Trash**.
2.  **Empty Trash**: Go to **Albums** → **Recently Deleted** → **Delete All**. 
3.  **Safety**: Since your Mac server is in **"Receive Only"** mode, it will keep all these photos even after you delete them from the phone.

### Future Sync Behavior (The "iOS Rule"):
- **Background Sync**: iOS is strict. Möbius Sync will try to sync in the background via "Background App Refresh," but it's not always instant.
- **Pro Tip**: After taking a lot of new photos, simply **open the Möbius app for 10 seconds**. This "wakes up" the engine and ensures your new memories are immediately vaulted to the SSD.
- **Charging**: Syncing is most reliable when the phone is on a charger and the app is open (perfect for overnight backups).

The system is now your invisible safety net. Enjoy the extra 18GB of space!
