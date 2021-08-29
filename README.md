# ASUS ROG B460I Hackintosh

Building a Hackintosh on ROG STRIX B460-I using OpenCore.
## Software
| Name | Version |
| :-: | :-: |
| macOS | 11.5.2 |
| OpenCore | 0.7.2 |

## Features
| Function | Status | Comments |
| :-: | :-: | :-: |
| USB | ✅ | See USB mapping section |
| Ethernet | ✅ | |
| Wi-Fi | ✅ | Need to swap Wi-Fi card |
| Bluetooth | ✅ | Need to swap Wi-Fi card |
| AirDrop, Handoff, Universal Clipboard | ✅ | Need to swap Wi-Fi card |
| Hardware Acceleration | ✅ | Enable iGPU Multi-Monitor in BIOS |
| DRM | ❗ | WhateverGreen has no support for Navi 21 yet, use iGPU if you need DRM |
| Sleep | ❗ | Instant wake up when use HDMI and DP together, otherwise sleep is fine |
| USB-C on dGPU | ❗ | Charging and video output working, but no data transmission |

## Hardware
| Part | Model | Comments |
| :-: | :-: | :-: |
| CPU | Intel Core i9-10850K | Remove power limit, OC to 4.7G |
| MOBO | ASUS ROG B460I | |
| RAM | Asgard T2 32G*2 3000 MHz| Limited to 2933 MHz due to B460 chipset limit |
| GPU | AMD RX 6800 | Support from macOS 11.4 onwards |
| Storage 1 | Sandisk Ultra 2TB | Used for OS installation and dualboot|
| Storage 2 | Hikvision C2000 Pro 1TB | Used for Time Machine |
| PSU | Corsair SF750 Platinum | |
| Wi-Fi Card | BCM93452Z | Fenvi card, no kexts needed |

## Wi-Fi Card
The reason to choose this mobo is that it has an AX200 Wi-Fi Card pre-installed which means no CNVi so we can swap the card.
There are quite limiting choices for M.2 Wi-Fi card:
| Chipset | Model |
| :-: | :-: |
| BCM94360NG | Fenvi BCM94360NG |
| BCM943602| Dell DW1830 |
| BCM94352Z | Fenvi AC1200 <br> Dell DW1560 <br> Lenovo Lite-On WCBN802B <br> AzureWave AW-CB162NF |
| BCM94350ZAE | Lenovo Foxconn T77H649 <br> Lite-On WCBN808B <br> Dell DW1820A |

If you're not using a Fenvi card, then you'll need two more kexts for it to work:

**AirportBrcmFixup**(https://github.com/acidanthera/AirportBrcmFixup/releases)
**BrcmPatchRAM**(https://github.com/acidanthera/BrcmPatchRAM/releases)
<ul>
<li>BrcmBluetoothInjector</li>
<li>BrcmFirmwareData</li>
<li>BrcmPatchRAM3 for 10.14+, paired with BrcmBluetoothInjector</li>
</ul>

You need to put the required kexts in OC/Kexts. After that, run **File -> OC Snapshot** in ProperTree and choose your OC folder so the new kexts will be loaded.

You can also use mini PCIe Wi-Fi card with M.2 adapter but it won't fit into the original container so it's a hit or miss.

## USB Mapping
This motherboard has several USB ports and they're defined as the following:
![Top](https://lightfocus-1256547063.cos.ap-hongkong.myqcloud.com/hackintosh/mb-top.png)
![Back](https://lightfocus-1256547063.cos.ap-hongkong.myqcloud.com/hackintosh/mb-back.png)

The 20Gbps type-c port on the back panel does not need mapping as it's not controlled by XHC but PXSX, possibly a PCIe controller. The type-c port that's used as audio output is useless in macOS.

For my personal use, I mapped HS01 - HS08, HS12, SS01 - SS02, SS05 - SS08 due to the 15 ports limitation in macOS. So I have all the USB ports on the back panel work albeit port 03 and 04 are only USB 2.0, also I have two USB 3.0 ports on the front panel.

If you want to change the mapping, make your own USBMap.kext and replace the one in the OC/Kexts folder.

## BIOS
Few things need to be taken care of in the BIOS.

| Setting | Value|
| :-: | :-: |
| Fast Boot | Disable |
| Secure Boot | Disable |
| VT-d | Disable |
| CSM | Disable |
| Intel SGX | Disable |
| VT-x | Enable |
| Above 4G decoding | Enable |
| Hyper-Threading | Enable |
| XHCI Hand-off | Enable |
| DVMT Pre-Allocated | 64MB |
| SATA Mode | AHCI |
| iGPU Multi-Monitor | Enable |

Note: In the newer version of BIOS, when enabling **Above 4G decoding**, please make sure **Re-size BAR Support** is **Disable**. Make sure you connect the monitor to the dGPU.

## Modify config.plist in the OC folder
This repository contains EFI based on OpenCore 0.6.6. If you're using the same mobo, then this EFI is likely working for you. But if you have different parts other than mobo, please read the following content and modify it accrodingly.

You need ProperTree(https://github.com/corpnewt/ProperTree) to open and edit config.plist.

### DeviceProperties
Since I have both iGPU and dGPU, I set it as output by dGPU and iGPU is only used for hardware accleration.
If you wish to use the iGPU as output, you'll need to change **AAPL,ig-platform-id** from **0300C89B** to **00009B3E**.

### NVRAM

For Navi users(RX 5000/6000 series), you need to add **agdpmod=pikera**.


If you hate all the debug info when booting, you can also remove **-v** parameter but I suggest you remove it only when your hacking is up and running fine.

### PlatformInfo
I left this part blank intentionally because you really need your own serial number.

To create a new serial number, you can use GenSMBIOS(https://github.com/corpnewt/GenSMBIOS)

As for SMBIOS, it depends on your CPU:
| SMBIOS | CPU |
| :-: | :-: |
| iMac20,1 | i7-10700K and lower (8 core and lower) |
| iMac20,2 | i9-10850K and higher (10 core) |

Using GenSMBIOS, you'll get **Type** , **Serial** , **Board Serial** and **SmUUID**

**Type** goes to Generic -> SystemProductName
**Serial** goes to Generic -> SystemSerialNumber
**Board Serial** goes to Generic -> MLB
**SmUUID** goes to Generic -> SystemUUID

**TL;DR** You need your own serial number and if you have a different CPU or want to use iGPU as output, you need to change a few things in config.plist

## Screenshots and Benchmark (Tested on macOS 11.5.2)

**Hardware Info**

![Hardware](https://lightfocus-1256547063.cos.ap-hongkong.myqcloud.com/hackintosh/Sensei.png)

**Geekbench 5**

CPU Scores

![CPU](https://lightfocus-1256547063.cos.ap-hongkong.myqcloud.com/hackintosh/CPU.png)

GPU Scores

![GPU](https://lightfocus-1256547063.cos.ap-hongkong.myqcloud.com/hackintosh/GPU.png)

**Blackmagic RAW Speed Test**

![BM RAW](https://lightfocus-1256547063.cos.ap-hongkong.myqcloud.com/hackintosh/BM.png)

