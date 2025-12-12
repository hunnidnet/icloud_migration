# Project Sovereign Cloud: iCloud to Local Infrastructure Migration

**Version:** 1.0 (Tahoe Era / Nov 2025)  
**Status:** Active Migration  
**Objective:** Migrate 20+ family iOS devices from 2TB iCloud subscription to self-hosted infrastructure (Mac Mini + 10TB Pegasus RAID) with zero data loss and high WAF (Wife Acceptance Factor).

---

## 📋 Executive Summary
This project transitions the family digital life from a "Rented" model (iCloud) to an "Owned" model. 

* **Primary Storage:** Mac Mini Server with 10TB Pegasus R4 RAID.
* **Photo Management:** [Immich](https://immich.app/) (Self-Hosted Google Photos alternative).
* **Device Backups:** [iMazing 3.4](https://imazing.com/) (Local snapshots).
* **Message Archival:** `imessage-exporter` (Vendor-neutral HTML archives).

**The Core Strategy:** Use a **"Parallel Trial"** to validate the new system while keeping iCloud active as a safety net, followed by a **"Space Swap"** to move storage priority from Photos (cloud) to Messages (local).

---

## 🏗 Architecture

```mermaid
graph TD
    subgraph Family["Family Devices (iOS 26)"]
        iPhone["iPhone 13 (Wife)"]
        iPad["iPad (Mom/Dad)"]
    end

    subgraph Server["Mac Mini Server (macOS Tahoe)"]
        PhoneApp["Mac Phone App (Call Logs)"]
        iMazing["iMazing 3.4 (Backups)"]
        Immich["Immich Server (Docker)"]
        MsgExport["imessage-exporter (CLI)"]
    end

    subgraph Storage["Pegasus RAID (10TB)"]
        PhotoArch["Original iCloud Photo Archive"]
        iMazingDB["Device Snapshots"]
        ImmichDB["Immich Database"]
    end

    iPhone -->|Auto-Upload| Immich
    iPhone -->|Daily WiFi Backup| iMazing
    iPhone -->|Syncs Calls| PhoneApp
    
    Immich --> Storage
    iMazing --> Storage
    MsgExport --> Storage
````

-----

## 🛠 Prerequisites

  * **Hardware:** Mac Mini Server, 10TB External RAID (APFS format recommended).
  * **Software:** \* iMazing 3.4+ (Compatible with macOS Tahoe).
      * Docker Desktop (Running Immich).
      * `osxphotos` (Python tool for metadata extraction).
      * `imessage-exporter` (Rust tool for chat archival).
  * **Access:** Admin access to the Mac; Passcodes for all family iPhones.

-----

## 🚀 Phase 1: The "Great Download" (Risk: Low)

*Goal: Create a master local copy of all iCloud data without modifying the phones.*

### 1.1 Multi-User Setup on Mac

1.  Create separate macOS User Accounts for each family member (e.g., `User_Wife`, `User_Mom`).
2.  Log in to `User_Wife` -\> **System Settings** -\> **iCloud** -\> Sign In.

### 1.2 The Photo Download (The Long Pole)

1.  Open **Photos.app**.
2.  **Settings** -\> **iCloud** -\> Check **"Download Originals to this Mac"**.
3.  **Wait:** Allow the Mac to download the full 300GB+ library to the External Drive.
      * *Tip:* Ensure the library is stored on the Pegasus drive by holding `Option` when launching Photos and creating the library on the external volume.

### 1.3 Immich Import

Once the download is 100% complete (Status bar in Photos says "Updated Just Now"):

1.  **Extract:** Use `osxphotos` to pull images + metadata out of the Apple library.
    ```bash
    osxphotos export /Volumes/Pegasus/Staging/Wife --db /Volumes/Pegasus/Libraries/Wife.photoslibrary --sidecar xmp
    ```
2.  **Import:** Push to her specific Immich account.
    ```bash
    immich-go upload --server=http://localhost:2283 --key=[HER_API_KEY] /Volumes/Pegasus/Staging/Wife
    ```

-----

## 🧪 Phase 2: The Parallel Trial (Risk: Low)

*Goal: Family tests the "New App" (Immich) while "Old App" (Photos/iCloud) stays active.*

1.  **Install Immich App** on all family phones.
2.  **Enable Background Backup** in Immich settings.
3.  **Strict Rule:** Instruct family **NOT** to use the "Free Up Space" button in Immich yet.
4.  **Duration:** Run for 2-4 weeks.
5.  **Success Criteria:**
      * [ ] Photos appear in Immich web view within minutes of being taken.
      * [ ] Wife approves of the UI/Sharing features.
      * [ ] No battery drain complaints.

-----

## 🔄 Phase 3: The "Space Swap" (Risk: High)

*Goal: Delete Photos from phones to make room for full Message history & Local Backups.*

### 🛑 STOP & VERIFY

  * **Verification:** Check three random photos from 5 years ago in Immich. Do they load full resolution?
  * **Backup:** Ensure the Mac Mini itself (and the 10TB drive) is backed up (e.g., Backblaze or Offline Disk).

### 3.1 Evict Photos

1.  On the iPhone, open **Immich**.
2.  Press **"Free Up Space"**.
3.  *Result:* Immich deletes local copies of photos that are safely on the server.
4.  *iCloud Reaction:* iCloud will also delete these photos from the cloud (since it mirrors the phone). **This is intended.**

### 3.2 Repatriate Messages

Now that the phone has \~100GB free:

1.  **Settings** -\> **[Name]** -\> **iCloud** -\> **Show All**.
2.  **Messages** -\> **Turn Off**.
3.  Select **"Disable and Download Messages"**.
4.  *Wait:* The phone will download the full 50GB attachment history from Apple's servers.

### 3.3 Consolidate Health & Files (Tahoe Fix)

1.  Open **Health Auto Export** app.
2.  Change export target from "iCloud Drive" to **"On My iPhone"**.
3.  Ensure WhatsApp "Auto Backup" is **OFF**.

### 3.4 The Master Backup

1.  Connect iPhone to Mac Mini via USB (first run).
2.  Run **iMazing**.
3.  Perform a full **Encrypted Backup** to the Pegasus Drive.
4.  *Verify:* Check that the backup size is now \~60GB+ (Messages + Health + WhatsApp), not the old 3GB.

-----

## ↩️ Rollback Plan (Plan B)

*If the family rejects the new system during the Trial Phase:*

1.  **Delete Immich App:** Simply remove the app from their phones.
2.  **Status Quo:** Since we never disabled iCloud Photos during Phase 2, their phones are unchanged.
3.  **Cleanup:** Delete the Docker container and the Immich data folder on the RAID.
      * *Cost:* $0.
      * *Data Loss:* None.

*If they want to revert AFTER Phase 3 (The Space Swap):*

1.  **Re-subscribe** to iCloud 2TB.
2.  **Turn On** iCloud Photos on the phone.
3.  **Upload:** The phone will seemingly be empty of photos. You must use the macOS Photos app (where the originals still live) to "Sync" them back up to iCloud.
      * *Note:* This will take days to re-upload 300GB, but the data is safe on your RAID.

-----

## 📅 Maintenance Schedule

| Frequency | Task | Tool |
| :--- | :--- | :--- |
| **Daily** | iOS Device Snapshots (Wi-Fi) | iMazing Scheduler |
| **Weekly** | Immich Database Dump (SQL) | Cron Job |
| **Monthly** | Archive iMessage History (HTML) | `imessage-exporter` (on Mac) |
| **Monthly** | Verify RAID Health | Pegasus Utility |

## 📝 Notes on macOS Tahoe (Nov 2025)

  * **Call Logs:** Do not try to extract these from iMazing. Let them sync to the **macOS Phone App** and back up the Mac's `~/Library/Application Support/CallHistoryDB`.
  * **Journal App:** Similarly, ensure the Mac Journal app is open occasionally to sync entries for local backup.
