# ASUS TUF Gaming F15 (FX507ZC4) — macOS Sequoia Hackintosh

<div align="center">

![macOS Version](https://img.shields.io/badge/macOS-Sequoia%2015.8.3-black?style=for-the-badge&logo=apple&logoColor=white)
![OpenCore](https://img.shields.io/badge/OpenCore-1.0.8%20DEBUG-blue?style=for-the-badge&logo=target)
![iGPU Status](https://img.shields.io/badge/Iris%20Xe%20(80EU)-Accelerated%20%40%20144Hz-success?style=for-the-badge&logo=intel)
![Project State](https://img.shields.io/badge/Status-WIP%20%2F%20Breakthrough-orange?style=for-the-badge)

<p align="center">
  <b>Experimental OpenCore EFI configuration enabling Full Hardware Acceleration (Metal 2) on Intel 12th Gen Alder Lake-H with 144Hz internal display.</b>
</p>

</div>

---

> [!WARNING]
> **EXPERIMENTAL RELEASE / WORK IN PROGRESS**  
> This setup achieves a previously "impossible" milestone: **full Metal 2 acceleration at 144Hz on an Alder Lake-H internal screen**. However, it is currently in wip, but ig i won't do smth with that, sr. Moving heavy vector UI windows (such as App Store) can cause a system hard-lock due to early GPU scheduler patches. Daily-driver stability is being worked on.

---

## 💻 Hardware Specifications

| Component | Detail |
| :--- | :--- |
| **Model** | ASUS TUF Gaming F15 (FX507ZC4) |
| **Processor** | Intel Core i5-12500H (12 Cores: 4P + 8E / 16 Threads, up to 4.5 GHz) |
| **Integrated GPU** | Intel Iris Xe Graphics (80 EU, Device ID `0x46A6`) |
| **Dedicated GPU** | NVIDIA GeForce RTX 3050 Ti Laptop *(Not disabled yet)* |
| **Internal Display** | 15.6" FHD (1920x1080) IPS, **144Hz Refresh Rate** (eDP) |
| **Target OS** | macOS Sequoia 15.8.3 |
| **SMBIOS** | `MacBookPro16,2` |

---

## 🚀 The Iris Xe (80 EU) Breakthrough

Previously, Alder Lake-H laptops were stuck in a **7MB FB-only safe mode** due to an eDP hand-off failure (`N97FramebufferStartAccepted = No`, `donor blocked`). 

### The Root Cause & Solution:
* **The Problem:** The macOS graphics driver expects the internal eDP port to report in **Mode 1** at hand-off. ASUS firmware leaves the combo port in **Mode 0**, resulting in driver rejection.
* **The Fix:** Utilizing a custom-compiled debug build of **WhateverXe** (forked by `@renedis` backporting `@Thatisjigen`'s Mode 0 admittance PR #38).
* **The Result:** The kernel accepts the eDP boot port in Mode 0, establishing donor parameters and activating the **AppleIntelXeGraphics** stack with **1536 MB VRAM and native 144Hz support**.

---

## 📊 Feature Status

### ✅ Working
- **Full Graphics Acceleration:** Metal 2, Quartz Extreme, transparent UI & blur effects.
- **Display Refresh Rate:** Native **144Hz** detected and smoothly running.
- **CPU Management:** P-Core & E-Core topology recognized.
- **Audio:** Realtek Audio via AppleALC.
- **Keyboard:** Just working keyboard, nothing more, fn?? did not test. uh, do not have asussmc.kext, so ig no, just add it and it should work

### ⚠️ In Progress / Known Issues
- **Window Movement Hard-Lock:** Dragging complex Metal-rendered windows (e.g. App Store) can cause a GPU scheduler crash / kernel freeze.
- **Sleep / Wake:** Currently disabled or unstable due to memory payload signature differences between macOS Tahoe and Sequoia 15.8.3. \ untested btw
- **AppleIntelTGLCompat:** Must be disabled or handled carefully to avoid early boot AMFI panics.
- touchpad?? idk why it does not work, so lazy to work with it..

---

## ⚙️ BIOS Configuration

Ensure the following toggles are set in your ASUS BIOS before booting:

| Setting | State | Note |
| :--- | :--- | :--- |
| **Fast Boot** | `Disabled` | Prevents skip of critical GOP initialization |
| **Secure Boot** | `Disabled` | Mandatory for OpenCore |
| **VTD (VT-d)** | `Disabled` | idk y when i switch it on the opencore freezes at watchdog
| **Hyper-Threading** | `Enabled` | Fixed EXIT.BS:START by adding 80,FF to Kernel-Emulate- Cpuid1Data, Cpuid1Mask|

---

## 🔑 Post-Installation: Generate Your Own SMBIOS

To ensure iCloud, iMessage, and App Store work (and to prevent Apple ID blacklisting), you **must** generate your own unique serials.

1. Download [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS).
2. Run GenSMBIOS and select option `Generate SMBIOS`.
3. Enter model identifier: `MacBookPro16,2`.
4. Open `EFI/OC/config.plist` with **ProperTree** and navigate to `PlatformInfo -> Generic`.
5. Populate the following keys:
   - `SystemProductName` : `MacBookPro16,2`
   - `SystemSerialNumber` : *(Your generated Serial)*
   - `MLB` : *(Your generated Board Serial)*
   - `SystemUUID` : *(Your generated SmUUID)*
   - `ROM` : *(Your generated ROM)*
6. Save your `config.plist`.

---

## 🧩 Boot-Args

```
-v debug=0x100 keepsyms=1 -allow3d -disablegfxfirmware -igfxdvmt -vi2c-force-polling ctrsmt=full -wexdbg -wex-display13-trace
```

```
-allow3d : Forces Metal/3D acceleration activation.
-disablegfxfirmware : Disables native Apple graphics driver.

(-v, debug=0x100, keepsyms=1, -wexdbg, -wex-display13-trace are made for debug, so not nesseccary ig)
```


---

## 👏 Credits & Acknowledgments

- [Acidanthera](https://github.com/acidanthera) — For OpenCore, Lilu, VirtualSMC, and essential kernel documentation.
- [b00t0x](https://github.com/b00t0x/WhateverXe) — For creating the groundbreaking WhateverXe project.
- [@renedis](https://github.com/renedis) — For the Mode 0 hand-off porting and display wake research branch.
- [@Thatisjigen](https://github.com/Thatisjigen) — For the original PR #38 patch admitting Mode 0 eDP ports.

- The entire **r/hackintosh** community for endless testing and reverse engineering.

---

```
btw, readme made by ai (googleaistudio) :3 
