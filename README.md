# AetherFS 🌿
### Native, User-Space EXT4 for macOS.

![Swift](https://img.shields.io/badge/Swift-6.0-orange.svg) ![Platform](https://img.shields.io/badge/platform-macOS%2015+-lightgrey.svg) ![License](https://img.shields.io/badge/license-GPLv2-blue.svg) ![Status](https://img.shields.io/badge/status-EXPERIMENTAL-red.svg)

**AetherFS** is a modern, open-source storage bridge that allows macOS to read and write Linux **EXT4** drives at native speeds.

It is built on Apple's new **FSKit** framework (introduced in macOS Sequoia), running entirely in user space. This means:
* 🚫 **No Kernel Extensions (`kexts`):** Your system stays stable.
* 🚫 **No FUSE:** No reliance on legacy `macFUSE` or `osxfuse` layers.
* 🚫 **No "Reduced Security":** Runs without compromising Apple Silicon security policies.
* ⚡ **Native Performance:** Uses Apple's optimized VFS bridge.

> **⚠️ WARNING: PRE-ALPHA SOFTWARE**
> This project is currently in active development. **Do not use this on critical data without backups.** While Read operations are generally safe, Write support is experimental.

---

## 🧐 Why AetherFS?

For years, macOS users have been forced to choose between:
1.  **Expensive Proprietary Drivers:** Paying ~$40/version for closed-source drivers (e.g., Paragon) that require invasive system permissions.
2.  **Slow/Broken FOSS:** Using FUSE-based solutions that are often read-only, slow, or broken on newer macOS releases.

**AetherFS** solves this by wrapping the battle-tested embedded C library **[lwext4](https://github.com/gkostka/lwext4)** inside a modern Swift **FSKit** module. It is a "Solarpunk" solution: sustainable, open, and community-owned.

---

## 🏗 Architecture

AetherFS functions as a **System Extension**. When you plug in a drive, macOS identifies the EXT4 UUID and hands control to the AetherFS module.

1.  **FSKit Interface (Swift):** Handles the communication with the macOS Kernel (VFS).
2.  **Bridging Header:** Maps FSKit calls (`lookup`, `read`, `write`) to C functions.
3.  **Core Logic (C):** `lwext4` performs the raw block operations on the USB device.

---

## 🚀 Getting Started

### Prerequisites
* **macOS 15.0 (Sequoia)** or later.
* **Xcode 16+**.
* **Apple Developer Account** (Required for signing System Extensions).
* *Hardware:* Works on Apple Silicon (M1/M2/M3) and Intel.

### Installation (Development Mode)

Because FSKit is a restricted entitlement, you cannot simply "run" this without provisioning. To test it locally, you must temporarily disable SIP for filesystem debugging.

1.  **Disable SIP (Filesystem Debugging Only):**
    * Boot into Recovery Mode.
    * Run: `csrutil disable --without kext` (or full disable if that fails).
    * Reboot.

2.  **Clone & Build:**
    ```bash
    git clone [https://github.com/madfam/AetherFS.git](https://github.com/madfam/AetherFS.git)
    cd AetherFS
    xcodebuild -scheme AetherFS -configuration Debug
    ```

3.  **Install the Extension:**
    Drag the built `AetherFS.app` to your `/Applications` folder and launch it once to register the extension.

4.  **Mount a Drive:**
    Plug in your 4TB EXT4 USB drive. macOS should prompt you to allow "AetherFS" to manage the volume.

---

## 🗺 Roadmap

- [x] **Phase 1: The Skeleton**
    - [x] FSKit Module scaffolding.
    - [x] Detection of EXT4 Partition UUIDs.
- [ ] **Phase 2: The Reader (Current Focus)**
    - [ ] reliable `ls` (directory listing).
    - [ ] reliable `cp` (read-only file copy).
- [ ] **Phase 3: The Writer**
    - [ ] Journal replay (crash recovery).
    - [ ] Write/Delete support.
- [ ] **Phase 4: The Release**
    - [ ] Obtain `com.apple.developer.fskit.fsmodule` entitlement from Apple.
    - [ ] Public binary release (No SIP disable required).

---

## 🤝 Contributing

We are looking for **Systems Engineers** and **Swift/C Developers**.

**Key Areas of Need:**
* **FSKit Interop:** Mapping obscure VFS flags to `lwext4`.
* **Performance:** Optimizing block buffer sizes for 4K video playback.
* **Safety:** Implementing strict "Fail-Safe" logic to prevent corruption.

**To Contribute:**
1.  Fork the repo.
2.  Create a feature branch (`git checkout -b feature/journaling`).
3.  Submit a PR.

---

## 📜 License

**GPLv2** (Inherited from `lwext4`).
This ensures AetherFS remains free and open forever.

---

<div align="center">
  <sub>Built with 🖤 by <b>Innovaciones MADFAM</b>. <br>
  <i>"The future is built, not bought."</i></sub>
</div>
