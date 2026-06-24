# Security Threat Categories

## 1. Binary Protections

Identify and bypass mechanisms designed to prevent reverse engineering, tampering, or modification of the application binary.

---

## 2. Certificate Pinning Bypass

Demonstrate the ability to intercept and inspect protected network traffic by bypassing or disabling certificate pinning.

---

## 3. Environment Checks

### 3.1 Jailbreak Detection *(iOS)*
Bypass detection mechanisms using public or custom techniques.

### 3.2 Root Detection *(Android)*
Bypass mechanisms that detect rooted devices, allowing the application to run in a rooted environment despite protections.

### 3.3 Debug Detection
Identify and bypass mechanisms that detect debugging attempts.

### 3.4 Emulator Detection
Bypass mechanisms that detect emulators, allowing the app to run in an emulator.

### 3.5 Hooking Detection *(e.g. Frida, Xposed)*
Bypass protections against dynamic instrumentation.

---

## 4. Exploitation

Demonstrate practical exploitation of identified weaknesses, such as:

- **Remote Code Execution (RCE)** — Execute arbitrary code remotely by exploiting application vulnerabilities.
- **Local Privilege Escalation** — Gain elevated permissions beyond those initially granted to a process or user.
- **Sensitive Data Exfiltration** — Extract protected or confidential data from the application or its environment.

---

## 5. Cryptographic Keys

Extraction or exposure of request signing keys or similar secrets protecting sensitive endpoints.

---

## 6. Hardcoded Credentials

Discovery of hardcoded credentials that provide access to sensitive systems or customer data.


## Reference

1. https://github.com/android/security-samples/tree/main
2. https://source.android.com/docs/core/architecture/bootloader/locking_unlocking
