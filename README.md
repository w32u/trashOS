# trashOS 1.1 - Xeon adventures

## Note: This repo is an unofficial fork of MykolaG's trashOS repo with some of my own marks. The most up-to-date information (but not all of it) is in the [OpenCore Install Guide](https://dortania.github.io/OpenCore-Install-Guide/)

### This repo in under maintenance. Some information might not be up-to-date or working with your hardware or OpenCore version. It's more like individual notes, so consider reading official  Installation guide.
This repo is mainly used as a getting started with page with Core 2 Duo/Quad or Harpertown Xeon series hardware running on OpenCore.
The hardware that I was using:

**Motherboard:**

- Asus P5K (1201 AHCI BIOS with Xeon microcodes)

**Processor:**

- Intel Xeon E5450 (4 cores, 3.0 GHz)

**RAM:**

- 4x 2 GB Kingston DDR2 800 MHz memory

**GPUs:**

- Sapphire Radeon RX 470 4 GB OC
- Zotac GeForce GTX 260 896 MB (failed to inject)
- XFX GeForce 8800 GS XXX 384 MB (in process)

**Storage:**

- Crucial BX500 240 GB SSD
	- ~~There are actually some problems with this SSD model when using APFS like I/O errors, 10.12.6 with HFS+ boots and works just fine~~ Nevermind, it was my fault.
- Seagate Desktop HDD 500 GB 7200 RPM
## Install process

There's nothing unusual, proceed with the [Creating the USB](https://dortania.github.io/OpenCore-Install-Guide/installer-guide/#creating-the-usb) guide, set up the Legacy bootsector, EFI folder, kexts, etc. and install macOS as usual.
A special note is that for 10.14+ you may require [telemetrap.kext](https://forums.macrumors.com/threads/mp3-1-others-sse-4-2-emulation-to-enable-amd-metal-driver.2206682/post-28447707) for loading the system

## OpenCore Specifics

### config.plist

#### ACPI


#### Add

* [SSDT-EC](https://dortania.github.io/Getting-Started-With-ACPI/Universal/ec-methods/ssdttime.html#fixing-embedded-controllers-ssdttime)
  * Allows AppleBusPowerController to load
* [SSDT-HPET](https://dortania.github.io/Getting-Started-With-ACPI/Universal/irq.html)
  * Used for resolving IRQ conflicts

#### Patch

* Just use all the given patches after generating SSDTs in SSDTTime from `Patches_OC.plist`

#### Quirks

* FadtEnableReset: `True`
   * Needed for proper shutdown/restarts

### Booter

Disable all quirks related to Booter *except*:

* RebuildAppleMemoryMap: `True`
  * Required to boot 10.6 and older due early kernel panics
* ForceExitBootServices: `True`
    * Helps to avoid `X64 EXCEPTION TYPE` error on OS X 10.6 and older

### Kernel

#### Kexts

The main ones are as follows:

* [VirtualSMC](https://github.com/acidanthera/VirtualSMC/releases) **or** [FakeSMC](https://bitbucket.org/RehabMan/os-x-fakesmc-kozlek/downloads/)
* [Lilu](https://github.com/acidanthera/Lilu/releases)
* [WhateverGreen](https://github.com/acidanthera/WhateverGreen/releases)
* [AppleALC](https://github.com/acidanthera/AppleALC/releases) **or** [VoodooHDA](https://sourceforge.net/projects/voodoohda/)
    * Not needed if you use USB plug-and-play card or headphones 
* [telemetrap.kext](https://forums.macrumors.com/threads/mp3-1-others-sse-4-2-emulation-to-enable-amd-metal-driver.2206682/post-28447707) for 10.14+
* [MouSSE](https://forums.macrumors.com/threads/mp3-1-others-sse-4-2-emulation-to-enable-amd-metal-driver.2206682/) for emulating SSE4.2 instruction set, needed by AMD metal drivers (7xxx+) since 10.13
    * If you don't use AMD GPUs, adding this kext will be pretty much useless

Ethernet gets a bit more complicated as we're going into the depths of legacy hackintosh kexts, so support on Catalina can be a bit sketchy:

* [AppleIntelE1000e](https://github.com/chris1111/AppleIntelE1000e/releases)
* [IntelMausi](https://github.com/acidanthera/IntelMausi/releases)
* [RealtekRTL8100](https://github.com/Mieze/RealtekRTL8100)
* [RealtekRTL8111](https://github.com/Mieze/RTL8111_driver_for_OS_X/releases)
* [AtherosE2200Ethernet](https://github.com/Mieze/AtherosE2200Ethernet/releases)
* [iats](https://code.google.com/archive/p/iats/downloads), which includes:
    * AtherosL1Ethernet/AttansicL1Ethernet for 10.5 and 10.6 (the latter works even with Catalina)
    * Atheros L1c kexts
    * Atheros L1e kexts

#### Quirks

* PanicNoKextDump: `True`
  * Makes debugging kernel panics in 10.13+ a bit easier

### Misc

* SecureBootModel: `j137`
    * Required for booting 10.13.2 - 10.15.7
    * Actually you can use any value listed in SecureBootModel section of Post-Install guide, but pay attention to the `Minimum macOS Version` section
* Vault: `Optional`
   * Required for booting unless you've signed OpenCore yourself
* ScanPolicy: `0`
   * To show all drives at the OpenCore picker

### NVRAM

* boot-arg:
  * `-nehalem_error_disable`
     * used with MacPro5,1 SMBIOS to avoid kernel panics
  * `-no_compat_check`
     * Allows older SMBIOS to be used in newer versions of macOS
  * `nv_disable=1`
     * A temporary bootarg for turning off old NVIDIA acceleration. If you try to boot without that arg and without NVIDIA card patched, you'll get a black screen.
  * `npci=0x2000`
     * Use this bootarg if you have trouble booting a GPU that has over 4 GBs of VRAM and you don't have `Above 4G Decoding` feature in your BIOS settings

### PlatformInfo

Several SMBIOSES are included here:

* **Core 2 Duo series:**
    * `iMac7,1`
        * Recommended for 10.4.10 - 10.5.8
    * `iMac10,1`
        * Recommended for 10.6.1 - 10.13.6
    * `MacPro6,1`
        * Recommended for 10.14 and higher in tandem with [telemetrap.kext](https://forums.macrumors.com/posts/28447707) 

* **Core 2 Quad / Harpertown Xeon series:**
    * `MacPro2,1`
        * Should be used for 10.4.9 - 10.4.11 (haven't tested yet)
    * `MacPro3,1`
        * Should be used for 10.5.1 - 10.11.6 (haven't tested yet)
    * `MacPro5,1`
        * Recommended for 10.12 - 10.14.6
            * 10.14 - 10.14.6 should be used with [telemetrap.kext](https://forums.macrumors.com/posts/28447707) 
        * This SMBIOS needs `-nehalem_error_disable` bootarg
    * `MacPro6,1`
        * Recommended for 10.15 and higher in tandem with [telemetrap.kext](https://forums.macrumors.com/posts/28447707) 

### UEFI

**Firmware Drivers(.efi)**:

* [OpenRuntimeServices](https://github.com/acidanthera/OpenCorePkg/releases)
   * Helps with Native NVRAM and other misc. things
* [OpenUsbKbDxe](https://github.com/acidanthera/OpenCorePkg/releases)
   * Needed for the keyboard support in the picker
* [HfsPlusLegacy](https://github.com/acidanthera/OcBinaryData)
   * Needed for seeing HFS volumes

## Credits
- [Apple](https://www.apple.com/ru/) for macOS
- [Acidanthera](https://dortania.github.io/) for creating OpenCore and the original guide and for the SMBIOS table
- [Asus](https://www.asus.com/ru/) for creating the original P5K BIOS
- [xeon-e5450.ru](https://xeon-e5450.ru/socket-775/bios-asus/) for hosting Asus P5K bioses and for information about 771 Xeons compatibility with 775 mobos
- [khronokernel](https://github.com/khronokernel) for the original repo
- [Mactracker](https://mactracker.ca/) for more accurate macOS release versions on required SMBIOSes
