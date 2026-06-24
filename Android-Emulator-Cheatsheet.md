# Android Emulator Security Research Cheatsheet

## Quick Start

```bash
# Gain root access
adb root

# Enter shell as root
adb shell

# Check if you have root
whoami  # Should output: root
```

---

## ADB Connection & Setup

```bash
# Start adb server
adb start-server

# List connected devices
adb devices

# Connect to emulator
adb connect localhost:5555

# Disconnect
adb disconnect

# Reboot emulator
adb reboot

# Reboot to bootloader
adb reboot bootloader

# Kill adb server
adb kill-server
```

---

## Device Information

```bash
# Get all device properties
adb shell getprop

# Specific properties
adb shell getprop ro.build.version.release        # Android version
adb shell getprop ro.build.version.sdk            # SDK version
adb shell getprop ro.product.model                # Device model
adb shell getprop ro.build.fingerprint            # Build fingerprint
adb shell getprop ro.debuggable                   # Debuggable (1 = yes)
adb shell getprop persist.sys.locale              # Language/Region

# Get device serial
adb shell getprop ro.serialno

# Check SELinux status
adb shell getenforce
adb shell setenforce 0  # Disable SELinux (Permissive)
adb shell setenforce 1  # Enable SELinux (Enforcing)
```

---

## File System Access & Permissions

```bash
# List directory contents
adb shell ls -la /data/data/                      # App data directory
adb shell ls -la /data/app/                       # Installed apps
adb shell ls -la /system/app/                     # System apps
adb shell ls -la /vendor/                         # Vendor partition

# Check file permissions
adb shell stat /data/data/com.example.app/

# Change permissions
adb shell chmod 777 /path/to/file

# Change ownership
adb shell chown root:root /path/to/file

# Read protected files
adb shell cat /data/data/com.example.app/databases/app.db
adb shell cat /proc/version                       # Kernel info
adb shell cat /system/build.prop                  # Build properties
```

---

## Package & Application Management

```bash
# List all packages
adb shell pm list packages                        # All packages
adb shell pm list packages -s                     # System packages
adb shell pm list packages -3                     # Third-party packages
adb shell pm list packages -u                     # Uninstalled packages

# Get package info
adb shell dumpsys package com.example.app

# Get app UID/GID
adb shell cat /data/system/packages.list | grep com.example.app

# Install APK
adb install app.apk
adb install -r app.apk                            # Replace existing
adb install -g app.apk                            # Grant permissions

# Uninstall app
adb uninstall com.example.app

# Start activity
adb shell am start -n com.example.app/.MainActivity

# Start service
adb shell am startservice com.example.app/.MyService

# Get app permissions
adb shell dumpsys package com.example.app | grep android.permission

# Grant permission at runtime
adb shell pm grant com.example.app android.permission.CAMERA
adb shell pm revoke com.example.app android.permission.CAMERA
```

---

## Database & Data Extraction

```bash
# Pull database from app
adb pull /data/data/com.example.app/databases/app.db

# Push file to emulator
adb push local_file /data/data/com.example.app/

# Access SQLite database
adb shell sqlite3 /data/data/com.example.app/databases/app.db

# Common SQLite commands (inside shell)
.tables                                           # List tables
.schema table_name                                # Show schema
SELECT * FROM table_name;                         # Query data
.dump table_name > /sdcard/backup.sql             # Export table

# Read shared preferences
adb shell cat /data/data/com.example.app/shared_prefs/prefs.xml

# Pull entire app directory
adb pull /data/data/com.example.app/ ./app_backup/
```

---

## Network & Traffic Analysis

```bash
# Get network interface info
adb shell ifconfig
adb shell ip addr

# Test connectivity
adb shell ping google.com

# Get open ports
adb shell netstat -tuln

# Get network statistics
adb shell netstat -i

# Dump network configuration
adb shell dumpsys connectivity

# Check DNS
adb shell getprop ro.dns1
adb shell getprop ro.dns2

# Capture network traffic (tcpdump)
adb shell tcpdump -i any -w /sdcard/capture.pcap
# Then: adb pull /sdcard/capture.pcap
```

---

## Process & Memory Analysis

```bash
# List all processes
adb shell ps -A
adb shell ps -A | grep com.example.app

# Get process details
adb shell ps -ef com.example.app

# Monitor process memory
adb shell dumpsys meminfo com.example.app

# Get all running services
adb shell dumpsys activity services

# Get running activities
adb shell dumpsys activity activities | grep mFocusedActivity

# Memory usage
adb shell free                                    # RAM usage
adb shell df                                      # Disk usage
```

---

## Logging & Debugging

```bash
# Get logcat output
adb logcat

# Filter logs by app
adb logcat | grep com.example.app

# Filter by log level
adb logcat *:W                                    # Warnings and above
adb logcat *:E                                    # Errors only

# Save logs to file
adb logcat > logfile.txt

# Clear logcat
adb logcat -c

# Get crash logs
adb logcat | grep FATAL
adb logcat | grep AndroidRuntime

# Get system logs
adb shell dmesg                                   # Kernel logs
adb shell cat /var/log/logcat                     # System logcat history
```

---

## Frida & Dynamic Analysis

```bash
# Install Frida server on emulator
adb push frida-server /data/local/tmp/
adb shell chmod +x /data/local/tmp/frida-server
adb shell /data/local/tmp/frida-server

# Forward port (in separate terminal)
adb forward tcp:27042 tcp:27042

# List running processes (from Frida CLI)
frida-ps -U

# Attach to app
frida -U com.example.app

# Spawn app with Frida
frida -U -f com.example.app
```

---

## Burp Suite Integration

```bash
# Set proxy on emulator
adb shell settings put global http_proxy 10.0.2.2:8080

# Remove proxy
adb shell settings put global http_proxy :0

# For HTTPS/Certificate Pinning
# 1. Export Burp CA certificate
# 2. Convert to .0 hash format
# 3. Push to system certs:
adb push burp_cert.0 /system/etc/security/cacerts/

# Disable certificate pinning (if needed)
# Use Frida script to bypass
```

---

## Intent & Broadcast Testing

```bash
# Send broadcast
adb shell am broadcast -a android.intent.action.BOOT_COMPLETED

# Start intent with data
adb shell am start -a android.intent.action.VIEW -d "https://example.com"

# Send intent with extras
adb shell am start -a android.intent.action.SEND \
  --es android.intent.extra.TEXT "Message"

# List available intents
adb shell dumpsys package resolvers
```

---

## Sensitive Data & Credentials

```bash
# Check for hardcoded credentials
adb shell grep -r "password" /data/data/com.example.app/
adb shell grep -r "apikey" /data/data/com.example.app/
adb shell grep -r "token" /data/data/com.example.app/

# Check SharedPreferences
adb shell cat /data/data/com.example.app/shared_prefs/*.xml

# Check environment variables
adb shell printenv

# Look for config files
adb shell find /data/data/com.example.app/ -name "*.conf"
adb shell find /data/data/com.example.app/ -name "*.xml"
adb shell find /data/data/com.example.app/ -name "*.json"

# Check app cache
adb shell ls -la /data/data/com.example.app/cache/
adb shell cat /data/data/com.example.app/cache/*
```

---

## File Transfer

```bash
# Push file to emulator
adb push /path/to/local/file /data/local/tmp/

# Pull file from emulator
adb pull /data/data/com.example.app/file.db ./

# Push entire directory
adb push ./local_dir /data/local/tmp/

# Pull entire directory
adb pull /data/data/com.example.app/ ./app_backup/

# Push APK and install
adb push test.apk /data/local/tmp/
adb shell pm install /data/local/tmp/test.apk
```

---

## Root & Privilege Escalation Testing

```bash
# Check current user
adb shell whoami
adb shell id

# Check sudoers
adb shell cat /etc/sudoers

# List sudo commands available
adb shell sudo -l

# Check for SUID binaries
adb shell find / -perm -4000 2>/dev/null

# Check capabilities
adb shell getcap /path/to/binary

# Test privilege escalation
adb shell su  # Should work on rooted emulator
```

---

## WebView & JavaScript Debugging

```bash
# Check WebView version
adb shell dumpsys package com.android.webview

# Enable WebView debugging
adb shell setprop log.tag.WebView VERBOSE

# Get WebView processes
adb shell ps | grep webview

# Check for JavaScript enabled
adb shell dumpsys package com.example.app | grep webview
```

---

## Reverse Engineering

```bash
# Pull APK from emulator
adb shell pm path com.example.app                 # Get APK path
adb pull /data/app/com.example.app-ABC/base.apk

# Decompile APK
apktool d base.apk
jadx base.apk

# Check APK signature
jarsigner -verify -verbose -certs base.apk

# Extract certificate info
adb shell dumpsys package com.example.app | grep Certificates
```

---

## Custom Shell Scripts (One-Liners)

```bash
# Find all data files in app directory
adb shell find /data/data/com.example.app -type f

# Search for strings in files
adb shell grep -r "SECRET_KEY" /data/data/

# Monitor app startup
adb shell dumpsys activity intents | grep com.example.app

# Get all app URLs/Intents
adb shell dumpsys package com.example.app | grep -i intent

# Monitor file access
adb shell strace -f -e openat -p $(adb shell pidof com.example.app)

# Check app certificate pinning
adb shell dumpsys package com.example.app | grep -i pin
```

---

## Common Security Research Scenarios

### Scenario 1: Extract App Database
```bash
adb root
adb shell
cd /data/data/com.example.app/databases/
ls -la
sqlite3 app.db ".tables"
sqlite3 app.db "SELECT * FROM users;" 
exit
adb pull /data/data/com.example.app/databases/app.db ./
```

### Scenario 2: Monitor App Permissions
```bash
adb shell dumpsys package com.example.app | grep -A 100 "granted permissions"
```

### Scenario 3: Capture Network Traffic
```bash
adb shell tcpdump -i any -w /sdcard/traffic.pcap &
# Run app actions
# Stop with: adb shell killall tcpdump
adb pull /sdcard/traffic.pcap ./
# Analyze with Wireshark
```

### Scenario 4: Hook Method with Frida
```bash
# Create hook.js
Java.perform(function() {
  var MainActivity = Java.use("com.example.app.MainActivity");
  MainActivity.sensitiveMethod.implementation = function(arg1) {
    console.log("sensitiveMethod called with: " + arg1);
    return this.sensitiveMethod(arg1);
  };
});

# Run:
frida -U -l hook.js -f com.example.app
```

### Scenario 5: Bypass Certificate Pinning
```bash
# Use Frida to hook SSL verification
# Download universal unpinner script or write custom one
frida -U -l unpinner.js -f com.example.app
```

---

## Useful Tools & Setup

```bash
# Install useful tools in emulator
adb shell pm install sqlite3
adb shell pm install strace
adb shell pm install tcpdump

# Or manually:
adb push sqlite3 /data/local/tmp/
adb shell chmod +x /data/local/tmp/sqlite3
```

---

## Tips & Best Practices

✅ **Always backup** before testing: `adb pull /data/data/com.example.app/ ./backup/`

✅ **Use separate emulator instances** for different test scenarios

✅ **Disable SELinux** for testing: `adb shell setenforce 0`

✅ **Enable USB Debugging** in emulator settings

✅ **Snapshot before testing** invasive changes

✅ **Use logcat filtering** to focus on specific apps

✅ **Combine tools**: adb + Frida + Burp Suite for comprehensive testing

✅ **Check file permissions** before accessing sensitive data

✅ **Monitor background processes** while app is running

---

## Quick Reference

| Task | Command |
|------|---------|
| Root shell | `adb shell` (if rooted) |
| List apps | `adb shell pm list packages` |
| Install APK | `adb install app.apk` |
| Pull file | `adb pull /path/file ./` |
| Push file | `adb push ./file /path/` |
| View logs | `adb logcat` |
| Check props | `adb shell getprop` |
| Network info | `adb shell netstat -tuln` |
| Processes | `adb shell ps -A` |
| SQLite query | `adb shell sqlite3 /path/app.db` |

---

**Last Updated:** 2026
**For:** Android Security Research & Testing
