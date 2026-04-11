# HP Pavilion Gaming Laptop 15-DK0XXX - Hackintosh EFI

This repository contains the OpenCore EFI configuration required to run latest macOS natively on the HP Pavilion Gaming Laptop 15-DK0XXX. The setup focuses on system stability, maintaining maximum security (SIP Enabled), and avoiding intrusive root patches (no OCLP required for core functionality).

## 💻 System Specifications

- **Laptop Model:** HP Pavilion Gaming Laptop 15-DK0XXX (HM370 Chipset)
- **Processor:** Intel Core i7-9750H (6-Core, Coffee Lake)
- **Integrated Graphics:** Intel UHD Graphics 630 (1920x1080 Internal Display)
- **Discrete Graphics:** NVIDIA GeForce GTX 1650 (Unsupported in macOS - Disabled via boot-args `wegnoegpu`)
- **Storage:** KIOXIA BG3 NVMe SSD
- **Ethernet:** Realtek RTL8168 Gigabit Ethernet
- **Target OS:** macOS Tahoe (Version 26)
- **Bootloader:** OpenCore v1.0.7
- **SMBIOS:** `MacBookPro16,1`

## 🚀 Working Features & Configurations

### Network & Connectivity (Upgraded to Intel AX210)
The stock Wi-Fi card was physically replaced with an **Intel AX210NGW** module.

- **Wi-Fi:** Fully functional using the `itlwm.kext` driver. Requires the **HeliPort** application running on macOS to select and connect to wireless networks.
- **Bluetooth:** Native Bluetooth functionality is fully operational.
  - Handled by the `OpenIntelWireless` suite (`IntelBluetoothFirmware.kext`, `IntelBTPatcher.kext`, and `BlueToolFixup.kext`).
  - **Tahoe Compatibility Note:** To bypass Apple's strict Bluetooth firmware checks in macOS Tahoe, two critical variables are natively injected via OpenCore NVRAM: `bluetoothExternalDongleFailed` and `bluetoothInternalControllerInfo`.

### USB Mapping
- Internal and external USB ports have been strictly mapped using `UTBMap.kext` to ensure correct power delivery, sleep functioning, and internal Bluetooth hardware discovery.

### Security
- **System Integrity Protection (SIP):** Fully ENABLED (`csr-active-config` = `00000000`).
- The system is engineered to run as close to real Mac hardware dependencies as possible.

## ⚙️ Post-Installation Notes
- **HeliPort:** Remember to add HeliPort to your macOS "Login Items" (System Settings > General > Login Items) so it launches automatically on boot and auto-connects to your saved Wi-Fi networks.
- **NVRAM Resets:** Any major changes applied to this EFI configuration (especially Kexts and NVRAM keys) require an NVRAM Reset from the OpenCore boot picker to take effect successfully.

---
*Note: This README is continuously updated as new hardware features are patched and stabilized.*
