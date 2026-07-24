# 📱 Device Compatibility Guide

This document defines the compatibility requirements for nikitos4683 kernels.

---

## 💎 Supported Target: OnePlus 13 (OP13)

This repository builds exactly one kernel: **OnePlus 13** on **OxygenOS 16**. There is no secondary target.

> [!IMPORTANT]
> **Stock ROMs Only:** The kernel is built using the official [OnePlus Source](https://github.com/OnePlusOSS/) and is strictly intended for **Stock OxygenOS 16** (Android 16) installations.

### 📜 Technical Specifications
*   **Model:** OnePlus 13 (`OP13`), SoC `sm8750`
*   **OS Version:** OxygenOS 16 (Android 16) — `A16`
*   **GKI base (KMI):** `android15-6.6` — an ABI designation, **not** the marketing OS version
*   **Kernel Version:** `6.6.x`
*   **Kernel Source:** OnePlusOSS branch `oneplus/sm8750_b_16.0.0_oneplus_13`, tracked at build time
*   **Manifest:** local `manifests/a16/oneplus_13_w.xml`

---

## ✅ Before You Flash: Check Your KMI

A GKI kernel only fits firmware with a matching KMI. Verify it instead of guessing:

```bash
adb shell uname -r
```

*   Your stock kernel must report the **`android15-6.6`** KMI — i.e. something like `6.6.30-android15-…`.
*   If it reports anything else (for example `android16-6.12`), **this kernel is not built for your firmware** — do not flash it.
*   The base a release was built against is encoded in the ZIP filename:
    `AK3-NIKITOS4683-OP13_A16_android15-6.6.30_KSUN_…_SuSFS_….zip`

After a successful flash, `uname -r` ends with `-android15-nikitos4683`.

---

## ⚠️ Stability & Risks

*   **Major OTAs:** Do not reuse a kernel across a major Android upgrade (e.g. A16 → A17). Wait for a build released against the new base.
*   **Minor OTAs:** After any OxygenOS update, re-check `uname -r` and confirm the KMI is unchanged before reflashing an older ZIP.
*   **Cross-device flashing:** Anything outside OP13 / A16 is unsupported unless a release explicitly says otherwise.
*   **Other OnePlus devices:** Use the upstream [WildKernels project](https://github.com/WildKernels/OnePlus_KernelSU_SUSFS) instead — this fork will not add devices.

---

<div align="center">
  <sub>This kernel is built specifically for the OnePlus 13. Flashing on other devices is not supported.</sub>
</div>
