# JioRouter AX6000 OpenWrt Firmware with MoCI (Qualcomm & MediaTek)

Production-grade OpenWrt firmware builds and integration suite for the **JioRouter AX6000** family featuring the **MoCI (Modern Configuration Interface)** web administration console. Supports both **Qualcomm IPQ9554** (Unified JIDU6J11 Series) and **MediaTek MT7986A Filogic 830** (JIDU6101 and JIDU6J01 Series).

---

## MoCI: Modern Configuration Interface

This build set replaces OpenWrt's traditional LuCI web interface with **MoCI** (Modern Configuration Interface: https://github.com/HudsonGraeme/moci).

### Key Architectural Advantages of MoCI:
* **Zero Compilation Overhead on Device:** Pure vanilla JavaScript Single-Page Application (SPA) delivered as pre-minified static assets.
* **Direct ubus JSON-RPC Engine:** MoCI interfaces directly with OpenWrt's native `/ubus` RPC endpoint via `rpcd` and `uhttpd-mod-ubus`. It completely eliminates the heavy Lua runtime, uc templates, and complex CGI fork-exec pipelines required by traditional LuCI.
* **Instantaneous UI Rendering:** Sub-millisecond page transitions, live statistics, real-time interface monitoring, and responsive layouts designed for modern mobile and desktop browsers.
* **Minimal Storage & Memory Footprint:** The entire MoCI package occupies under 80 KB on flash, freeing valuable RAM and storage for routing workloads, containerization, and packet buffers.
* **Seamless Web Root Redirection:** Requests to the router's root web address (`http://192.168.1.1/`) are seamlessly redirected to `/moci/` via an optimized HTTP refresh stub in `/www/index.html`.



## Technical Highlights & Included Components

This release is compiled against bleeding-edge OpenWrt `main` with **Linux Kernel 6.18**, modern **GCC 14.4.0**, and **musl libc**. Every component is built directly into the root filesystem (`=y`) to provide out-of-the-box functionality without external package repository dependencies.

### 1. Active Queue Management (SQM & Bufferbloat Mitigation)
* **CAKE Qdisc (`kmod-sched-cake`):** Comprehensive Active Queue Management designed specifically for consumer broadband and bufferbloat elimination.
* **Ingress Traffic Shaping (`kmod-ifb`):** Intermediate Functional Block pseudo-device support for bidirectional line-rate traffic shaping.
* **Kernel Schedulers (`kmod-sched-core`):** Core packet scheduling modules and `tc-tiny` for precise traffic control.
* **SQM Scripts (`sqm-scripts`):** Out-of-the-box support for cake and fq_codel shaping configurations.

### 2. Plug-and-Play USB Tethering & Cellular Dongle Suite
The firmware integrates the complete Linux USB networking and mobile broadband subsystem:
* **Host Protocols:** `kmod-usb-net-rndis` (Android USB tethering), `kmod-usb-net-cdc-ether`, `kmod-usb-net-cdc-ncm`, `kmod-usb-net-cdc-mbim`.
* **Cellular Modems:** `kmod-usb-net-qmi-wwan`, `kmod-usb-net-huawei-cdc-ncm`, `kmod-usb-serial-option`, `usb-modeswitch`, `uqmi`, `umbim`.
* **iOS Support:** `kmod-usb-net-ipheth` (Apple iPhone personal hotspot tethering).
* **USB Ethernet Adapters:** `kmod-usb-net-asix`, `kmod-usb-net-asix-ax88179`, `kmod-usb-net-rtl8150`, `kmod-usb-net-rtl8152`.
* **Dynamic Hotplug Handler:** `/etc/hotplug.d/net/99-usb-tethering` automatically identifies cellular tethering interfaces (`usb0`, `eth1`, `wwan0`, etc.) upon connection and brings them up within the WAN firewall zone.

### 3. Dynamic Internet Status LED Daemon
An automated background health monitor (`/usr/bin/led_internet_monitor.sh` managed by `procd` via `/etc/init.d/led_internet_monitor`):
* Automatically detects the platform architecture (Qualcomm IPQ9554 GPIO LEDs vs. MediaTek MT7986A Filogic GPIO LEDs).
* Continuously checks default gateway reachability and verifies external DNS connectivity:
  * **Solid Green**: Active WAN connection and internet reachability verified.
  * **Solid Red**: WAN disconnected or external internet gateway unreachable.
  * **Red Blinking**: System initialization and bootup phase.

### 4. Integrated USB Storage Subsystem
* `kmod-usb-storage`, `kmod-usb-storage-uas`, `kmod-fs-ext4`, `kmod-fs-vfat`, `kmod-fs-ntfs3`, `kmod-fs-exfat`, and `block-mount` are pre-compiled into the firmware.
* Full Samba 4 file sharing (`samba4-server`, `samba4-libs`) and MiniDLNA media streaming (`minidlna`, `ffmpeg`).
* External storage devices format and mount automatically on plug.

### 5. Memory Optimizations, Wi-Fi Stability & Bug Fixes
* **Qualcomm 2.4GHz WCSS Remoteproc Fix:** Explicit `&q6_region { status = "okay"; };` memory node assignment in the Device Tree prevents remoteproc allocation failures (`error -22`), keeping the integrated IPQ9574 AHB 2.4GHz radio (`wifi0`) fully active.
* **ath11k Wi-Fi Driver Optimization Suite:**
  * Merged regulatory domain tables unlock full 30 dBm transmit power and 160 MHz channel width on Band 2 (channels 36-64).
  * Shrunk RXDMA (2048) and monitor ring buffers (256) eliminate Out-Of-Memory kernel panics under sustained multi-gigabit workloads.
  * Dedicated page-frag caches for RXDMA resolve page exhaustion leaks without legacy kmalloc penalties.
  * Idempotent external IRQ control ensures clean Wi-Fi crash recovery and service restarts.
* **Qualcomm `ath11k` Mode-2 Memory Profile (`smallbuffers`):** Configured via Device Tree (`qcom,ath11k-fw-memory-mode = <2>`) for both radios (`wifi0` and `wifi1`), saving 15-30 MB of kernel RAM.
* **MediaTek Filogic Hardware Offloading:** Fully utilizes hardware packet offloading (WED/WO flow offloading) via MediaTek's `mt76` driver suite with 4x4 MU-MIMO and beamforming enabled.

### 6. Pre-installed Management & Network Services
* **Web Interface:** MoCI (`/moci/`) web interface powered by `uhttpd`, `uhttpd-mod-ubus`, `rpcd`, and `jsonfilter`.
* **VPN & Privacy:** Native WireGuard (`kmod-wireguard`, `wireguard-tools`), Tailscale (`tailscale`, `tailscaled`), and AdGuardHome (`adguardhome`).
* **Diagnostic & Monitoring Utilities:** `ttyd` web terminal, `btop`, `htop`, `iperf3`, `tcpdump`, `ethtool`, `nano`, `curl`, and `watchcat`.

---

## Supported Models & Image Sizing

Pre-compiled images are available on GitHub Releases:
* **[v2026.11.26-moci Release (Latest)](https://github.com/dexapro-on-lexapro/jio-ax6000-openwrt-moci/releases/tag/v2026.11.26-moci)** - Cross-maintainer `SUPPORTED_DEVICES` & sysupgrade `CONTROL` aliases across all J11 (`6111/6411/6611/6711/6811/6911`) and J01 (`6201/6401/6601/6701/6801`) variants, unsigned `fwtool` metadata (`# CONFIG_SIGN_FIRMWARE is not set`), and fixed Qualcomm base-files scripts (`02_network`, `11-ath11k-caldata`, `platform.sh`, `uboot-envtools`).
* **[v2026.11.24-moci Release](https://github.com/dexapro-on-lexapro/jio-ax6000-openwrt-moci/releases/tag/v2026.11.24-moci)** - Initial MoCI release with unified JIDU6J11 target.

| Platform Architecture | Hardware Model | SoC / CPU | Flash Layout | Required Firmware Image |
| :--- | :--- | :--- | :--- | :--- |
| **Qualcomm Networking Pro 400** | **JioRouter AX6000 JIDU6J11 Series**<br>*(JIDU6111, JIDU6411, JIDU6611, JIDU6711, JIDU6811, JIDU6911)* | Qualcomm IPQ9554<br>(4x A53 @ 1.5 GHz) | 128MB SPI-NAND<br>(Stock dual rootfs UBI) | `openwrt-qualcommbe-ipq95xx-jiorouter_ax6000-jidu6j11-squashfs-sysupgrade.bin`<br>`openwrt-qualcommbe-ipq95xx-jiorouter_ax6000-jidu6j11-initramfs-factory.ubi` |
| **MediaTek Filogic 830** | **JioRouter AX6000 JIDU6101** | MediaTek MT7986A<br>(4x A53 @ 2.0 GHz) | 128MB SPI-NAND<br>(Stock dual rootfs UBI) | `openwrt-mediatek-filogic-jiorouter_ax6000-jidu6101-squashfs-sysupgrade.bin`<br>`openwrt-mediatek-filogic-jiorouter_ax6000-jidu6101-initramfs-kernel.bin` |
| **MediaTek Filogic 830** | **JioRouter AX6000 JIDU6J01**<br>*(6201, 6401, 6601, 6701, 6801)* | MediaTek MT7986A<br>(4x A53 @ 2.0 GHz) | 128MB SPI-NAND<br>(Stock dual rootfs UBI) | `openwrt-mediatek-filogic-jiorouter_ax6000-jidu6j01-squashfs-sysupgrade.bin`<br>`openwrt-mediatek-filogic-jiorouter_ax6000-jidu6j01-initramfs-kernel.bin` |



## Repository Structure

```
├── .gitignore
├── LICENSE                                    # GNU General Public License v2.0
├── README.md                                  # Complete technical documentation
├── files/                                     # Root filesystem overlay tree
│   ├── etc/
│   │   ├── hotplug.d/net/99-usb-tethering     # Auto-configuration for USB tethered interfaces
│   │   ├── init.d/led_internet_monitor        # Procd init service for LED status daemon
│   │   └── uci-defaults/
│   │       ├── 90-enable-wifi                 # Automatic first-boot Wi-Fi activation
│   │       ├── 95-moci-setup                  # MoCI uhttpd home and ubus prefix configuration
│   │       └── 99-init-led-monitor            # First-boot automatic service enablement
│   ├── usr/bin/led_internet_monitor.sh        # Dynamic network status check and LED controller
│   └── www/
│       └── index.html                         # HTTP redirect stub to /moci/
├── package/
│   └── moci/                                  # MoCI package definition, Makefile & web assets
└── patches/
    ├── 0001-qualcommbe-and-mediatek-jio-support.patch # OpenWrt master git patch series
    └── target/linux/qualcommbe/dts/           # Reference Device Tree definitions
```

---

## Building From Source

To build this MoCI firmware from source using the official OpenWrt build system:

1. **Clone OpenWrt Master:**
   ```bash
   git clone https://github.com/openwrt/openwrt.git
   cd openwrt
   ```

2. **Apply the JioRouter AX6000 Patch Series:**
   ```bash
   git apply /path/to/jio-ax6000-openwrt-moci/patches/0001-qualcommbe-and-mediatek-jio-support.patch
   ```

3. **Copy the MoCI Package Definition:**
   ```bash
   cp -r /path/to/jio-ax6000-openwrt-moci/package/moci package/
   ```

4. **Install and Update Feeds:**
   ```bash
   ./scripts/feeds update -a
   ./scripts/feeds install -a
   ```

5. **Copy the Rootfs Overlay:**
   ```bash
   cp -r /path/to/jio-ax6000-openwrt-moci/files .
   ```

6. **Select the Device and MoCI in Menuconfig:**
   * Enable MoCI: `CONFIG_PACKAGE_moci=y`
   * Enable ubus HTTP module: `CONFIG_PACKAGE_uhttpd-mod-ubus=y`
   * Disable LuCI packages: `# CONFIG_PACKAGE_luci is not set`
   * Select your hardware target (`jiorouter_ax6000-jidu6j11`, `jiorouter_ax6000-jidu6101`, or `jiorouter_ax6000-jidu6j01`).

7. **Compile:**
   ```bash
   make -j$(nproc)
   ```

---

## AI Assistance & Transparency Disclosure

In compliance with open-source engineering transparency and AI attribution standards:
* **Tool:** Assisted by **Google Antigravity / Gemini Advanced Agentic Coding**.
* **Scope of Assistance:** Automated cross-compilation pipeline management, kernel 6.18 Device Tree syntax refactoring, multi-SKU compatibility de-duplication, dynamic LED state engine development, hotplug tethering scripting, and package curation.
* **Human Oversight:** Codebase review, verification, and maintainership led by [@dwaipayandex](https://github.com/dwaipayandex).

---

## Credits & Acknowledgments

This project is built on the research, reverse engineering, and collaborative contributions of the open-source networking community:

* **[Graeme Hudson (@HudsonGraeme)](https://github.com/HudsonGraeme)** - Creator and lead developer of [MoCI](https://github.com/HudsonGraeme/moci) (Modern Configuration Interface for OpenWrt). Author of the vanilla JavaScript SPA architecture and ubus RPC client engine.
* **[OpenWrt Project](https://openwrt.org/)** - Foundation of the open embedded Linux distribution, packaging system, and toolchain.
* **[@Sandiep](https://github.com/Sandiep) (Sandeep Sondagar)** - Initial hardware reverse engineering, bootloader analysis, device tree authorship, and smallbuffers memory research for the Qualcomm IPQ9554 (JIDU6J11) platform; co-author of the ath11k driver optimization series (idempotent IRQ handling, private page-frag DMA caches, regulatory domain merge, and RXDMA ring buffer reduction).
* **[@adityabakare](https://github.com/adityabakare) (Aditya Bakare)** - Root-causing and fixing the 2.4GHz internal WCSS PIL reservation regression in Linux kernel 6.18 (`&q6_region { status = "okay"; };`), restoring the integrated IPQ9574 Hexagon DSP remoteproc binding.
* **[@SahilB69](https://github.com/SahilB69) (Sahil Bakshi)** - Maintenance of community snapshot builds, authoring `openwrt/iwinfo#51` for IPQ9574 SoC detection, contributing first-boot Wi-Fi activation scripts, and co-curating the modern `ath11k` patch series.
* **[@themayankraut](https://github.com/themayankraut) (Mayank Raut)** - Hardware documentation, board teardown research, SKU cross-referencing, and community support for the Jio AX6000 hardware ecosystem.
* **[@AliLostInTheDark](https://github.com/AliLostInTheDark) (Ifran Ali), [@AKoo7](https://github.com/AKoo7) (AK Sharma), and [@sh3ikh-faisal](https://github.com/sh3ikh-faisal) (Sheikh Faisal)** - Collaborative device tree refactoring and multi-SKU platform support across the JioRouter hardware line (upstream OpenWrt PR #26551 and PR #26498).
* **Robert Marko** - Upstream development and architecture of the `ath11k` firmware memory mode subsystem in the Linux kernel.
* **Michael Phan** - Upstream foundation for `ath11k` socket allocation and DMA memory leak mitigations.

---

## License & Disclaimer

* **License:** This project is licensed under the [GNU General Public License v2.0 (GPL-2.0)](LICENSE) to maintain full compatibility with the OpenWrt and Linux kernel projects. MoCI web UI assets are licensed under the MIT License by Graeme Hudson.
* **Disclaimer:** Custom firmware modifications carry inherent risks. This repository provides source patches, overlays, and build artifacts for development, educational, and testing purposes only. No warranties or guarantees of fitness for a particular purpose are expressed or implied.
