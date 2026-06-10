# Fastboot Command Reference

Fastboot is a powerful command-line tool used primarily to interact with the bootloader of Android devices. It allows you to flash images, boot custom kernels, and manage device partitions.

## Important Note
Using fastboot commands incorrectly can damage your device (bricking). Ensure you have the correct files and a stable connection before proceeding.

---

## Core Fastboot Commands

### Device Connection
* `fastboot devices`: Lists all connected devices currently in fastboot mode. Useful for verifying the connection.
* `fastboot reboot`: Exits fastboot mode and reboots the device normally.
* `fastboot reboot-bootloader`: Reboots the device back into the bootloader/fastboot mode.

### Information
* `fastboot getvar all`: Displays all available bootloader variables (e.g., product name, serial number, secure boot status).
* `fastboot oem device-info`: Checks the bootloader unlock status (e.g., `Device unlocked: true/false`).

### Flashing Commands
* `fastboot flash <partition> <filename.img>`: Writes a specific image file to a designated partition (e.g., `fastboot flash boot boot.img`).
* `fastboot flashall`: Flashes all partitions defined in a `flash-all.zip` script (usually found in factory image packages).
* `fastboot flash:raw <partition> <kernel> [ramdisk]`: Flashes a kernel/ramdisk directly. Rarely used by general users.

### Partition Management
* `fastboot erase <partition>`: Erases/wipes a specific partition (e.g., `fastboot erase userdata`). **Warning: This usually wipes data.**
* `fastboot format <partition>`: Formats a specific partition. Often used for clearing data or cache.

### Booting
* `fastboot boot <filename.img>`: Temporarily boots a kernel image without permanently flashing it to the device. Useful for testing custom recoveries (e.g., `fastboot boot twrp.img`).

### Unlocking/Locking
* `fastboot oem unlock`: Unlocks the device bootloader (device-specific; may wipe data).
* `fastboot oem lock`: Relocks the device bootloader.
* `fastboot flashing unlock`: A newer command for unlocking the bootloader on modern Android devices.
* `fastboot flashing lock`: A newer command for locking the bootloader on modern Android devices.

---

## Troubleshooting
If `fastboot devices` returns no results:
1. Check your USB cable and port.
2. Ensure you have the latest Android SDK Platform-Tools installed.
3. Verify that the device is actually in fastboot mode (usually indicated on the screen).
4. Update or reinstall your device drivers (Windows).
