# trashOS 1.1 - Xeon adventures

## Note: This repo is an unofficial fork of MykolaG's trashOS repo with some of my own marks. The most up-to-date information (but not all of it) is in the [OpenCore Install Guide](https://dortania.github.io/OpenCore-Install-Guide/)

### This repo in under maintenance. Some information might not be up-to-date or working with your hardware or OpenCore version. It's more an individual notes, so consider reading official  Installation guide.
This repo is mainly used as a getting started with page with Core 2 Duo/Quad or Harpertown Xeon series hardware running on OpenCore.
The hardware that I was using:
- Asus P5K (1201 AHCI BIOS with Xeon microcodes)
- Intel Xeon E5450 (4 cores, 3.0 GHz)
- 4x 2 GB Kingston DDR2 800 MHz memory
- Sapphire Radeon RX 470 4 GB OC
- Crucial BX500 240 GB SPM
- Seagate Desktop HDD 500 GB 7200 ROM
## Install process

1. Get macOS: [GibmacOS](https://github.com/corpnewt/gibMacOS)
2. Build installer: [`createinstallmedia`](https://support.apple.com/en-us/HT201372)
3. Enable legacy booting: [Legacy Install](https://dortania.github.io/OpenCore-Desktop-Guide/extras/legacy)
4. Build rest of installer
5. Boot and install as usual
6. Mojave and newer users: You'll also want to grab the [telemetrap.kext](https://forums.macrumors.com/threads/mp3-1-others-sse-4-2-emulation-to-enable-amd-metal-driver.2206682/post-28447707) to fix the SSE4,2 requirement


## OpenCore Specifics

### config.plist

#### ACPI


#### Add

* [SSDT-EC](https://github.com/acidanthera/OpenCorePkg/blob/master/Docs/AcpiSamples/SSDT-EC.dsl)
  * Allows AppleBusPowerController to load
* [SSDT-HPET](https://github.com/corpnewt/SSDTTime)
  * Used for resolving IRQ conflicts, see here on how to generate the file: [SSDTTime](https://dortania.github.io/Getting-Started-With-ACPI/)

#### Patch

* [HPET _CSR to XCSR Patch](https://github.com/corpnewt/SSDTTime)
  * To be used with SSDT-HPET. See here on how to generate the file: [SSDTTime](https://dortania.github.io/Getting-Started-With-ACPI/)

#### Quirks

* FadtEnableReset: `True`
   * Needed for proper shutdown/restarts

### Booter

Disable all quirks related to Booter *except*:

* RebuildAppleMemoryMap: `True`
  * Required to boot 10.6 and older due early kernel panics

### Kernel

#### Kexts

The main ones are as follows:

* [VirtualSMC](https://github.com/acidanthera/VirtualSMC/releases) **or** FakeSMC
* [Lilu](https://github.com/acidanthera/Lilu/releases)
* [WhateverGreen](https://github.com/acidanthera/WhateverGreen/releases)
* [AppleALC](https://github.com/acidanthera/AppleALC/releases) **or** [VoodooHDA](https://sourceforge.net/projects/voodoohda/)
* [telemetrap.kext](https://forums.macrumors.com/threads/mp3-1-others-sse-4-2-emulation-to-enable-amd-metal-driver.2206682/post-28447707)

Ethernet gets a bit more complicated as we're going into the depths of legacy hackintosh kexts, so support on Catalina can be a bit sketchy:

* [AppleIntelE1000e](https://github.com/chris1111/AppleIntelE1000e/releases)
* [IntelMausi](https://github.com/acidanthera/IntelMausi/releases)
* [RealtekRTL8100](https://github.com/Mieze/RealtekRTL8100)
* [RealtekRTL8111](https://github.com/Mieze/RTL8111_driver_for_OS_X/releases)
* [AtherosE2200Ethernet](https://github.com/Mieze/AtherosE2200Ethernet/releases)

#### Quirks

* PanicNoKextDump: `True`
  * Makes debugging kernel panics in 10.13+ a bit easier

### Misc

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

### PlatformInfo

Few SMBIOS to choose from:

* `iMac10,1`
  * Recommended for High Sierra(10.13) and older
* `MacPro5,1`
  * Alternative to iMac10,1, also has support in Catalina
* `MacPro6,1`
  * Recommended for Big Sur(11)

### UEFI

**Firmware Drivers(.efi)**:

* [OpenRuntimeServices](https://github.com/acidanthera/OpenCorePkg/releases)
   * Helps with Native NVRAM and other misc. things
* [OpenUsbKbDxe](https://github.com/acidanthera/OpenCorePkg/releases)
   * Needed for the picker
* [HfsPlusLegacy](https://github.com/acidanthera/OcBinaryData)
   * Needed for seeing HFS volumes
* [ExFatDxeLegacy](https://github.com/acidanthera/OcBinaryData)
   * Needed for seeing drives made by Bootcamp assistant
* [PartitionDxeLegacy](https://github.com/acidanthera/OcBinaryData/blob/master/Drivers/PartitionDxeLegacy.efi)
  * Required for booting recovery partitions with 10.7 through 10.9. 10.10 and newer do not require this

**Kexts**:

The main ones are as follows:

* [VirtualSMC](https://github.com/acidanthera/VirtualSMC/releases) **or** FakeSMC
* [Lilu](https://github.com/acidanthera/Lilu/releases)
* [WhateverGreen](https://github.com/acidanthera/WhateverGreen/releases)
* [AppleALC](https://github.com/acidanthera/AppleALC/releases) **or** [VoodooHDA](https://sourceforge.net/projects/voodoohda/)
* [telemetrap.kext](https://forums.macrumors.com/threads/mp3-1-others-sse-4-2-emulation-to-enable-amd-metal-driver.2206682/post-28447707)

Ethernet gets a bit more complicated as we're going into the depths of legacy hackintosh kexts, so support on Catalina can be a bit sketchy:

* [AppleIntelE1000e](https://github.com/chris1111/AppleIntelE1000e/releases)
* [IntelMausi](https://github.com/acidanthera/IntelMausi/releases)
* [RealtekRTL8100](https://github.com/Mieze/RealtekRTL8100)
* [RealtekRTL8111](https://github.com/Mieze/RTL8111_driver_for_OS_X/releases)
* [AtherosE2200Ethernet](https://github.com/Mieze/AtherosE2200Ethernet/releases)
* [iats](https://code.google.com/archive/p/iats/downloads), which includes:
    + AtherosL1Ethernet/AttansicL1Ethernet for 10.5 and 10.6 (the latter works even with Catalina)
    + Atheros L1c kexts
    + Atheros L1e kexts

## Credits
- [Apple](https://www.apple.com/ru/) for macOS
- [Acidanthera](https://dortania.github.io/) for creating OpenCore and the original guide
- [xeon-e5450.ru](https://xeon-e5450.ru/socket-775/bios-asus/) for hosting Asus P5K bioses and for information about 771 Xeons compatibility with 775 mobos