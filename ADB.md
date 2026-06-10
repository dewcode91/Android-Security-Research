# Android Debug Bridge (ADB) Command Reference Guide

An exhaustive reference manual and troubleshooting guide for **Android Debug Bridge (ADB)**. This document provides technical breakdowns of ADB commands, syntax structures, category filters, and typical Android developer debugging workflows.

---

## Table of Contents
1. [ADB Architecture Overview](#1-adb-architecture-overview)
2. [Global Command Syntax](#2-global-command-syntax)
3. [ADB Command Reference by Category](#3-adb-command-reference-by-category)
   - [Basic & Connection Management](#basic--connection-management)
   - [File Operations](#file-operations)
   - [App / Package Management (`pm`)](#app--package-management-pm)
   - [Activity Manager (`am`) & System Controls](#activity-manager-am--system-controls)
   - [Logging, System Info & Diagnostics](#logging-system-info--diagnostics)
   - [Hardware & Input Simulation](#hardware-input-simulation)
   - [Reboot & Recovery Options](#reboot--recovery-options)
4. [Common Developer Workflows](#4-common-developer-workflows)
   - [Workflow A: Connecting via Wireless Debugging (ADB over Wi-Fi)](#workflow-a-connecting-via-wireless-debugging-adb-over-wi-fi)
   - [Workflow B: Recording and Extracting a Demonstration Video](#workflow-b-recording-and-extracting-a-demonstration-video)
   - [Workflow C: Troubleshooting an "Unauthorized" Device](#workflow-c-troubleshooting-an-unauthorized-device)
   - [Workflow D: Simulating Broadcast Events & Intents](#workflow-d-simulating-broadcast-events--intents)
   - [Workflow E: Profile and Performance Audits](#workflow-e-profile-and-performance-audits)
5. [ADB Command Cheat Sheet Summary](#5-adb-command-cheat-sheet-summary)

---

## 1. ADB Architecture Overview

Android Debug Bridge is a versatile command-line tool that lets you communicate with a device. The ADB tool facilitates a variety of device actions, such as installing and debugging apps. It operates as a client-server program that includes three key components:

1. **Client**: The entity that sends commands. The client runs on your development machine. You can invoke a client from a command-line terminal by issuing an `adb` command.
2. **Daemon (adbd)**: The program that runs commands on the target device. The daemon runs as a background process on each Android device or emulator instance.
3. **Server**: The coordinator that manages communication between the client and the daemon. The server runs as a background process on your development machine to multiplex commands across active devices.

```
 [ Development PC ]                         [ Android Device ]
+-------------------+                      +-------------------+
|  ADB Client       |                      |                   |
|  (Terminal, IDE)  |                      |                   |
|        |          |                      |                   |
|  (TCP 5037)       |                      |                   |
|        v          |                      |                   |
|  ADB Server       |<==== USB / Wi-Fi ===>|  ADB Daemon       |
|  (Background Port)|                      |  (adbd background)|
+-------------------+                      +-------------------+
```

---

## 2. Global Command Syntax

The general format for executing ADB commands is:

```bash
adb [-d | -e | -s <serial_number>] <command>
```

### Global Flags
* `-d`: Direct the command to the only connected physical USB device. Returns an error if multiple physical devices are connected.
* `-e`: Direct the command to the only running emulator instance. Returns an error if multiple emulators are running.
* `-s <serial_number>`: Target a specific device or emulator by its unique serial number (retrieved using `adb devices`). **Highly recommended when working with multiple connected targets.**

---

## 3. ADB Command Reference by Category

### Basic & Connection Management

These commands govern the ADB server lifecycle, device discovery, and network connections.

| Command | Syntax | Explanation |
| :--- | :--- | :--- |
| **List Devices** | `adb devices` | Lists all emulator/device instances currently recognized by the host. |
| **Verbose List** | `adb devices -l` | Lists devices with added details such as product name, model, device type, and transport ID. |
| **Start Server** | `adb start-server` | Spawns the background ADB server on the host machine (if not already running). |
| **Kill Server** | `adb kill-server` | Terminates the background ADB server. Use this to reset connections when ADB is unresponsive. |
| **Connect (IP)** | `adb connect <host>[:<port>]` | Connects over TCP/IP to a device running on the specified host IP and port (default is 5555). |
| **Disconnect IP** | `adb disconnect [<host>[:<port>]]` | Disconnects from the specified TCP/IP device. Leave blank to disconnect from all. |
| **Wireless Pair** | `adb pair <host>:<port> [<pairing_code>]` | Pairs a target device for wireless debugging using Android 11+ security standards. |
| **Restart via USB** | `adb usb` | Directs the ADB daemon to listen for connections on the physical USB interface. |
| **Restart via TCP/IP** | `adb tcpip <port>` | Instructs the ADB daemon to restart listening for connections on the specified port over TCP/IP (usually port `5555`). |
| **Get State** | `adb get-state` | Prints the current state of the device: `offline`, `bootloader`, `device`, or `recovery`. |

---

### File Operations

Commands used to transfer files between the development host computer and the Android device file system.

#### `adb push`
Copies files or directories from your development computer to the device.
* **Syntax**: `adb [-s <serial>] push <local_path> <remote_path>`
* **Example**:
  ```bash
  adb push ~/Desktop/sample.mp4 /sdcard/Movies/
  ```

#### `adb pull`
Copies files or directories from the device back to your development computer.
* **Syntax**: `adb [-s <serial>] pull <remote_path> [<local_path>]`
* **Example**:
  ```bash
  adb pull /sdcard/DCIM/Camera/IMG_20260610.jpg ~/Downloads/
  ```

#### Common Shell File Manipulation APIs
You can invoke Linux container commands on the device prefixing them with `adb shell`:
* **List Directory**: `adb shell ls -la /sdcard/`
* **Create Folder**: `adb shell mkdir -p /sdcard/AppData/`
* **Delete File**: `adb shell rm /sdcard/temp.txt`
* **Delete Folder**: `adb shell rm -rf /sdcard/old_folder/`
* **View File**: `adb shell cat /proc/cpuinfo`

---

### App / Package Management (`pm`)

The Package Manager (`pm`) allows you to interact with the package database on Android, retrieve package names, install new applications, and clear stored application data.

#### `adb install`
Installs an Android application package (`.apk`) to the device.
* **Syntax**: `adb install [options] <path_to_apk>`
* **Key Options**:
  * `-r`: Reinstall an existing app, keeping its data (updating state).
  * `-g`: Grant all permissions listed in the app manifest automatically upon installation.
  * `-t`: Allow installation of test APKs.
  * `-d`: Allow version code downgrade.
* **Example**:
  ```bash
  adb install -r -g target_build.apk
  ```

#### `adb uninstall`
Removes an application package from the system.
* **Syntax**: `adb uninstall [options] <package_name>`
* **Options**:
  * `-k`: Keep the cache and data directories of the application after removal.
* **Example**:
  ```bash
  adb uninstall com.example.mydebugapp
  ```

#### Troubleshooting and List Utilities
* **List Installed Packages**:
  ```bash
  # Lists all installed application packages
  adb shell pm list packages
  
  # Lists only third-party (non-system) developer packages
  adb shell pm list packages -3
  
  # Search for a package with a specific keyword matching a filter
  adb shell pm list packages | grep "google"
  ```
* **Clear App Data (Reset App)**:
  Resets all user-specific data, cache, databases, and permissions for the app. Perfect for simulating a fresh installation.
  ```bash
  adb shell pm clear com.example.mydebugapp
  ```
* **Get App Source Path**:
  Reveals where the physical `.apk` file for a given package is installed on the underlying secure filesystem.
  ```bash
  adb shell pm path com.example.mydebugapp
  ```

---

### Activity Manager (`am`) & System Controls

The Activity Manager (`am`) is one of ADB's most powerful shells. It allows you to simulate user behavior, launch applications, send broadcast intents, trigger background services, and alter display density parameters.

#### Force Stop an App
Kills all processes associated with a package instantly.
```bash
adb shell am force-stop com.example.mydebugapp
```

#### Kill Background Processes
Safely shuts down only background processes that are safe to terminate.
```bash
adb shell am kill com.example.mydebugapp
```

#### Launch an Activity
Starts a specific Activity on your Android UI. Optionally can pass intents and data.
* **Syntax**: `adb shell am start -n <package_name>/<activity_path>`
* **Options**:
  * `-a <action>`: Specify intent action (e.g., `android.intent.action.VIEW`).
  * `-d <data_uri>`: Pass data URI (e.g., a deep link URL).
  * `--es "<key>" "<value>"`: Pass extra string data.
  * `--ei "<key>" <value>`: Pass extra integer data.
* **Example**:
  ```bash
  # Launch Main Launcher Activity
  adb shell am start -n com.example.mydebugapp/com.example.mydebugapp.MainActivity
  
  # Trigger a custom deep link action
  adb shell am start -a android.intent.action.VIEW -d "myapp://profile?user=123"
  ```

#### Run Background Services
* **Start a Service**:
  ```bash
  adb shell am startservice -n com.example.mydebugapp/.MyBackgroundService
  ```
* **Send an Intent Broadcast**:
  Broadcast an event to registered Broadcast Receivers (useful for testing system triggers, deep-linking, offline actions, or testing custom events).
  ```bash
  adb shell am broadcast -a com.example.mydebugapp.ACTION_UPDATE_DATA --es "payload" "refresh_all"
  ```

---

### Logging, System Info & Diagnostics

These interfaces help developers tap into system telemetry, kernel pipes, event logs, and status dashboards.

#### Logcat (The Logging Core)
Logcat captures continuous traces of stack outputs, crash telemetry, application logs, and system error events.

* **Stream Real-time Logs**:
  ```bash
  adb logcat
  ```
* **Filter Logs by Tag & Priority Level**:
  Levels: `V` (Verbose), `D` (Debug), `I` (Info), `W` (Warning), `E` (Error), `F` (Fatal), `S` (Silent).
  ```bash
  # Format: <tag>:<severity> *:S (Suppress all other tags)
  adb logcat MyComponentTag:D *:S
  ```
* **Search / Filter using Grep**:
  ```bash
  # Search logs matching keyword case-insensitively
  adb logcat | grep -i "exception"
  ```
* **Clear Logcat Buffer**:
  Always clear log filters before starting a fresh debug run to remove historic clutter.
  ```bash
  adb logcat -c
  ```
* **Write Logs to a Local Text File**:
  ```bash
  adb logcat -d > local_debug_dump.txt
  ```

#### Dumpsys (System State Diagnostic Tool)
Provides deep inspection of all Android active services.
* **Basic Dumpsys**: `adb shell dumpsys` (dumps absolute system telemetry - extremely large!)
* **Audit Device Battery Metrics**:
  ```bash
  adb shell dumpsys battery
  ```
* **Inspect Active Application Window/Layout Stack**:
  ```bash
  adb shell dumpsys window | grep -E 'mCurrentFocus|mFocusedApp'
  ```
* **Inspect Memory Allocation Profile of an App**:
  ```bash
  adb shell dumpsys meminfo com.example.mydebugapp
  ```
* **Detailed Graphics Profiling / Frame Latencies**:
  ```bash
  adb shell dumpsys gfxinfo com.example.mydebugapp
  ```

#### Bugreport
Generates a massive, structured archive of system configuration, error logs, and dumpsys traces. Highly requested by hardware teams for post-crash analysis.
* **Syntax**: `adb bugreport [local_archive_destination]`
* **Example**:
  ```bash
  adb bugreport ~/Desktop/bugreports/
  ```

---

### Hardware & Input Simulation

Allows you to simulate hardware button presses, touch taps, keyboard inputs, swipe gestures, screen adjustments, and sensor parameters.

#### Simulated Keyboard and Text Entry
* **Inject Text Typing**:
  Simulates physical key typing on focus fields (spaces are represented as `%s` or wrapped in nested quotes).
  ```bash
  adb shell input text "Debugging%sADB%sInterface"
  ```
* **Simulate Hardware Button Actions (`keyevent`)**:
  | Action / Target Key | ADB Shell Input Command |
  | :--- | :--- |
  | **Home Button** | `adb shell input keyevent 3` |
  | **Back Button** | `adb shell input keyevent 4` |
  | **Power Button** | `adb shell input keyevent 26` |
  | **Volume Up** | `adb shell input keyevent 24` |
  | **Volume Down** | `adb shell input keyevent 25` |
  | **App Switch / Recents** | `adb shell input keyevent 187` |
  | **Enter Key** | `adb shell input keyevent 66` |
  | **Tab Key**| `adb shell input keyevent 61` |
  | **Wakeup Screen** | `adb shell input keyevent KEYCODE_WAKEUP` |

* **Simulate Screen Interactions**:
  * **Single Tap** (at $X, Y$ coordinate coordinates):
    ```bash
    # adb shell input tap <X> <Y>
    adb shell input tap 450 1200
    ```
  * **Swipe Gesture / Drag-and-Drop** (from $X_1, Y_1$ to $X_2, Y_2$ over a set duration of milliseconds):
    ```bash
    # adb shell input swipe <X1> <Y1> <X2> <Y2> [duration_ms]
    adb shell input swipe 100 1500 900 1500 500
    ```

---

### Reboot & Recovery Options

Safely direct device startup processes to load specific device target contexts.

* **Standard Reboot**:
  ```bash
  adb reboot
  ```
* **Reboot into Bootloader (Fastboot Mode)**:
  Restarts the target device directly into its hardware bootloader menu. Used for flashing custom partitions, kernels, kernels, or unlocking the hardware state.
  ```bash
  adb reboot bootloader
  ```
* **Reboot into Stock Recovery Mode**:
  Loads the partition responsible for factory wipes, partition cache updates, and manual zip applications.
  ```bash
  adb reboot recovery
  ```
* **Reboot directly to Sideload State**:
  Used to sideload custom manufacturer updates over USB in stock operations.
  ```bash
  adb reboot sideload
  ```

---

## 4. Common Developer Workflows

### Workflow A: Connecting via Wireless Debugging (ADB over Wi-Fi)

For Android 11 and above, Google provides a secure pairing mechanism using random PIN codes directly on the local network.

#### Step 1: Connect host and mobile device to the SAME Wi-Fi network.
#### Step 2: Open Developer Options on Device
1. Scroll down to **Wireless Debugging** and toggle it **ON**.
2. Tap **Wireless Debugging** to enter its settings pane.
3. Tap **Pair device with pairing code**. Note the *IP Address*, *Port*, and *Pairing Code*.

#### Step 3: Run the Pair Command
Run the command on your host computer:
```bash
adb pair <IP_address>:<Pairing_Port>
# Prompt asks for the PIN. Enter the numeric code and hit Enter.
```

#### Step 4: Run the Connect Command
Find the main wireless IP address and port on the initial page (different from the pairing port) and type:
```bash
adb connect <IP_address>:<Connection_Port>
```

#### Step 5: Verify Connection
```bash
adb devices
# Output should list the IP address as a device:
# 192.168.1.100:5555   device
```

---

### Workflow B: Recording and Extracting a Demonstration Video

Often used to attach video screencasts to GitHub PRs or task tracking systems.

#### Step 1: Trigger Screen Record
Start recording the screen. Command will output a default 180p MP4 file. The max length is 180 seconds (3 minutes). Press `Ctrl + C` in the CLI to stop recording earlier.
```bash
adb shell screenrecord /sdcard/Movies/demo_run.mp4
```

#### Step 2: Extract File to Local PC
Once recording completes, copy the generated file from the device storage straight to your local directory.
```bash
adb pull /sdcard/Movies/demo_run.mp4 ~/Desktop/
```

#### Step 3: Remove File from Device to Free Space
```bash
adb shell rm /sdcard/Movies/demo_run.mp4
```

---

### Workflow C: Troubleshooting an "Unauthorized" Device

When executing `adb devices`, if the status displays `unauthorized`, the device is waiting for the user to grant explicit USB debugging permissions through the security popup prompt on the phone's lock screen.

#### Step 1: Check Connection Status
```bash
adb devices
# Output:
# SerialNumberXYZ    unauthorized
```

#### Step 2: Force Reset ADB Host Keys
If the popup confirmation prompt does not appear automatically on the mobile handset screen:
1. Revoke existing permissions: Go to **Settings > System > Developer Options > Revoke USB debugging authorizations**.
2. Kill active ADB server on host PC:
   ```bash
   adb kill-server
   ```
3. Re-plug USB physical cable.
4. Start the ADB server fresh:
   ```bash
   adb start-server
   ```
5. Look at your Android phone panel and accept the security dialog asking for authorization ("Always allow from this computer").

---

### Workflow D: Simulating Broadcast Events & Intents

Used during automated testing to simulate external state changes, such as low memory, battery level warnings, or network configuration events.

#### Simulate Low Battery Broadcast
If you want to trigger battery levels to test low-power UX features:
```bash
# Set battery health state to not charging
adb shell dumpsys battery set status 2 

# Manually assign battery charge level percentage
adb shell dumpsys battery set level 5

# Reset diagnostic systems back to live readings
adb shell dumpsys battery reset
```

#### Simulate Custom App Intent Trigger
To test deep links or dynamic activities using a custom scheme:
```bash
adb shell am start -a android.intent.action.VIEW -d "searchapp://results?query=android"
```

---

### Workflow E: Profile and Performance Audits

How to capture and review performance bottlenecks from frame rates to CPU loops.

#### Profiling Frame Latency (Skipped Frames)
Check whether your app is dropping frames or failing to hit target FPS marks:
```bash
# Clear existing logs for a clean profiling window
adb shell dumpsys gfxinfo com.example.mydebugapp reset

# [Perform swipes, transitions, scrolls in your App on-device now]

# Dump layout analytics statistics
adb shell dumpsys gfxinfo com.example.mydebugapp
```

#### Capturing CPU Profiles
See active process loops, system loads, and trace threads:
```bash
# Streams top active threads running on the system in real time
adb shell top -m 10 -s cpu
```

---

## 5. ADB Command Cheat Sheet Summary

| Focus Area | Command Syntax | Useful Flag/Details |
| :--- | :--- | :--- |
| **Connectivity** | `adb devices` | `-l` for extended labels |
| **TCP/IP Setup** | `adb tcpip 5555` | Prepares wireless link |
| **ADB Lifecycle** | `adb kill-server` | Restart server to solve hang |
| **File Push** | `adb push <local> <remote>` | Push data onto device |
| **File Pull** | `adb pull <remote> <local>` | Pull logs or screenshots to PC |
| **Fast Install** | `adb install -r -g app.apk` | Keep app data & auto-grant permissions |
| **List User Apps**| `adb shell pm list packages -3` | Identify third party packages |
| **Reset App State**| `adb shell pm clear <pkg>` | Clear cache/databases instantly |
| **Launch Activity**| `adb shell am start -n <class>` | Start customized activity |
| **Force Stop App**| `adb shell am force-stop <pkg>` | Kills application instantly |
| **Live Logs** | `adb logcat *:E` | Stream warning and error levels |
| **Log Clearing** | `adb logcat -c` | Clear history logs from cache buffer |
| **Screen Capture** | `adb shell screencap -p /sdcard/s.png` | Screen capture command |
| **Touch Interact** | `adb shell input tap 500 500` | Simulates screen click coordinates |
| **Hardware Key** | `adb shell input keyevent 4` | Simulate Back navigation button |
| **Reboot Mode** | `adb reboot bootloader` | Enters fastboot environment |

---
*For official documentation, consult the [Android Developer Tooling Site](https://developer.android.com/tools/adb).*
