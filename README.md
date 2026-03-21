<div align="center">

# 🌟 nikitos4683 Kernel for OnePlus 13

**A premium custom kernel experience focused on security, performance, and features.**

[![KernelSU](https://img.shields.io/badge/KernelSU-Supported-1a7f37?style=for-the-badge&logo=android&logoColor=white)](https://kernelsu.org/)
[![KernelSU-Next](https://img.shields.io/badge/KSU--Next-Integrated-1a7f37?style=for-the-badge&logo=github&logoColor=white)](https://kernelsu-next.github.io/webpage/)
[![SUSFS](https://img.shields.io/badge/SUSFS-Enhanced-d97706?style=for-the-badge&logo=gitlab&logoColor=white)](https://gitlab.com/simonpunk/susfs4ksu)

---

### 🚀 Powered by [WildKernels](https://github.com/WildKernels) | Branch: OOS16

> This repository is a boutique, nikitos4683-branded kernel fork tailored for the **OnePlus 13 (OP13)** running **OxygenOS 16**. It combines the power of modern kernel development with cutting-edge security extensions.

</div>

---

## 📋 Table of Contents
- [⚠️ Disclaimer](#-disclaimer)
- [🔗 Additional Resources](#-additional-resources)
- [✨ Features](#-features)
- [📝 Installation](#-installation)
- [🌟 Special Thanks](#-special-thanks)
- [💬 Support](#-support)

---

## ⚠️ Disclaimer

Flashing a custom kernel is a choice you make for your device. While we strive for stability, there is always a inherent risk.

*   **Backup:** Always keep a fresh backup of your data.
*   **Knowledge:** Read the documentations and understand the features before flashing.
*   **Responsibility:** I am **not responsible** for bricked devices, damaged hardware, or any issues arising from the use of this kernel.
*   **Your Choice:** By flashing this, you accept full responsibility for any outcomes.

---

## 🔗 Additional Resources

| Resource | Description |
| :--- | :--- |
| 🩹 [Kernel Patches](https://github.com/WildKernels/kernel_patches) | Core patches used in the build process |
| ⚡ [Kernel Flasher](https://github.com/fatalcoder524/KernelFlasher) | Recommended tool for safe flashing |
| 📱 [Compatibility Info](./compatibility.md) | Verify supported base versions |

---

## ✨ Features

Our build configurations are meticulously tuned for the best experience on the OnePlus 13.

### 💎 Core Integrations
*   **KernelSU / KernelSU-Next:** First-class support for both. Defaulting to `KSUN` (dev branch) for the latest security features.
*   **SUSFS Support:** Built-in hooks for advanced system-user-space filesystem isolation.
*   **OxygenOS 16 Optimized:** specifically targeted at `OP13` with Android 15 & Kernel 6.6.

### 🛠️ Enabled Features
| Feature | Description |
| :--- | :--- |
| ✅ **HMBIRD SCX** | Advanced scheduler extensions |
| ✅ **BBRv1** | Google's TCP congestion control for better network responsiveness |
| ✅ **TTL Target** | Native support for TTL modification |
| ✅ **WireGuard** | High-performance, secure VPN kernel support |
| ✅ **IP Set & IPv6 NAT** | Enhanced networking and firewall capabilities |
| ✅ **Thin LTO** | Link-Time Optimization for a balanced build performance |
| ✅ **TMPFS XATTR** | POSIX ACL Support for enhanced filesystem security |
| ✅ **Custom Branding** | Localversion set to `nikitos4683` |

### 🚫 Disabled / Experimental
*   **Baseband Guard (BBG):** Currently inactive.
*   **Unicode Bypass Fix:** Disabled in the current stable branch.
*   **Rust Path:** Bindgen build path is currently disabled.

---

## 📝 Installation Instructions

For a safe and successful installation, please follow the official GKI guidelines:

👉 **[KernelSU Official Installation Guide](https://kernelsu.org/guide/installation.html)**

> [!TIP]
> Always check the specific release notes for each version, as they may contain important update-specific instructions.

---

## 🌟 Special Thanks

This project stands on the shoulders of giants. immense gratitude to the following developers and projects:

<div align="center">

| Project | Developer | GitHub / Profile |
| :--- | :--- | :--- |
| **KernelSU** | **tiann** | [![GitHub](https://img.shields.io/badge/GitHub-blue?style=flat-square&logo=github)](https://github.com/tiann) |
| **KernelSU-Next** | **rifsxd** | [![GitHub](https://img.shields.io/badge/GitHub-blue?style=flat-square&logo=github)](https://github.com/rifsxd) |
| **SUSFS** | **simonpunk** | [![GitLab](https://img.shields.io/badge/GitLab-orange?style=flat-square&logo=gitlab)](https://gitlab.com/simonpunk) |
| **WildKernels** | **fatalcoder524** | [![GitHub](https://img.shields.io/badge/GitHub-blue?style=flat-square&logo=github)](https://github.com/WildKernels) |
| **SUSFS Module** | **sidex15** | [![GitHub](https://img.shields.io/badge/GitHub-blue?style=flat-square&logo=github)](https://github.com/sidex15) |

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

