<div align="center">

<img src="https://img.shields.io/badge/Version-3.2%20Apex-0078D4?style=for-the-badge&logo=android&logoColor=white"/>
<img src="https://img.shields.io/badge/Platform-Android-3DDC84?style=for-the-badge&logo=android&logoColor=white"/>
<img src="https://img.shields.io/badge/Architecture-Zero--Trust-0A1628?style=for-the-badge"/>
<img src="https://img.shields.io/badge/Status-Research%20Prototype-FF6B00?style=for-the-badge"/>
<img src="https://img.shields.io/badge/NDK-C%2B%2B17-00599C?style=for-the-badge&logo=cplusplus&logoColor=white"/>

# 🛡️ AegisVerity V3.2 (Apex Edition)

**Holistic Zero-Trust Mobile Defense via Offline Hardware Attestation,  
Asymmetric Entropy Telemetry, and Deterministic Lockdown**

*A research-grade Android security architecture engineered to neutralize  
Advanced Persistent Threats, physical Evil Maid attacks, and obfuscated  
data exfiltration — without relying on any external cloud dependency.*

</div>

---

## 📋 Table of Contents

- [Executive Summary](#-executive-summary)
- [Threat Landscape](#-threat-landscape--motivation)
- [Architecture Overview](#-architecture-overview)
- [Pre-Flight Attestation](#-pre-flight-integrity--hardware-attestation)
- [Module Details](#-module-details)
- [Incident Response](#-incident-response-scenarios)
- [Design Ethics](#-design-ethics--privacy-principles)
- [Technical Stack](#-technical-stack)
- [Project Status](#-project-status)
- [Whitepaper](#-whitepaper)
- [License](#-license)

---

## 🎯 Executive Summary

AegisVerity V3.2 (Apex Edition) is a **research-grade Android security architecture** that has evolved through three rigorous design review cycles (V3.0 Titan → V3.1 Hardened → V3.2 Apex). The system transitions from reactive, signature-based detection to a **proactive, behavioral, and hardware-enforced defense architecture**.

Operating under an **"Assume Breach"** posture, it prioritizes:

- **Impact containment** over perimeter prevention  
- **Hardware-rooted trust** via StrongBox TEE — no cloud dependency  
- **Deterministic enforcement** via Android Device Owner (DO) privileges  
- **User privacy** as a non-negotiable design constraint  

> ⚠️ **Research Prototype** — AegisVerity is an architecture research project.  
> It demonstrates what is *possible* within the Android security model.  
> It is not a production-ready consumer application.

---

## 🌐 Threat Landscape & Motivation

Modern mobile threats that conventional EPPs fail to address:

| Threat Vector | Conventional EPP | AegisVerity V3.2 |
|---|---|---|
| Encrypted C2 Tunneling | Signature-based only | Behavioral entropy analysis |
| DNS Protocol Evasion (DoT/DoQ) | Blind | Passive DNS correlation + Port 853 radar |
| APK Repackaging / Tampering | No self-verification | Hardware-rooted dual-layer integrity |
| Kernel Rootkit (pre-boot) | Post-infection detection | StrongBox TEE attestation gate |
| Physical Evil Maid Attack | Disabled by attacker | TEE biometric + volatile sessions |
| ADB Privilege Stripping | Race condition exploitable | Genesis Block zero-window locking |
| Background Sensor Surveillance | OS permissions only | Device Owner hardware sensor kill |

---

## 🏗️ Architecture Overview

AegisVerity V3.2 is structured as a **closed-loop defense matrix** of six independent modules. No single module is a single point of failure.

```
┌─────────────────────────────────────────────────────────────────┐
│                    AEGISVERITY V3.2 — APEX                      │
│                                                                 │
│  ┌──────────────┐    ┌─────────────────────┐    ┌───────────┐  │
│  │  HYDRA CORE  │    │  CORTEX & THE ORACLE │    │  OBSIDIAN │  │
│  │  Network     │───▶│  Asymmetric DPI      │    │  GUARD    │  │
│  │  Telemetry   │    │  Entropy Analysis    │    │  NDK/C++  │  │
│  └──────┬───────┘    └──────────┬──────────┘    └─────┬─────┘  │
│         │                       │                      │        │
│         ▼                       ▼                      ▼        │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                   REACTOR CORE                          │   │
│  │            Device Owner Enforcement Engine              │   │
│  │  Genesis Block │ Hardware Lockdown │ Nuclear Failsafe   │   │
│  └─────────────────────────────────────────────────────────┘   │
│         ▲                       ▲                      ▲        │
│  ┌──────┴───────┐    ┌──────────┴──────────┐    ┌─────┴─────┐  │
│  │    IAM       │    │   SMART GATEKEEPER   │    │  PRE-FLIGHT│  │
│  │  Front Door  │    │   Sensor Control     │    │  TEE ATTEST│  │
│  └──────────────┘    └─────────────────────┘    └───────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔐 Pre-Flight Integrity & Hardware Attestation

Before any security logic executes, AegisVerity performs **hardware-rooted integrity verification** of the device environment.

### Offline StrongBox TEE Attestation

Unlike solutions relying on cloud APIs (vulnerable to network-level attacks), V3.2 leverages the device's **dedicated hardware security module (StrongBox)** to generate cryptographic proof of device integrity — verified entirely **offline**.

```
Device Boot
    │
    ▼
StrongBox TEE
    │ generates X.509 certificate chain
    ▼
Offline verification against hardcoded Root CAs (C++ native layer)
    │
    ├── MEETS_STRONG_INTEGRITY  ──▶ Full initialization
    ├── MEETS_DEVICE_INTEGRITY  ──▶ Degraded mode + user notification  
    └── FAILS_INTEGRITY         ──▶ REFUSE TO INITIALIZE
```

| Device State | Attestation Result | Behavior |
|---|---|---|
| Unmodified, verified bootloader | MEETS_STRONG_INTEGRITY | Full initialization |
| Hardware-backed key available | MEETS_DEVICE_INTEGRITY | Degraded mode |
| Unlocked bootloader / custom ROM | FAILS_INTEGRITY | Refuses to initialize |
| StrongBox unavailable | Hardware not present | Software TEE fallback |

---

## 📦 Module Details

### Module A — Hydra Core (Network Telemetry Engine)

Passive `NetworkStatsManager` polling to detect **Ghost Protocol** — traffic discrepancies between kernel-level interface stats and application-level UIDs that indicate rootkit-hidden data transmission.

- **Ghost Protocol Detection**: Volumetric delta analysis between `TrafficStats` and per-UID stats  
- **Lazarus Watchdog**: Signature-protected `BroadcastReceiver` + atomic rate-limited `startForeground` to survive OEM battery-killers and DoS crash-loop attacks

---

### Module B — The Cortex & The Oracle (Asymmetric DPI Pipeline)

A two-tier inspection pipeline balancing accuracy with performance.

**The Oracle (Fast Path)**  
Concurrent `LruCache` bounded to **5,000 entries** (preventing OOM and Eviction Storm attacks) backed by asynchronous Reverse DNS. Verified CDN traffic bypasses inspection.

**ECH Counter-measure**: Correlates outbound encrypted packets with preceding plaintext DNS requests — no TLS decryption required, no privacy violation.

**The Cortex (Slow Path)**  
Shannon Entropy `H(X)` analysis of unverified traffic payloads, executed at the **native C++ layer** for performance and instrumentation resistance.

```
H(X) = -Σ[p(x) · log₂(p(x))]
```

**DoT / DoQ Port 853 Radar**: Detects DNS-over-TLS (TCP) and DNS-over-QUIC (UDP) tunnel attempts with ARM-safe unsigned port extraction.

---

### Module C — Obsidian Guard (Native Self-Protection Layer)

NDK-based C++ self-defense module operating below the Java security model.

**Dual-Layer Supply Chain Verification**:
- Layer 1: APK signing certificate SHA-256 verification (constant-time comparison, prevents timing side-channel)
- Layer 2: Native `.so` binary self-integrity hash verification

**Anti-Debug Triple Layer**:
- `TracerPid` check via `/proc/self/status`
- Nanosecond-precision timing heuristic (detects single-step debugging latency)
- `/proc/self/maps` scan for Frida, Xposed, linjector, and Substrate footprints

**ARM-Safe Memory Handling**:
```cpp
// Prevents SIGBUS on ARM (unaligned memory access)
uint32_t dest_ip_raw;
memcpy(&dest_ip_raw, &packet[16], 4);
inet_ntop(AF_INET, &dest_ip_raw, dest_ip, INET_ADDRSTRLEN);
```

On any integrity violation → immediate `_exit(1)`. No state preserved, no prompt displayed.

---

### Module D — Identity & Access Management

- Sessions stored as `AtomicBoolean` in **volatile memory only** — never written to disk  
- Timeout validated via `SystemClock.elapsedRealtime()` — immune to NTP Time Spoofing  
- `ON_STOP` lifecycle → **unconditional session revocation**  
- `FLAG_SECURE` → visual concealment from Android task switcher  

---

### Module E — Smart Gatekeeper (Zero-Trust Sensor Control)

On `ACTION_SCREEN_OFF`, enforces sensor access restrictions across all non-whitelisted applications using `DevicePolicyManager.setPermissionGrantState()` — **not** standard runtime permissions which lack cross-application authority.

```
Screen OFF
    │
    ▼
setPermissionGrantState(camera)   → PERMISSION_GRANT_STATE_DENIED
setPermissionGrantState(microphone) → PERMISSION_GRANT_STATE_DENIED
    │
    ▼ (for all non-whitelisted UIDs)
Background surveillance capability: ELIMINATED
```

---

### Module F — Reactor Core (Indestructible Device Owner)

Final enforcement authority receiving threat signals from all modules.

**Genesis Block — Zero-Window Privilege Locking**:

```kotlin
// V3.1 (VULNERABLE — race condition window exists)
dpm.setActiveAdmin(...)
// ← attacker can execute: dpm remove-active-admin HERE
dpm.addUserRestriction(DISALLOW_DEBUGGING_FEATURES)

// V3.2 (HARDENED — zero window)
override fun onProfileProvisioningComplete(...) {
    // This callback fires AT THE MOMENT DO is granted
    dpm.addUserRestriction(DISALLOW_DEBUGGING_FEATURES)
    // No window. No race condition.
}
```

**System App Safeguard**: Logic gate preventing suspension of `FLAG_SYSTEM` packages — protects against logic-induced bootloops.

**Ghost Switch + Biometric Gate**:
```kotlin
fun onGhostSwitchTapped(context: Context) {
    val now = System.currentTimeMillis()
    if (now - lastTapTime > tapWindowMs) tapCount = 0  // 3s window
    lastTapTime = now

    val km = context.getSystemService(KEYGUARD_SERVICE) as KeyguardManager
    if (km.isKeyguardLocked) { tapCount = 0; return }  // Prevent tap accumulation

    if (++tapCount >= 7) {
        tapCount = 0
        initiateNuclearFailsafeWithBiometrics()  // BIOMETRIC_STRONG required
    }
}
```

---

## 🚨 Incident Response Scenarios

### Scenario A: Background Data Exfiltration
```
Screen OFF → Smart Gatekeeper activates
    → Hydra detects TX spike above screen-off threshold
        → Reactor: suspend offending UID
        → Reactor: lockNow()
        → Reactor: maintain setCameraDisabled(true)
        → Audit log entry written
```

### Scenario B: Obfuscated C2 via ECH
```
Unknown app → low-reputation IP
    → Oracle: Cache Miss → async DNS resolution
    → Oracle: DNS correlation check
    → Cortex (C++): entropy analysis
        → H(X) > threshold → packet DROPPED at tun interface
        → Reactor: setPackagesSuspended(offending UID)
        → Audit log entry written
```

### Scenario C: Physical Evil Maid
```
Attacker gains unlocked device
    → Opens AegisVerity
    → AppLifecycleObserver: ON_START detected
    → SessionVault: session was revoked on prior ON_STOP
    → Hard redirect to BiometricPrompt (CLEAR_TASK)
    → Attacker sees: biometric scanner
    → Dashboard: INACCESSIBLE
```

---

## ⚖️ Design Ethics & Privacy Principles

AegisVerity was built with an explicit commitment that **security must not be achieved at the expense of user privacy**.

### Rejected: Forced DNS Plaintext Mode

During development, an approach was evaluated that would disable Private DNS (DoT/DoH) system-wide to give the Oracle complete DNS visibility.

**This was explicitly rejected** because:
- It exposes all domain queries to ISPs and network observers
- On hostile public networks, it opens new DNS spoofing attack surfaces  
- It reduces privacy for all applications, not just monitored ones
- Security and privacy must be compatible goals, not competing ones

**Adopted alternative**: Passive DNS correlation — comparable detection capability without downgrading existing encryption.

### Privacy Architecture Commitments

- ✅ No traffic *content* is decrypted or stored — only behavioral metadata  
- ✅ Audit logs contain network metadata only, never payload content  
- ✅ Users retain ultimate control via authenticated Ghost Switch failsafe  
- ✅ All enforcement is transparent through audit logging  
- ✅ Sensor restrictions protect users *from third-party apps*, not from themselves  

---

## 🛠️ Technical Stack

| Component | Technology | Version |
|---|---|---|
| Native Layer | C++ (NDK) | C++17, CMake 3.22.1 |
| Application Layer | Kotlin + Coroutines | Kotlin 1.9+ |
| UI Framework | Jetpack Compose Material3 | BOM 2024+ |
| Biometric Auth | androidx.biometric | 1.1.0 (stable) |
| Encrypted Database | SQLCipher for Android | 4.5+ |
| Cryptography | OpenSSL (static) | 3.x |
| Network Monitor | NetworkStatsManager | API 23+ |
| VPN Engine | VpnService | API 14+ |
| Hardware Attestation | Android Keystore / StrongBox | API 28+ |
| Device Policy | DevicePolicyManager | API 21+ |
| **Minimum SDK** | **Android 8.0 (Oreo)** | **API 26** |
| **Target SDK** | **Android 14** | **API 34** |

---

## 📊 Project Status

| Capability | Status |
|---|---|
| Pre-boot hardware integrity attestation | ✅ Implemented |
| APK supply chain dual-layer verification | ✅ Implemented |
| Native anti-analysis triple layer | ✅ Implemented |
| Encrypted DNS evasion detection (DoT/DoQ) | ✅ Implemented |
| C2 behavioral entropy analysis | ✅ Implemented |
| Genesis Block zero-window DO locking | ✅ Implemented |
| Cross-application sensor control | ✅ Implemented |
| Physical access biometric gate | ✅ Implemented |
| System stability FLAG_SYSTEM safeguard | ✅ Implemented |
| Forensic SQLCipher audit logging | ✅ Implemented |
| IPv6 inspection support | 🔄 Roadmap |
| Multi-user / Work Profile support | 🔄 Roadmap |
| SIEM platform export (mTLS) | 🔄 Roadmap |
| NIST SP 800-124 / OWASP MASVS alignment | 🔄 Roadmap |

---

## 📄 Whitepaper

A full public architecture overview is available:

> **AegisVerity V3.2 — Public Architecture Overview (Redacted Edition)**  
> Selected implementation details are withheld from the public release.  
> The full unredacted document is available to qualified enterprise partners under NDA.

*[Link to whitepaper — add your Speakerdeck / Google Drive link here]*

---

## 🔒 Responsible Disclosure

If you identify a logical flaw or security gap in the architecture described in this repository, responsible disclosure is appreciated. Please open a GitHub Issue marked `[SECURITY RESEARCH]` or reach out directly.

This is a research project. Identified gaps contribute to making the architecture stronger.

---

## 📜 License

```
AegisVerity V3.2 (Apex Edition)
Copyright (c) 2026 — All Rights Reserved

This repository contains architecture documentation and reference 
implementation code for research purposes. The architecture concepts, 
module designs, and security mechanisms described herein are proprietary.

Permission is granted to read, study, and reference this work for 
academic and security research purposes with attribution.

Commercial use, redistribution, or deployment requires explicit 
written permission from the author.
```

---

<div align="center">

**AegisVerity V3.2 — Apex Edition**  
*Engineered for the threat landscape of today. Architected for the adversaries of tomorrow.*

[![LinkedIn](https://img.shields.io/badge/Connect-LinkedIn-0077B5?style=flat&logo=linkedin)](https://linkedin.com/in/your-profile)

</div>
