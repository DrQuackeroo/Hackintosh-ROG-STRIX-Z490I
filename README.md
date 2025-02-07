# Hackintosh-ROG-STRIX-Z490I

This repository is about hackintosh on **Asus ROG STRIX Z490I**. For now, all the hardware is working as expected, and it's ready for daily usage.

Anyone who has the same board can use my EFI directly. The source EFI folder uses debug version of OpenCore, mainly used for installation and testing. It’s recommended to use the release version for daily usage, you can replace it yourself or just download my release. Either way, don’t forget to edit the `EFI/OC/config.plist` file, you should generate your own SMBIOS info by following the [Comet Lake Config Guide #PlatformInfo](https://dortania.github.io/OpenCore-Install-Guide/config.plist/comet-lake.html). 

Highly recommended reading the whole [OpenCore Install Guide](https://dortania.github.io/OpenCore-Install-Guide/) before start.

## Hardware

* Motherboard: Asus ROG STRIX Z490-I
    * Ethernet: Intel I225-V 2.5Gbit
    * Wi-Fi/BT: Intel AX201NGW (onboard) / BCM94360CS(M.2 Adapter)
    * Audio: Realtek ALCS1220A
* CPU: Intel i7-10700
* GPU: Intel UHD630 / AMD Radeon RX 6600XT
* RAM: CORSAIR VENGEANCE LPX DDR4 3200 32GB(16G×2)
* SSD: Samsung 970 EVO Plus NVMe

## Software

* Bootloader: OpenCore 0.8.1-DEBUG
* OS: macOS Monterey 12.4 (iMac20,1)

## What's working

- [x] Intel UHD630 (iGPU)
- [x] AMD Radeon RX 6600XT (dGPU)
- [x] Audio Realtek ALCS1220A
- [x] Intel I225-V 2.5Gb Ethernet
- [x] Wi-Fi/BT (BCM94360CS)
- [x] USB
- [x] Restart/Shutdown
- [x] Sleep/Wake
- [x] Power Management (Native support)

## Details

### GPU

#### Intel UHD630

HDMI/DP display and audio output are working fine.

Working by:

* ig-platform-id = `07009B3E`

DeviceProperties: 

```xml
<key>PciRoot(0x0)/Pci(0x2,0x0)</key>
<dict>
    <key>AAPL,ig-platform-id</key>
    <data>BwCbPg==</data>
    <key>framebuffer-patch-enable</key>
    <data>AQAAAA==</data>
</dict>
```

#### AMD Radeon RX 6600XT

Native support with boot-args:

```
agdpmod=pikera
```

### Audio

Working by:

* AppleALC.kext
* layout-id=7

DeviceProperties: 

```xml
<key>PciRoot(0x0)/Pci(0x1f,0x3)</key>
<dict>
    <key>layout-id</key>
    <integer>7</integer>
</dict>
```

### Ethernet 

Working by:

* device-id=`F2150000`

DeviceProperties: 

```xml
<key>PciRoot(0x0)/Pci(0x1C,0x4)/Pci(0x0,0x0)</key>
<dict>
    <key>device-id</key>
    <data>8hUAAA==</data>
</dict>
```

Kernel Patch:

```xml
<dict>
    <key>Base</key>
    <string>__Z18e1000_set_mac_typeP8e1000_hw</string>
    <key>Comment</key>
    <string>I225-V patch</string>
    <key>Count</key>
    <integer>1</integer>
    <key>Enabled</key>
    <true/>
    <key>Find</key>
    <data>8hUAAA==</data>
    <key>Identifier</key>
    <string>com.apple.driver.AppleIntelI210Ethernet</string>
    <key>MaxKernel</key>
    <string>20.4.0</string>
    <key>MinKernel</key>
    <string>19.0.0</string>
    <key>Replace</key>
    <data>8xUAAA==</data>
</dict>
```

boot-args

```
dk.e1000=0
```

Use `e1000=0` instead of `dk.e1000=0` since macOS Monterey 12.3.


### Wi-Fi/BT

Working by using an m.2 M-Key adapter with Apple Airport Card BCM94360CS. It's natively supported, airdrop, handoff, and sidecar are working perfectly. The bottom side m.2 slot for SSD is occupied and the bottom SSD heat sink must be abandoned.

The Bluetooth can not be recognized by default, it uses the onboard 9-pin USB2.0 port for power supply, so USB mapping should be fixed to make it work.

<img src="imgs/wifi-bt.png" width="500" alt="wifi-bt"/>


> The onboard wireless network card Intel AX201NGW uses m.2 E-Key slot and CNVi protocol. I tried to replace it with an m.2 A-Key BCM94352Z card, the slot is compatible but it didn't work even in Windows, so don't try to replace the onboard card if you don't know what you're doing.


### USB

All ports are working fine except for the ones disabled due to the 15 port limit.

<img src="imgs/usb-rear.png" width="500" alt="usb-rear"/>
<img src="imgs/usb-onboard.png" width="500" alt="usb-onboard"/>

USB Ports:

| No. | Type | Port |
|-----|------|------|
| 1   | USB 2.0 | HS12 |
| 2   | USB 2.0 | HS13 |
| 3   | USB 3.2 Gen 1 | HS09/SS09 |
| 4   | USB 3.2 Gen 1 | HS10/SS10 |
| 5   | USB 3.2 Gen 2 | HS05/SS05 |
| 6   | USB 3.2 Gen 2 | HS06/SS06 |
| 7   | USB 3.2 Gen 2 | HS03/SS03 |
| 8   | USB 3.2 Gen 2 | HS04/SS04 |
| 9   | USB 2.0 Hub | HS11 |
| 10  | USB 3.2 Gen 2 | HS01/SS01 |
| 11  | USB 3.2 Gen 1 | HS07/SS07 + HS08/SS08 |

> All ports: HS01 ~ HS14, SS01 ~ SS10, USR1 ~ USR2
>
> HS02: AURA LED Controller /  HS14: Onboard bluetooth


You can make your own mapping by hackintool, here's my choice:

<img src="imgs/usb-ports.png" width="500" alt="usb-ports"/>

### Sleep/Wake

Works with DP output and power button. GPRW Patch is used to disabling the USB device instant wake.

**Note:**
1. When using HDMI, the display cannot be woken up.
2. Without enabling GPRW, a keyboard press or mouse click can wake up the display as well, but a second press or click is needed when the light is on, I tried to fix it by following [Keyboard Wake Issues Guide](https://dortania.github.io/USB-Map-Guide/misc/keyboard.html), but didn't work. So my choice is to just use the power button, disable `SSDT-GPRW` if you want to use a keyboard or mouse to wake up.

### F1 Boot Error

> Don't bother if you didn't run into this error.

Add patch to `Kernel -> Patch`:

```xml
<dict>
    <key>Base</key>
    <string></string>
    <key>Comment</key>
    <string>F1 Startup patch</string>
    <key>Count</key>
    <integer>1</integer>
    <key>Enabled</key>
    <true/>
    <key>Find</key>
    <data>dTMPtw==</data>
    <key>Identifier</key>
    <string>com.apple.driver.AppleRTC</string>
    <key>Limit</key>
    <integer>0</integer>
    <key>Mask</key>
    <data></data>
    <key>MaxKernel</key>
    <string></string>
    <key>MinKernel</key>
    <string></string>
    <key>Replace</key>
    <data>6zMPtw==</data>
    <key>ReplaceMask</key>
    <data></data>
    <key>Skip</key>
    <integer>0</integer>
</dict>
```

### BIOS

> Version: 2403

#### Disable

* Fast Boot
* VT-d
* CSM
* Intel SGX
* CFG Lock (no option in BIOS, Asus Z490 motherboards are factory unlocked. The `AppleCpuPmCfgLock` and `AppleXcpmCfgLock` quirks are not necessary)

#### Enable

* VT-x (no option in BIOS, it's enabled by default)
* Above 4G decoding
* Hyper-Threading
* EHCI/XHCI Hand-off
* OS type: Windows UEFI Mode (Clear Secure Boot Keys or choose `Other` type)
* DVMT Pre-Allocated(iGPU Memory): 64MB

### EFI

#### SSDTs

Compiled by following the [Dortania's ACPI Guide](https://dortania.github.io/Getting-Started-With-ACPI/), the `.dls` SSDT files can be found in SSDTS folder. 

* SSDT-AWAC.aml
* SSDT-EC-USBX.aml
* SSDT-PLUG.aml
* SSDT-SBUS-MCHC.aml
* SSDT-RHUB.aml
* SSDT-GPRW.aml (prebuild)

#### Kexts

* [VirtualSMC.kext](https://github.com/acidanthera/VirtualSMC) `1.2.9`
* [SMCProcessor.kext](https://github.com/acidanthera/VirtualSMC) `1.2.9`
* [SMCSuperIO.kext](https://github.com/acidanthera/VirtualSMC) `1.2.9`
* [Lilu.kext](https://github.com/acidanthera/Lilu) `1.6.0`
* [WhateverGreen.kext](https://github.com/acidanthera/WhateverGreen) `1.5.9`
* [AppleALC.kext](https://github.com/acidanthera/AppleALC) `1.7.2`
* [NVMeFix.kext](https://github.com/acidanthera/NVMeFix) `1.0.9`
* USBPorts-All.kext (disabled by default, include all ports of this board, for iMac20,1)
* USBPorts.kext (exported by hackintool, for iMac20,1, personal edition)

> Deprecated

* RadeonBoost.kext `v1.6`
* IntelMausi.kext `1.0.7`
* FakePCIID.kext (from RehabMan `2018-1027`)
* FakePCIID_intel_I225-V.kext
* FakePCIID_Intel_HDMI_Audio.kext (from RehabMan `2018-1027`)
* USBInjectAll.kext `v0.7.6` (missing HS11 port used for bluetooth)

## Misc

### Installation

The [OpenCore Install Guide](https://dortania.github.io/OpenCore-Install-Guide/) is quite clear and easy, so there will be no detailed installation tutorials here. Give it some patience and you can build your own EFI.

### Tools

* Generating SMBIOS: [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS)
* Compiling SSDT: [MaciASL](https://github.com/acidanthera/MaciASL)
* Mounting EFI system partition: [MountEFI](https://github.com/corpnewt/MountEFI)
* Editing plist file: [PropereTree](https://github.com/corpnewt/ProperTree)
* Dumping DSDT: [SSDTTime](https://github.com/corpnewt/SSDTTime)
* Toolbox: [Hackintool](https://github.com/headkaze/Hackintool)

### Benchmarks

| Item | Score |
|---|---|
| CPU - Geekbench | Single-Core: 1234 / Multi-Core: 8909 |
| Intel UHD630 - Geekbench | OpenCL: 4826 / Metal: 4790 |
| AMD Radeon RX 6600XT - Geekbench | OpenCL: 63628: / Metal: 76831 |

## Credits

* [OpenCorePkg](https://github.com/acidanthera/OpenCorePkg)
* [OpenCore Install Guide](https://dortania.github.io/OpenCore-Install-Guide/)
