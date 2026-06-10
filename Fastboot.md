# Android Fastboot Command Reference

A comprehensive, production-grade guide detailing the usage, syntax, and debugging workflows for the Android `fastboot` command-line utility. This document serves as both a reference manual and a troubleshooting guide for developers, system builders, and device technicians.

---

## Table of Contents
1. [Prerequisites & Environment Setup](#prerequisites--environment-setup)
2. [Command Syntax & Argument Rules](#command-syntax--argument-rules)
3. [Fastboot Commands: Complete Categorized Dictionary](#fastboot-commands-complete-categorized-dictionary)
    - [Device Discovery & State Querying](#device-discovery--state-querying)
    - [Flashing & Partition Management](#flashing--partition-management)
    - [Device Formatting & Erasing](#device-formatting--erasing)
    - [Booting & Active Slot Manipulation](#booting--active-slot-manipulation)
    - [OEM/Vendor Specialized Commands](#oemvendor-specialized-commands)
    - [Logical & Dynamic Partitions (User-Space FastbootD)](#logical--dynamic-partitions-user-space-fastbootd)
4. [Universal Debugging Workflows](#universal-debugging-workflows)
    - [Workflow A: "Device Not Found" (Fails to Detect)](#workflow-a-device-not-found-fails-to-detect)
    - [Workflow B: "Command Not Allowed" (Locked Bootloader)](#workflow-b-command-not-allowed-locked-bootloader)
    - [Workflow C: "Failed (remote: data too large)" or Limit Errors](#workflow-c-failed-remote-data-too-large-or-limit-errors)
    - [Workflow D: Rescue from Bootloop (A/B Active Slot Mismatch)](#workflow-d-rescue-from-bootloop-ab-active-slot-mismatch)
5. [Interactive Explorer Application](#interactive-explorer-application)

---

## Prerequisites & Environment Setup

Fastboot communication occurs at the bootloader layer when the system kernel is not loaded. To communicate with a device in Fastboot mode, you need the proper client tools and drivers.

### 1. Platform Tools Installation
Verify that your Android SDK Platform Tools are up to date. Outdated fastboot clients often fail when communicating with modern Google Tensor or Qualcomm Snapdragon devices utilizing dynamic partitions or super images.
- **macOS (via Homebrew):** `brew install android-platform-tools`
- **Linux (Debian/Ubuntu):** `sudo apt update && sudo apt install android-tools-fastboot android-tools-adb`
- **Windows:** Download official ZIP binaries directly from the Android Developer Portal, extract them, and add the folder path to system Environment Variables (`PATH`).

### 2. Device Driver Setup (Windows Specific)
Windows frequently failed to identify Android devices in bootloader state.
1. Download the **Google USB Driver**.
2. Boot your device into bootloader mode (`adb reboot bootloader`).
3. Open **Device Manager**, find the yellow exclamation mark indicating the unrecognized device (often labeled `Android` or `Kedacom USB Device`).
4. Right-click -> **Update Driver** -> **Browse my computer for drivers** -> point to the location of the extracted Google USB Driver -> choose **Android Bootloader Interface**.

---

## Command Syntax & Argument Rules

Fastboot syntax aligns with the following primary signature:

```bash
fastboot [options] <command> [arguments]
```

### Global Options Reference
*   `-s <serial>`: Route traffic to a specific connected device serial number (essential when handling multi-device environments).
*   `-w`: Wipe user data and cache (executes format on `/data` and `/cache` partitions).
*   `--slot <slot>`: Specify active slot override (`a`, `b`, or `all`) on active A/B-partitioned devices.
*   `--skip-secondary`: Bypass secondary slot flashing when carrying out structural partition writes.
*   `--disable-verity` / `--disable-verification`: Disable AVB (Android Verified Boot) signature validation. Used heavily when flashing custom recovery, custom kernels (`boot.img`), or modified systems.

---

## Fastboot Commands: Complete Categorized Dictionary

### Device Discovery & State Querying

#### `fastboot devices`
*   **Syntax:** `fastboot devices [-l]`
*   **Purpose:** Lists all connected devices currently registered under the bootloader or FastbootD protocol interfaces.
*   **Detailed Output:** Returns the unique device hardware serial number along with its state identifier (`fastboot` or `recovery`). Adding `-l` appends detailed USB hardware path strings.

#### `fastboot getvar`
*   **Syntax:** `fastboot getvar <variable_name>` or `fastboot getvar all`
*   **Purpose:** Fetches structural hardware, software, and partition state parameters from a target device bootloader.
*   **Crucial Variables to Query:**
    *   `fastboot getvar product`: Identifies target device board name (e.g., `sunfish`, `redfin`, `shiba`) to prevent flashing mismatched ROMs.
    *   `fastboot getvar current-slot`: Declares active boot/system slot (`a` or `b`).
    *   `fastboot getvar secure`: Indicates if bootloader is structurally locked (`yes` / `no`).
    *   `fastboot getvar max-download-size`: Declares physical RAM storage boundary for incoming split partition chunks.
    *   `fastboot getvar has-slot:system`: Checks if the system partition utilizes A/B layouts (returns `yes` or `no`).

---

### Flashing & Partition Management

#### `fastboot flash`
*   **Syntax:** `fastboot flash [--slot <slot>] <partition> <local_file_path>`
*   **Purpose:** Overwrites a destination raw partition with a specified binary matrix payload.
*   **Examples:**
    *   `fastboot flash boot boot.img`: Overwrites active slot kernel/ramdisk image.
    *   `fastboot flash recovery recovery.img`: Writes custom recovery environment (e.g., TWRP).
    *   `fastboot flash --slot all system system.img`: Writes a system image directly to all logical slots.

#### `fastboot flashall`
*   **Syntax:** `fastboot flashall`
*   **Purpose:** Automates full device flashing by tracking custom system builds.
*   **Mechanics:** Requires the env variable `ANDROID_PRODUCT_OUT` to point directly to a directory containing active binary compilation outputs (such as `boot.img`, `system.img`, `vendor.img`, `super.img`). Flashes all structures dynamically in correct order.

---

### Device Formatting & Erasing

#### `fastboot erase`
*   **Syntax:** `fastboot erase <partition>`
*   **Purpose:** Destroys data index nodes and filesystems residing on raw partition matrices. Performs low-level block cleansing.
*   **Warning:** Causes unrecoverable loss of target sectors. Usually executed preceding structural partition swaps.
*   **Example:** `fastboot erase userdata`

#### `fastboot format`
*   **Syntax:** `fastboot format[:ext4|f2fs] <partition>`
*   **Purpose:** Re-initializes empty, compliant filesystems over a targeted raw block mapping. Helps resolve "device corrupt" or encryption partition mismatch flags.
*   **Example:** `fastboot format:f2fs userdata`

---

### Booting & Active Slot Manipulation

#### `fastboot boot`
*   **Syntax:** `fastboot boot <local_kernel_image_path>`
*   **Purpose:** Downloads a temporary kernel or custom recovery image directly into volatile device RAM and triggers execution immediately *without* permanently writing to dry flash partitions.
*   **Use-Case:** Running dynamic data recovery tasks or testing custom builds without running hazards of bootloop.
*   **Example:** `fastboot boot live_recovery.img`

#### `fastboot set_active`
*   **Syntax:** `fastboot set_active <slot_letter>`
*   **Purpose:** Adjusts active physical boot sector pointing reference on modern dual A/B layout devices.
*   **Example:** `fastboot set_active a`

#### `fastboot reboot`
*   **Syntax:** `fastboot reboot [bootloader | recovery | fastboot]`
*   **Purpose:** Signals device control unit to power cycle the system.
*   **Destination Options:**
    *   `fastboot reboot`: Boots to primary user space Android UI.
    *   `fastboot reboot bootloader`: Soft resets back into current Bootloader mode.
    *   `fastboot reboot recovery`: Moves into diagnostic active user recovery mode.
    *   `fastboot reboot fastboot`: Boots system directly into userspace `fastbootd` container, exposing access to logical dynamic volumes.

---

### OEM/Vendor Specialized Commands

#### `fastboot oem` (Legacy / Hardware Specific)
*   **Syntax:** `fastboot oem <sub_command>`
*   **Purpose:** Execution conduit for device-vendor specific commands (Qualcomm, Samsung, Pixel).
*   **Common Sub-Commands:**
    *   `fastboot oem unlock`: Triggers system prompt to unlock bootloader structures (legacy).
    *   `fastboot oem lock`: Relocks the partition sectors to active factory rules.
    *   `fastboot oem device-info`: Displays general secure boot parameters.

#### `fastboot flashing` (Modern Android Standard)
*   **Syntax:** `fastboot flashing [unlock | lock | unlock_critical | lock_critical]`
*   **Purpose:** Modern standardized implementation governing boot security.
*   **Standard Modifiers:**
    *   `fastboot flashing unlock`: Grants sector modification rights to standard user partitions.
    *   `fastboot flashing unlock_critical`: Grants modification rights to critical bootloader, partition table, and primary SOC structures.

---

### Logical & Dynamic Partitions (User-Space FastbootD)

Modern Android devices group logical directories (such as `/system`, `/vendor`, `/product`, `/system_ext`) inside a single large physical block called the `super` partition. Manipulating these dynamic volumes requires entry into **FastbootD** (indicated by an aesthetic interface reading "FASTBOOTD" in red text on your device).

#### Dynamic System Updates (DSU) & Logical Commands:
*   `fastboot delete-logical-partition <partition_name>`: Destroys virtual mount nodes on the super system layer to free overall space blocks.
*   `fastboot create-logical-partition <partition_name> <size_bytes>`: Allocates static size boundary inside the dynamic volume array.
*   `fastboot resize-logical-partition <partition_name> <size_bytes>`: Dynamically adjusts borders when flashing modern custom system GSIs (Generic System Images).

---

## Universal Debugging Workflows

### Workflow A: "Device Not Found" (Fails to Detect)

If typing `fastboot devices` returns blank responses while your phone is plugged in with its bootloader screen visible:

```
                            [ Device In Bootloader ]
                                       │
                         Is ADB active or Fastboot?
                         (fastboot != ADB protocol)
                                       │
               ┌───────────────────────┴───────────────────────┐
         [ Windows OS ]                                  [ Linux / macOS ]
               │                                               │
    Check Device Manager                                Ensure libusb permissions
    Does yellow "Android" icon appear?                  Are udev rules configured?
               │                                               │
   ┌───────────┴───────────┐                         ┌─────────┴─────────┐
[ YES ]                  [ NO ]                   [ YES ]             [ NO ]
   │                       │                         │                   │
Install Google USB      Try different USB cable;  Try using sudo    Create rule in
Driver manually via     avoid USB-C to USB-C;     fastboot devices  /etc/udev/rules.d/
Device Manager.         prefer native USB-A.                          51-android.rules
```

1.  **Cable & Port Topology Audit:** Avoid USB 3.0/3.1 blue ports or native USB-C to USB-C paths on modern AMD Ryzen chipsets. They frequently refuse handshake synchronization during raw low-level transport. Use a USB 2.0 Hub or clean USB-C to USB-A cabling.
2.  **Udev Rules Verification (Linux Only):** Create `/etc/udev/rules.d/51-android.rules` and append:
    ```
    SUBSYSTEM=="usb", ATTR{idVendor}=="18d1", MODE="0666", GROUP="plugdev"
    ```
    Reload rules using `sudo udevadm control --reload-rules`.

---

### Workflow B: "Command Not Allowed" (Locked Bootloader)

If writes fail displaying: `FAILED (remote: 'Command not allowed')` or `FAILED (remote: 'partition table write protected')`:

1.  Boot the device normally into Android.
2.  Navigate to **Settings** -> **About Phone** -> Tap **Build Number** 7 times.
3.  Go to **Developer Options** -> Toggle **OEM Unlocking** to **ON**. (Requires network connectivity).
4.  Reboot back into Fastboot Mode: `adb reboot bootloader`.
5.  Execute:
    ```bash
    fastboot flashing unlock
    ```
6.  Use the physical Volume keys on your device to accept UI warning prompt, and confirm selection via Power button.

---

### Workflow C: "Failed (remote: data too large)" or Limit Errors

If image size exceeds bootloader buffer boundaries during copy operations, yielding `FAILED (remote: data too large)` or `FAILED (remote: 'Too many open files')`:

1.  Find the target partition's buffer boundary using:
    ```bash
    fastboot getvar max-download-size
    ```
2.  If manual flashing, enforce sparse splitting structures:
    ```bash
    # Enforces chunking limits automatically
    fastboot -S 256M flash system system.img
    ```
3.  If booting dynamic images, flash directly to the physical `super` container or run inside userspace `fastbootd` state:
    ```bash
    fastboot reboot fastboot
    fastboot flash system system.img
    ```

---

### Workflow D: Rescue from Bootloop (A/B Active Slot Mismatch)

When flashing custom system ROM or security updates triggers continuous reboot loop cycles:

1.  Check what slot you are active on:
    ```bash
    fastboot getvar current-slot
    ```
2.  Check if current partitions have successfully populated on both partitions or just one.
3.  Change pointing vector destination:
    ```bash
    # If currently visual slot is 'a', force toggle to 'b'
    fastboot set_active b
    fastboot reboot
    ```
4.  If the loop stops, your target kernel image on slot `a` was malformed or missing adequate headers. Swap back and re-flash partition sectors individually to slot `a`:
    ```bash
    fastboot flash boot_a boot.img
    fastboot flash system_a system.img
    ```

---

## Interactive Explorer Application

The React application running on this instance provides an interactive catalog of these commands and diagnostic wizards to assist you in learning Fastboot syntax and debugging hardware states in real-time.

To use the interactive tool:
1. Access the web sandbox environment containing this project.
2. Search based on commands (`getvar`, `flash`, `reboot`), select commands to view detailed parameter maps, or navigate the **Interactive Troubleshooting Wizard** to get tailored advice for your specific hardware symptoms.
