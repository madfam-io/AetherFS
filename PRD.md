# Product Requirements Document: Project MAD-LINK

**Version:** 0.1.0 (Draft)
**Status:** In Development
**Owner:** Innovaciones MADFAM (CEO)
**Codename:** *Project MAD-LINK* (Phase 1), *Project AetherFS* (Phase 2)
**Target Platform:** macOS Sequoia (15.x) on Intel Core i7 & Apple Silicon

-----

## 1\. Executive Summary

**Project MAD-LINK** is a high-performance, open-source storage bridge designed to enable native-speed Read/Write access to Linux filesystems (EXT4) on macOS.

Current market solutions are antithetical to the Solarpunk ethos:

  * **Paragon extFS:** Proprietary, expensive (\~$40), and historically invasive (kernel extensions that compromise system security).
  * **macFUSE/ext4fuse:** Slow, unstable, or read-only due to "User Space" limitations and lack of modern Journaling support.

**Our Vision:** A two-phase strategy to build the "Standard" for open storage on macOS.

1.  **Phase 1 (The MVP):** A "Unikernel Bridge" using hardware virtualization to bypass macOS driver limitations entirely.
2.  **Phase 2 (The Product):** A native `FSKit` module that utilizes Apple's new user-space filesystem framework to compete directly with Paragon on ease of use.

-----

## 2\. Problem Statement

The User (CEO of MADFAM) requires 4TB of media transfer capability between macOS and a Linux-based Jellyfin server.

  * **The Bottleneck:** macOS cannot write to EXT4.
  * **The Constraint:** The User refuses to compromise on security (disabling SIP for untrusted kexts) or ethics (proprietary lock-in).
  * **The Opportunity:** Apple’s recent deprecation of Kernel Extensions (kexts) in favor of `FSKit` has leveled the playing field. Paragon is struggling to adapt; this is the moment to strike.

-----

## 3\. Phase 1: The MVP (Project MAD-LINK)

**Goal:** Achieve "Paragon-like" speeds using "Hacker" methods on existing Intel hardware.
**Concept:** Instead of writing a filesystem driver (hard), we run a microscopic Linux kernel that *is* the driver (smart).

### 3.1 Technical Architecture

We will utilize **QEMU** with the **HVF (Hypervisor Framework)** accelerator to run a headless Alpine Linux VM. This VM will mount the physical USB drive and re-export it to macOS via NFS v4 (Network File System).

  * **Host (macOS):** Intel Core i7 (3.2 GHz).
  * **Hypervisor:** QEMU `x86_64` with `-accel hvf` (Near-native CPU performance).
  * **Guest (The Bridge):** Alpine Linux (50MB RAM footprint).
  * **Transport Layer:** NFS over `localhost` (Gigabit+ speeds, zero encryption overhead).

### 3.2 Functional Requirements

  * **FR-01 (Raw Access):** The system must unmount the USB drive from macOS Disk Utility to allow the VM exclusive access.
  * **FR-02 (Auto-Mount):** A single CLI command (`madlink mount`) must boot the VM and mount the drive in `/Volumes/MAD-LINK`.
  * **FR-03 (Performance):** Write speeds must exceed 100MB/s (saturating USB 3.0, not CPU).
  * **FR-04 (Integrity):** The EXT4 Journal must be active to prevent corruption during power loss.

### 3.3 The "Code" (Shell Script Prototype)

This script is the MVP deliverable. It replaces the $40 Paragon license.

```bash
#!/bin/bash
# madlink.sh - The MADFAM EXT4 Bridge
# Dependencies: qemu, diskutil

DISK_ID=$1 # e.g., /dev/disk4
VM_MEM="512M"
MOUNT_POINT="/System/Volumes/Data/MAD_LINK"

echo "🌿 MAD-LINK: Initializing Solarpunk Storage Bridge..."

# 1. Safety Check: Unmount from macOS
if mount | grep -q "$DISK_ID"; then
    echo "🔌 Unmounting $DISK_ID from macOS..."
    diskutil unmountDisk $DISK_ID
fi

# 2. Fix Permissions (Granting QEMU raw disk access)
# Note: On macOS 15, you may need to run this as sudo or grant Full Disk Access to Terminal
sudo chown $(whoami) $DISK_ID

# 3. Boot the Bridge (Headless)
# -virtfs is used here for simpler host-guest sharing if NFS feels too heavy
# But for MVP reliability, we use raw TCP forward for NFS
echo "🚀 Booting Micro-Kernel..."
qemu-system-x86_64 \
  -m $VM_MEM \
  -accel hvf \
  -cpu host \
  -smp 2 \
  -drive file=$DISK_ID,format=raw,if=virtio \
  -netdev user,id=n1,hostfwd=tcp::2049-:2049 \
  -device virtio-net-pci,netdev=n1 \
  -nographic \
  -kernel ./alpine-virt \
  -initrd ./initramfs-lts \
  -append "root=/dev/ram0 console=ttyS0 quiet" &

PID=$!
echo "✅ Bridge Active (PID: $PID). Waiting for NFS..."
sleep 10

# 4. Mount to Finder
sudo mount -t nfs -o vers=4,tcp,port=2049 localhost:/mnt/usb $MOUNT_POINT
open $MOUNT_POINT
```

-----

## 4\. Phase 2: The Competitor (Project AetherFS)

**Goal:** A native, drag-and-drop macOS app that replaces Paragon entirely.
**Concept:** A user-space `FSKit` module using the `lwext4` library.

### 4.1 The Strategic Advantage (The "Impossible" Path)

Paragon uses legacy kexts or complex System Extensions that require reducing macOS security settings (Reduced Security Mode).
**Project AetherFS** will use **FSKit** (introduced in macOS 14/15), which runs entirely in user-space.

  * **User Benefit:** No reboot required. No "Reduced Security" mode. No kernel panics.
  * **Developer Hurdle:** Requires the `com.apple.developer.fskit.fsmodule` entitlement.

### 4.2 Architecture

  * **Core Logic:** `lwext4` (Lightweight EXT4 C Library) - Battle-tested in embedded systems.
  * **Binding Layer:** Swift / Objective-C wrapper implementing `FSFileSystem` protocol.
  * **UI:** A minimal Menu Bar app ("Aether Tray") to manage mounts.

### 4.3 Development Roadmap

| Phase | Milestone | Deliverable | Est. Time |
| :--- | :--- | :--- | :--- |
| **Alpha** | The "Read-Only" Proof | Compile `lwext4` against FSKit headers. Verify mounting a 4TB drive Read-Only. | 2 Weeks |
| **Beta** | The "Write" Unlock | Implement `write`, `mkdir` ops mapped to `lwext4`. **High Risk of Corruption here.** | 1 Month |
| **Release** | The Entitlement | Apply to Apple via MADFAM corporate account for FSKit entitlement. | 1-3 Months |

-----

## 5\. Competitive Analysis

| Feature | Paragon extFS | macFUSE | **MAD-LINK (MVP)** | **AetherFS (Final)** |
| :--- | :--- | :--- | :--- | :--- |
| **Cost** | \~$40 USD | Free (Closed Source) | **Free (FOSS)** | **Free (FOSS)** |
| **Speed** | Native (High) | Medium/Low | **Near-Native** | **Native** |
| **Security** | **Low** (Kernel Access) | **Low** (Kext Required) | **High** (Sandboxed VM) | **High** (FSKit Sandboxed) |
| **Ease of Use** | Plug & Play | High Friction (CMD Line) | Medium (Script) | Plug & Play |
| **Architecture** | Legacy Kext | Legacy Kext | **Virtualization** | **Modern FSKit** |

**Would you like me to refine the Shell Script above to be "Copy-Paste" ready with the exact networking flags for your specific Mac Mini?**
