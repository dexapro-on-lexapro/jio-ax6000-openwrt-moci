# JioRouter AX6000 OpenWrt with MoCI - Release v2026.11.24-moci

Production-grade OpenWrt firmware release for the **JioRouter AX6000** family featuring the **MoCI (Modern Configuration Interface)** web administration console.

This release covers both the **Qualcomm IPQ9554** platform (Unified JIDU6J11 Series) and the **MediaTek MT7986A Filogic 830** platform (JIDU6101 and JIDU6J01).

---

## What is MoCI?

MoCI (Modern Configuration Interface: https://github.com/HudsonGraeme/moci) is a lightweight, ultra-responsive replacement for OpenWrt's traditional LuCI web interface:

* **Zero Lua/CGI Overhead:** Built entirely as a vanilla JavaScript Single-Page Application (SPA).
* **Direct ubus JSON-RPC:** Speaks directly to OpenWrt's native `/ubus` RPC endpoint via `rpcd` and `uhttpd-mod-ubus`.
* **Sub-Millisecond Responsiveness:** Instantaneous page switches and real-time dashboard telemetry.
* **Minimal Flash & Memory Footprint:** Occupies under 80 KB on flash, leaving maximum RAM and storage for routing and packet buffers.
* **Seamless Root Redirection:** Visiting `http://192.168.1.1/` automatically redirects to `http://192.168.1.1/moci/`.

---

## Qualcomm JIDU6J11 Single Unified Image (Non-Redundant)

The Qualcomm JioRouter AX6000 series comprises several marketing SKU numbers:
* **JIDU6111**
* **JIDU6411**
* **JIDU6611**
* **JIDU6711**
* **JIDU6811**
* **JIDU6911**

**All 6J11 models share 100% identical underlying hardware:**
* Qualcomm IPQ9554 Quad-Core ARM Cortex-A53 @ 1.5 GHz
* IPQ9574 integrated 4x4 2.4 GHz 802.11ax radio (`wifi0`)
* QCN6274 discrete 4x4 5 GHz 802.11ax radio (`wifi1`)
* 512 MiB DDR3L RAM
* 128 MiB SPI-NAND flash with identical dual rootfs partition layouts
* 2.5 GbE WAN + Gigabit LAN switch architecture

Rather than producing redundant, identical images for each SKU, this release provides a **single, unified image set (`jiorouter_ax6000-jidu6j11`)**. Multi-device tree compatibility and alternative device aliases (`DEVICE_ALT0` through `DEVICE_ALT4`) are embedded directly into the firmware image metadata. It validates and flashes seamlessly across ANY 6J11 router model without requiring `--force`.

---

## Included Features & Kernel Enhancements

* **Linux Kernel:** 6.18 LTS with GCC 14.4.0 and musl libc.
* **Active Queue Management (Bufferbloat Elimination):** Pre-compiled `kmod-sched-cake` (CAKE qdisc), `kmod-ifb` (Intermediate Functional Block pseudo-device for bidirectional traffic shaping), `kmod-sched-core`, and `sqm-scripts`.
* **Full USB Cellular & Tethering Suite:** Complete drivers for Android USB tethering (`rndis`), iOS hotspot (`ipheth`), CDC Ethernet (`cdc_ether`, `cdc_ncm`, `cdc_mbim`), and LTE/5G modems (`qmi_wwan`, `huawei_cdc_ncm`, `option`). Dynamic hotplug auto-configures connected modems directly into the WAN zone.
* **Dynamic Internet Status LED Daemon:** Background health monitor verifying gateway and DNS reachability, driving the hardware status LED (Green: Online, Red: Offline, Red Blinking: Booting).
* **USB Storage & Media Sharing:** Out-of-the-box support for ext4, vfat, ntfs3, exfat, Samba 4 server, and MiniDLNA streaming without vermagic mismatches.
* **VPN & Secure Mesh:** WireGuard (`kmod-wireguard`, `wireguard-tools`) and Tailscale pre-installed.
* **Qualcomm 2.4GHz Wi-Fi Fix:** Explicit `&q6_region { status = "okay"; };` memory node assignment in the Device Tree prevents remoteproc allocation failures (`error -22`), keeping the integrated IPQ9574 AHB 2.4GHz radio (`wifi0`) fully active.
* **ath11k Optimization Suite:** Merged regulatory tables unlocking 30 dBm transmit power, smallbuffers memory profile (mode 2) saving 15-30 MB RAM, and reduced RXDMA ring buffers preventing OOM kernel panics.
* **MediaTek Filogic Offloading:** Hardware flow offloading (WED/WO) enabled on MediaTek MT7986A via `mt76`.

---

## Supported Artifacts

### Qualcomm IPQ9554 (Unified JIDU6J11 Series)
* `openwrt-qualcommbe-ipq95xx-jiorouter_ax6000-jidu6j11-squashfs-sysupgrade.bin`: Permanent sysupgrade firmware for existing OpenWrt installations across all 6J11 models.
* `openwrt-qualcommbe-ipq95xx-jiorouter_ax6000-jidu6j11-initramfs-factory.ubi`: Custom 70 MiB kernel volume UBI image for zero-UART stock SSH injection flashing.
* `openwrt-qualcommbe-ipq95xx-jiorouter_ax6000-jidu6j11-initramfs-uImage.itb`: RAM-bootable recovery FIT image.

### MediaTek MT7986A Filogic 830
* **JIDU6101:**
  * `openwrt-mediatek-filogic-jiorouter_ax6000-jidu6101-squashfs-sysupgrade.bin`
  * `openwrt-mediatek-filogic-jiorouter_ax6000-jidu6101-initramfs-kernel.bin`
* **JIDU6J01:**
  * `openwrt-mediatek-filogic-jiorouter_ax6000-jidu6j01-squashfs-sysupgrade.bin`
  * `openwrt-mediatek-filogic-jiorouter_ax6000-jidu6j01-initramfs-kernel.bin`

---

## Verification Checksums (SHA-256)

```text
e492418ebd6223e9f3571186049785ed68aa2b400850c889c092b9b702886216  openwrt-mediatek-filogic-jiorouter_ax6000-jidu6101-initramfs-kernel.bin
09b37d1454b6785d6f93722a89d6586ad95705dd2bdd0264b19e5fad1de90795  openwrt-mediatek-filogic-jiorouter_ax6000-jidu6101-squashfs-sysupgrade.bin
9f057327cd98473e9cf5249697359ca034d17a2434e5817ba04e76182960f5df  openwrt-mediatek-filogic-jiorouter_ax6000-jidu6j01-initramfs-kernel.bin
23e691abfc35e24d46fb19e265389c30c09cb8a89c9dfc78bb138a0d8a8829ba  openwrt-mediatek-filogic-jiorouter_ax6000-jidu6j01-squashfs-sysupgrade.bin
810a87092e304f228ac32647d8897b976b3a59d41420b609de475019241b3000  openwrt-qualcommbe-ipq95xx-jiorouter_ax6000-jidu6j11-initramfs-factory.ubi
e527b15a715fb85c808effdd4ec51b43ab7ed1e5550e54746f8085f9f962f6cb  openwrt-qualcommbe-ipq95xx-jiorouter_ax6000-jidu6j11-initramfs-uImage.itb
d4a82f06ecd06dccaea2be107c21025515d36d8738b3940d3c9c62edcf45a562  openwrt-qualcommbe-ipq95xx-jiorouter_ax6000-jidu6j11-squashfs-factory.ubi
ec128f3309141c9979c1d7a98b3642e9a56aeb3335669c86bc050ef272939799  openwrt-qualcommbe-ipq95xx-jiorouter_ax6000-jidu6j11-squashfs-sysupgrade.bin
```

---

## Credits & Acknowledgments

* **Graeme Hudson (@HudsonGraeme)** - Creator and lead developer of MoCI (Modern Configuration Interface for OpenWrt).
* **OpenWrt Project** - The open-source embedded operating system foundation and toolchain.
* **Sandeep Sondagar (@Sandiep)** - Hardware reverse engineering, initial device tree authorship, and smallbuffers memory research for the Qualcomm IPQ9554 platform.
* **Aditya Bakare (@adityabakare)** - Root-causing and fixing the 2.4GHz internal WCSS PIL reservation regression in Linux kernel 6.18 (`&q6_region { status = "okay"; };`).
* **Sahil Bakshi (@SahilB69)** - Snapshot build maintenance, `openwrt/iwinfo#51` IPQ9574 support, first-boot Wi-Fi activation scripts, and ath11k driver patch curation.
* **Mayank Raut (@themayankraut)** - Hardware teardown documentation, board analysis, and SKU cross-referencing.
* **Ifran Ali (@AliLostInTheDark), AK Sharma (@AKoo7), and Sheikh Faisal (@sh3ikh-faisal)** - Device tree contributions and multi-SKU platform support across the JioRouter hardware line.
* **Robert Marko** - Upstream development and architecture of the ath11k firmware memory mode subsystem in the Linux kernel.
* **Michael Phan** - Upstream foundation for ath11k socket allocation and DMA memory leak mitigations.
