# HP Pavilion Gaming Laptop 15-DK0XXX - Hackintosh EFI

This repository contains the OpenCore EFI configuration required to run latest macOS natively on the HP Pavilion Gaming Laptop 15-DK0XXX. The setup focuses on system stability, maintaining maximum security (SIP Enabled), and avoiding intrusive root patches (no OCLP required for core functionality).

## System Specifications

- **Laptop Model:** HP Pavilion Gaming Laptop 15-DK0XXX (HM370 Chipset)
- **Processor:** Intel Core i7-9750H (6-Core, Coffee Lake)
- **Integrated Graphics:** Intel UHD Graphics 630 (1920x1080 Internal Display)
- **Discrete Graphics:** NVIDIA GeForce GTX 1650 (Unsupported in macOS - Disabled via boot-args `wegnoegpu`)
- **Storage:** KIOXIA BG3 NVMe SSD
- **Ethernet:** Realtek RTL8168 Gigabit Ethernet
- **Target OS:** macOS Tahoe (Version 26)
- **Bootloader:** OpenCore v1.0.7
- **SMBIOS:** `MacBookPro16,1`

## Working Features & Configurations

### Network & Connectivity (Upgraded to Intel AX210)
The stock Wi-Fi card was physically replaced with an **Intel AX210NGW** module.

- **Wi-Fi:** Fully functional using the `itlwm.kext` driver. Requires the **HeliPort** application running on macOS to select and connect to wireless networks.
- **Bluetooth:** Native Bluetooth functionality is fully operational.
  - Handled by the `OpenIntelWireless` suite (`IntelBluetoothFirmware.kext`, `IntelBTPatcher.kext`, and `BlueToolFixup.kext`).
  - **Tahoe Compatibility Note:** To bypass Apple's strict Bluetooth firmware checks in macOS Tahoe, two critical variables are natively injected into the `7C436110-AB2A-4BBB-A880-FE41995C9F82` NVRAM namespace: `bluetoothExternalDongleFailed` (Data: `00`) and `bluetoothInternalControllerInfo` (Data: `00000000 00000000 00000000 0000`). Additionally, the `-ibtcompatbeta` boot argument is used to ensure proper initialization of the Intel Bluetooth controller.

### USB Mapping
- Internal and external USB ports have been strictly mapped using `UTBMap.kext` to ensure correct power delivery, sleep functioning, and internal Bluetooth hardware discovery.

### Audio (Design Decision)
- **Internal Speakers/Mic (ALC285):** Deliberately bypassed. macOS Tahoe completely removed `AppleHDA`, breaking `AppleALC`. While third-party workarounds like `VoodooHDA` exist, they severely compromise system security by requiring the disabling of System Integrity Protection (SIP) to bind to the hardware. 
- **Recommendation:** Use a USB Audio Card or Bluetooth speakers/headphones. This preserves system stability and maintains a 100% physically secure OS environment.

### Deep System Stability (ACPI)
- **IRQ Patches:** This EFI utilizes `SSDT-HPET.aml` alongside OpenCore ACPI renames (`_STA to XSTA` and `_CRS to XCRS`). This is a critical architectural patch to resolve inherent HP Pavilion hardware interrupt conflicts, ensuring smooth sleep/wake cycles and power management.

### Security
- **System Integrity Protection (SIP):** Fully ENABLED (`csr-active-config` = `00000000`).
- The system is engineered to run as close to real Mac hardware dependencies as possible.

## Post-Installation & Best Practices
- **iServices (FaceTime, iMessage):** The `config.plist` requires you to generate your own unique SMBIOS strings (`SystemSerialNumber`, `SystemUUID`, `MLB`, and `ROM`) using [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS). Do not log into your Apple ID until you generate them!
- **HeliPort:** Remember to add HeliPort to your macOS "Login Items" (System Settings > General > Login Items) so it launches automatically on boot and auto-connects.
- **Windows Fast Boot:** If you dual-boot with Windows, you **MUST disable "Fast Startup"** in Windows Power Settings. Otherwise, Windows will lock the Intel AX210 hardware, causing it to fail when you boot into macOS.
- **NVRAM Resets:** Any major changes applied to this EFI configuration require an NVRAM Reset from the OpenCore boot picker to take effect successfully.

---
*Note: This README is continuously updated as new hardware features are patched and stabilized.*

## Credits

- [Acidanthera](https://github.com/acidanthera) for OpenCore, Lilu, WhateverGreen, VirtualSMC, and other core kexts.
- [Dortania](https://dortania.github.io/OpenCore-Install-Guide/) for the incredible and comprehensive OpenCore Installation Guide.
- [OpenIntelWireless](https://github.com/OpenIntelWireless) for `itlwm`, `IntelBluetoothFirmware`, and making Intel networking possible on macOS.
- [corpnewt](https://github.com/corpnewt) for indispensable tools like `GenSMBIOS`.
