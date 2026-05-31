# Realtek RTL930x / JT-COM JT-S508CL-8S Factory Firmware Dump

A collection of bit-for-bit physical SPI flash partition backups extracted from a **JT-COM JT-S508CL-8S** (an 8-Port 10G SFP+ managed switch running on the Realtek RTL9303 platform). 

These dumps were cleanly carved out over the network via raw read-only Memory Technology Device (`mtdXro`) interfaces using an OpenWrt initramfs recovery environment.


## 🛠️ Hardware Profile & Device Aliases
The hardware in this repository is highly cross-compatible and distributed under several white-label and commercial brand names. These files are highly relevant to:
* **JT-COM** JT-S508CL-8S
* **XikeStor** SKS8300-8X
* **ONTi** ONT-S508CL-8S
* **Hicomdata** HK-ES3058-8P-L
* **Core SoC:** Realtek RTL9303 / RTL930x Architecture
* **Flash Memory:** 32MB Winbond SPI Flash

---

## 📦 Repository Contents

| File Name | Memory Size (Hex) | Size (Decimal) | Description & Use Case |
| :--- | :--- | :--- | :--- |
| **`mtd0ro.u-boot.bin`** | `0x00100000` | **1 MiB** | **The Bootloader.** Critical fallback for physical unbricking via an external SPI chip programmer (e.g., CH341A clip). |
| **`mtd3ro.firmware.bin`** | `0x01e00000` | **30 MiB** | **Monolithic Factory Firmware.** The entire original runtime image ecosystem. Ideal for a total U-Boot factory restoration. |
| **`mtd4ro.kernel.bin`** | `0x00800000` | **8 MiB** | **OEM Linux Kernel.** Extracted sub-section of `mtd3` holding the factory-compiled kernel space. |
| **`mtd5ro.rootfs.bin`** | `0x01600000` | **22 MiB** | **The Root Filesystem.** Holds the factory web UI assets, configurations, and proprietary SDK binaries (`imi`, `dcli`). |

*⚠️ **Privacy & Hardware Safety Note:** `mtd1ro.board-info` (unique MAC addresses and factory SFP+ laser calibrations) and `mtd2ro.syslog` (internal runtime log environment maps) have been intentionally omitted to protect individual machine identity and prevent network hardware conflicts.*

---

## 🔍 Flash Geometry Layout

The underlying layout mapped out by `/proc/mtd`:

```text
dev:    size     erasesize  name
mtd0: 00100000   00010000   "u-boot"
mtd1: 00030000   00010000   "board-info"
mtd2: 000d0000   00010000   "syslog"
mtd3: 01e00000   00010000   "firmware"
mtd4: 00800000   00010000   "kernel"
mtd5: 01600000   00010000   "rootfs"
```

---

## 🪵 Boot Log Analysis (`boot.txt`)

The included `boot.txt` capture contains the complete serial console output from the switch’s physical UART headers during a cold boot sequence. Analysing this log reveals several critical design choices implemented by the OEM:

### 1. Bootloader Environment
* **U-Boot Version:** `V2.00`
* **Storage Execution:** The system initialises a **Winbond** SPI flash chip and immediately searches for a valid primary partition.
* **Interrupt Hook:** The bootloader listens for a `Ctrl+B` escape sequence on boot. This key combination interrupts the normal boot sequence and drops the serial operator into a low-level Realtek recovery menu, allowing manual TFTP flash writes if the operating system is corrupted.

### 2. Filesystem & Image Structure
* **JFFS2 Compression:** The factory firmware container uses a compressed JFFS2 (Journalling Flash File System version 2) layer.
* **Memory Offset:** The compressed kernel image is loaded from the flash chip into system RAM at boundary address `0x81000000`. The image uses a standard **uImage Legacy format wrapper** starting at memory address `0x81000100`.

### 3. Software Versioning & Daemons
* **Build Date:** The factory software compilation date is hardcoded to `2025-03-12`.
* **Software Baseline:** Running version `V300SP10250312`.
* **Management Hooks:** On final initialisation, the Linux kernel sets up basic structural tasks (VLAN orchestration, MAC alignment) and spawns an unencrypted **HTTP management server on Port 80** to serve the stock web control panel.

---

## 🤝 Call for Contributions (Newer Firmware Dumps)

If you own a **JT-COM JT-S508CL-8S**, **XikeStor SKS8300-8X**, or an equivalent Realtek RTL9303 clone, and your hardware is running a newer system software baseline than **`V300SP10250312`** (Compile Date: `2025-03-12`), your contributions are highly welcome!

### How to Contribute:

1. Capture the serial boot log.
2. Boot your switch into an OpenWrt initramfs state.
3. Back up your firmware partitions to your local machine using the read-only MTD blocks:
   ```bash
   ssh root@192.168.1.1 "cat /dev/mtd0ro" > mtd0ro.u-boot.bin
   ssh root@192.168.1.1 "cat /dev/mtd1ro" > mtd1ro.board-info.bin
   ssh root@192.168.1.1 "cat /dev/mtd2ro" > mtd2ro.syslog.bin
   ssh root@192.168.1.1 "cat /dev/mtd3ro" > mtd3ro.firmware.bin
   ssh root@192.168.1.1 "cat /dev/mtd4ro" > mtd4ro.kernel.bin
   ssh root@192.168.1.1 "cat /dev/mtd5ro" > mtd5ro.rootfs.bin
   ```

4. Open a Pull Request or submit a new Issue tracking your hardware's boot.txt log output and firmware binaries.

Please ensure you omit mtd1ro.board-info.bin and mtd2ro.syslog.bin to protect your network's unique MAC address structures and local device configuration identifiers.

---

## 📚 Further Resources & Upstream Links

For deployment guides, operational documentation, and official firmware build profiles related to this platform, consult the following resources:

* **[Official OpenWrt Table of Hardware (ToH) Profile](https://openwrt.org/toh/xikestor/sks8300-8x):** The definitive upstream wiki page for the XikeStor SKS8300-8X and identical RTL9303 clones. Contains structural flashing guides, recovery strategies, and links to stable/snapshot OpenWrt system images.
* **[OEM Web Interface Operation Manual (PDF)](https://tomocha.net/files/XikeStor_SKS8300-8X/Web%20Interface%20Manual%20of%20SKS8300-8X%20&%20SKS8300-12X.pdf):** The original manufacturer documentation detailing the stock firmware feature set, internal switch configuration patterns, and web GUI administration maps for the SKS8300-8X & SKS8300-12X ecosystem.
* **[OpenWrt Realtek RTL930x Target Source](https://github.com/openwrt/openwrt):** Track the ongoing source-code implementation and ASIC driving layers for the underlying Realtek architecture.
