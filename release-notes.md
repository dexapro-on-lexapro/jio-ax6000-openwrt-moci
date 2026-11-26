# JioRouter AX6000 OpenWrt with MoCI - Release v2026.11.26-moci

Production-grade OpenWrt firmware release for the **JioRouter AX6000** family featuring the **MoCI (Modern Configuration Interface)** web administration console.

This release covers both the **Qualcomm IPQ9554** platform (Unified JIDU6J11 Series) and the **MediaTek MT7986A Filogic 830** platform (JIDU6101 and JIDU6J01 Series), resolving cross-maintainer sysupgrade compatibility issues across all SKU naming schemes.

---

## What Changed in v2026.11.26-moci

* **Cross-Maintainer Sysupgrade Compatibility (Qualcomm J11 & MediaTek J01):**
  * **Qualcomm `jiorouter_ax6000-jidu6j11`:** Added `SUPPORTED_DEVICES`, DTS `compatible` fallbacks, and non-directory sysupgrade tar `CONTROL` aliases for all 13 board strings (`jiorouter,ax6000-jidu6j11`, upstream PR #26498/#26551 `jiorouter,ax6000-jidu6{111,411,611,711,811,911}`, and legacy `jiorouter,ax6000-jidu6j11-{6111,6411,6611,6711,6811,6911}`), plus full `DEVICE_ALT0..5` metadata for all 6 J11 SKUs (including `JIDU6711`).
  * **MediaTek `jiorouter_ax6000-jidu6j01`:** Added `SUPPORTED_DEVICES`, DTS `compatible` fallbacks, and sysupgrade tar `CONTROL` aliases for `jiorouter,ax6000-jidu6{201,401,601,701,801}` alongside `jiorouter,ax6000-jidu6j01` (keeping `jidu6101` separate due to its distinct switch port layout and `u-boot-env` MAC mapping).
* **Unsigned `fwtool` Metadata (`# CONFIG_SIGN_FIRMWARE is not set`):** Explicitly disabled local build key firmware signing so sysupgrade images carry clean `fwtool` JSON metadata without untrusted local `ucert` signatures when flashing from other maintainers' builds.
* **Fixed Qualcomm Unified Base-Files Scripts:** Updated `/lib/upgrade/platform.sh`, `/etc/board.d/02_network`, `/etc/hotplug.d/firmware/11-ath11k-caldata`, and `uboot-envtools` (`qualcommbe_ipq95xx`) to match the unified `jiorouter,ax6000-jidu6j11` board name and all per-SKU J11 aliases (and accept cross-maintainer Jio sysupgrade tarballs cleanly in `platform_check_image`).
* **MediaTek `filogic` Alias Resilience:** Updated `02_network`, `11_fix_wifi_mac`, `platform.sh`, and `mediatek_filogic` to accept all J01 SKU aliases (`6201/6401/6601/6701/6801`).

---

## Supported Artifacts

### Qualcomm IPQ9554 (Single Unified JIDU6J11 Series Image — Covers 6111, 6411, 6611, 6711, 6811, 6911)
* `openwrt-qualcommbe-ipq95xx-jiorouter_ax6000-jidu6j11-squashfs-sysupgrade.bin`: Permanent sysupgrade firmware for existing OpenWrt installations across all 6 J11 models (supports direct GUI/CLI upgrades from community builds without `--force`).
* `openwrt-qualcommbe-ipq95xx-jiorouter_ax6000-jidu6j11-initramfs-factory.ubi`: Custom 70 MiB kernel volume UBI image for zero-UART stock SSH injection flashing.
* `openwrt-qualcommbe-ipq95xx-jiorouter_ax6000-jidu6j11-initramfs-uImage.itb`: RAM-bootable recovery FIT image.
* `openwrt-qualcommbe-ipq95xx-jiorouter_ax6000-jidu6j11-squashfs-factory.ubi`: Direct NAND factory UBI image.

### MediaTek MT7986A Filogic 830
* **JIDU6101:**
  * `openwrt-mediatek-filogic-jiorouter_ax6000-jidu6101-squashfs-sysupgrade.bin`
  * `openwrt-mediatek-filogic-jiorouter_ax6000-jidu6101-initramfs-kernel.bin`
* **JIDU6J01 *(6201, 6401, 6601, 6701, 6801)*:**
  * `openwrt-mediatek-filogic-jiorouter_ax6000-jidu6j01-squashfs-sysupgrade.bin`
  * `openwrt-mediatek-filogic-jiorouter_ax6000-jidu6j01-initramfs-kernel.bin`

---

## Verification Checksums (SHA-256)

```text
03e3b842d0826e0df3c31b8c789102bb9a44f1359a501e5b727224ba28157736  openwrt-mediatek-filogic-jiorouter_ax6000-jidu6101-initramfs-kernel.bin
e7d81cbd4d262cbce1d2b53ed3571160a0b937e5996637cf80c5ba9730123674  openwrt-mediatek-filogic-jiorouter_ax6000-jidu6101-squashfs-sysupgrade.bin
fcdd14b2631bcc44d7d50cbf9d93e29d94a73d59e4f98a4912a907f5d9fac59e  openwrt-mediatek-filogic-jiorouter_ax6000-jidu6j01-initramfs-kernel.bin
29672bf7ac7ec4bb36ef412972b6671c8363337b5f1fb7624b6c026daeaea235  openwrt-mediatek-filogic-jiorouter_ax6000-jidu6j01-squashfs-sysupgrade.bin
6942f7695778169a683070fac07c7eb7e72111eba0318c9c1f422692e314b57f  openwrt-qualcommbe-ipq95xx-jiorouter_ax6000-jidu6j11-initramfs-factory.ubi
f66b068fe31bdb02094df6ddff99e3ee2a6cf914e61f45ee801ba0a07409a7ec  openwrt-qualcommbe-ipq95xx-jiorouter_ax6000-jidu6j11-initramfs-uImage.itb
0808b1e3ddfce63bcf10ed946ae85cee4a7ee6c359f61b0a9e395de5a707db98  openwrt-qualcommbe-ipq95xx-jiorouter_ax6000-jidu6j11-squashfs-factory.ubi
553766dc28da6fff174a6a60f0784e2856943c27241d28647dbb9da467a78da0  openwrt-qualcommbe-ipq95xx-jiorouter_ax6000-jidu6j11-squashfs-sysupgrade.bin
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
