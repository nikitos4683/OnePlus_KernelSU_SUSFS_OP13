<div align="center">

# 🌟 nikitos4683 Kernel for OnePlus 13

**A OnePlus 13 / OxygenOS 16 focused kernel fork with KernelSU, KernelSU-Next, SUSFS, and a curated patch stack.**

[![KernelSU](https://img.shields.io/badge/KernelSU-Supported-1a7f37?style=for-the-badge&logo=android&logoColor=white)](https://kernelsu.org/)
[![KernelSU-Next](https://img.shields.io/badge/KSU--Next-Integrated-1a7f37?style=for-the-badge&logo=github&logoColor=white)](https://kernelsu-next.github.io/webpage/)
[![SUSFS](https://img.shields.io/badge/SUSFS-Enhanced-d97706?style=for-the-badge&logo=gitlab&logoColor=white)](https://gitlab.com/simonpunk/susfs4ksu)
[![OP13 Source Status](https://img.shields.io/badge/OP13--Source-active-84cc16?style=for-the-badge&logo=github&logoColor=white)](https://github.com/WildKernels/OnePlus_KernelSU_SUSFS/blob/status-page/README.md)

### 🚀 Powered by [WildKernels](https://github.com/WildKernels) | Branch: A16

> This repository is a boutique, nikitos4683-branded kernel fork tailored for the **OnePlus 13 (OP13)** running **OxygenOS 16**.

</div>

---

## 📋 Table of Contents
- [⚠️ Disclaimer](#-disclaimer)
- [🔗 Additional Resources](#-additional-resources)
- [✨ Features](#-features)
- [🛡️ Stability Policy](#-stability-policy)
- [📝 Installation](#-installation)
- [🌟 Special Thanks](#-special-thanks)
- [💬 Support](#-support)

---

## ⚠️ Disclaimer

Flashing a custom kernel is a choice you make for your device. While we strive for stability, there is always an inherent risk.

*   **Backup:** Always keep a fresh backup of your data.
*   **Knowledge:** Read the documentation and understand the features before flashing.
*   **Responsibility:** I am **not responsible** for bricked devices, damaged hardware, or any issues arising from the use of this kernel.
*   **Your Choice:** By flashing this, you accept full responsibility for any outcomes.

---

## 🔗 Additional Resources

| Resource | Description |
| :--- | :--- |
| 🩹 [Kernel Patches](https://github.com/WildKernels/kernel_patches) | Core patches used in the build process |
| ⚡ [Kernel Flasher](https://github.com/fatalcoder524/KernelFlasher) | Recommended tool for safe flashing |
| 📱 [Compatibility Info](./compatibility.md) | Verify supported base versions |
| 📊 [OnePlusOSS Tracking Dashboard](https://github.com/WildKernels/OnePlus_KernelSU_SUSFS/blob/status-page/README.md) | Upstream OP13 / A16 source tracker |

---

## 📱 OP13 Source Tracking

- 📊 **Live Dashboard**: [OP13 source tracking & changes](https://github.com/WildKernels/OnePlus_KernelSU_SUSFS/blob/status-page/README.md)
- ⏱️ **Update Frequency**: Every 12 hours (Automated)
- 🎯 **Scope**: Only the OnePlusOSS repositories and branches referenced by the active `OP13` / `A16` manifest
---

## ✨ Features

Our build configurations are meticulously tuned for the best experience on the OnePlus 13.

### 💎 Core Integrations
*   **KernelSU / KernelSU-Next:** First-class support for both. Defaulting to `KSUN` (dev branch) for the latest security features.
*   **SUSFS Support:** Built-in hooks for advanced system-user-space filesystem isolation (SUSFS `v2.2.0`).
*   **OxygenOS 16 Optimized:** Specifically targeted at `OP13` with Android 16 & Kernel 6.6.

### 🛠️ Enabled Features
| Feature | Description |
| :--- | :--- |
| ✅ **HMBIRD SCX** | Advanced scheduler extensions |
| ✅ **BBRv3-only** | BBRv3 is the kernel default; legacy BBRv1 is explicitly disabled |
| ✅ **CAKE & PIE qdisc** | Modern queue disciplines for lower latency (bufferbloat mitigation) |
| ✅ **TTL Target** | Native support for TTL modification |
| ✅ **IP Set & IPv6 NAT** | Enhanced networking and firewall capabilities |
| ✅ **Thin LTO** | Link-Time Optimization for a balanced build performance |
| ✅ **TMPFS XATTR** | POSIX ACL Support for enhanced filesystem security |
| ✅ **Custom Branding** | Localversion set to `nikitos4683` |
| ✅ **nikitos4683 Branding** | Local AnyKernel3 custom branding patch |

### 🚫 Deliberately Disabled

*   **General Optimization Patch Stack:** Disabled.
*   **Rust / Rust Binder Build Path:** Disabled.
*   **Baseband Guard (BBG):** Disabled.
*   **Droidspaces:** Disabled.
*   **NTSync:** Disabled.
*   **Unicode Bypass Fix:** Disabled.

---

## 🛡️ Stability Policy

This fork intentionally removes several upstream customizations that modify generic kernel module loading:

| Removed customization | Current behavior |
| :--- | :--- |
| **Module intercept / overlay** | Removed completely. The kernel does not embed external prebuilt `.ko` files and does not replace vendor modules at load time. |
| **Vendor-module debloat / blacklist** | Removed completely. No custom module blacklist is generated, no `CONFIG_DEBLOAT_VENDOR_MODULES` patch is applied, and OnePlus vendor modules retain their stock loading behavior. |
| **Automatic BBR-related module blocking** | Removed together with vendor-module debloat. Enabling BBRv3 does not blacklist `oplus_network_tuning` or `oplus_networks_tuning`. |
| **General optimization patch stack** | Kept disabled instead of applying the entire experimental stack as one unit. |

The following deliberate customizations remain:

*   **BBRv3-only:** Legacy BBRv1 is disabled, and BBRv3 is selected as the kernel default congestion control.
*   **`fake_config.patch`:** Retained as part of the root-hiding configuration-reporting behavior. It changes the configuration exposed through the embedded config data, not the features actually compiled into the kernel.
*   **SUSFS, KernelSU / KernelSU-Next, HMBIRD, TTL, IP Set, IPv6 NAT, qdisc support, Thin LTO, and branding:** Retained as documented above.

---

### ⬇️ Downloads
You can download the pre-built kernels from the **[Releases](../../releases)** tab. The files are distributed as AnyKernel3 flashable ZIPs.

Filename breakdown:
`AK3-NIKITOS4683-<Device>_<OS>_<Kernel>_<KSU-Variant>_SuSFS_<Version>.zip`

---

## 📝 Installation Instructions

Because this kernel is provided as an **AnyKernel3** flashable ZIP, the standard KernelSU boot.img patching method is not needed.

1. **Download** the zip of your choice from the Releases page.
2. **Flash** the zip using a kernel flasher app (we highly recommend [Kernel Flasher](https://github.com/fatalcoder524/KernelFlasher)).
3. **Reboot** your device.
4. **Install** the appropriate manager APK:
   - For `KSU` builds: [KernelSU Manager](https://github.com/tiann/KernelSU/releases)
   - For `KSUN` builds: [KernelSU-Next Manager](https://github.com/KernelSU-Next/KernelSU-Next/releases)

> [!TIP]
> Always check the specific release notes for each version, as they may contain important update-specific instructions or prerequisites.

---

## 🌟 Special Thanks

This project stands on the shoulders of giants. Immense gratitude to the following developers and projects:

<div align="center">

| Project | Developer | GitHub / Profile |
| :--- | :--- | :--- |
| **KernelSU** | **tiann** | [![GitHub](https://img.shields.io/badge/GitHub-blue?style=flat-square&logo=github)](https://github.com/tiann) |
| **KernelSU-Next** | **rifsxd** | [![GitHub](https://img.shields.io/badge/GitHub-blue?style=flat-square&logo=github)](https://github.com/rifsxd) |
| **SUSFS** | **simonpunk** | [![GitLab](https://img.shields.io/badge/GitLab-orange?style=flat-square&logo=gitlab)](https://gitlab.com/simonpunk) |
| **WildKernels** | **fatalcoder524** | [![GitHub](https://img.shields.io/badge/GitHub-blue?style=flat-square&logo=github)](https://github.com/WildKernels) |
| **SUSFS Module** | **sidex15** | [![GitHub](https://img.shields.io/badge/GitHub-blue?style=flat-square&logo=github)](https://github.com/sidex15) |
| **Baseband Guard** | **vc-teahouse** | [![GitHub](https://img.shields.io/badge/GitHub-blue?style=flat-square&logo=github)](https://github.com/vc-teahouse/Baseband-guard.git) |
| **Droidspaces** | **ravindu644** | [![GitHub](https://img.shields.io/badge/GitHub-blue?style=flat-square&logo=github)](https://github.com/ravindu644/Droidspaces-OSS.git) |

</div>

*If you’ve contributed and aren't listed, please reach out so I can add you!* 🤝

---

## 💬 Support

If you need assistance or want to report a bug:
*   🚀 **GitHub Issues:** [Open a new issue](../../issues)
*   💭 **Discussions:** Feel free to join the conversation in our community channels.

---

<div align="center">
  <sub>Built with passion for the OnePlus community.</sub>
</div>

