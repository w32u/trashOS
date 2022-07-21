# Mac OS X 10.4.11 Tiger on Harpertown hardware

## Note: This repo is an unofficial fork of MykolaG's trashOS repo with some of my own marks. The most up-to-date information (but not all of it) is in the [OpenCore Install Guide](https://dortania.github.io/OpenCore-Install-Guide/)

### This repo in under maintenance. Some information might not be up-to-date or working with your hardware or OpenCore version. It's more like individual notes, so consider reading official  Installation guide

### Also a good thing to note that at the moment of writing this section there has been no succesfull attempts to run 10.4 Tiger on my hardware

This repo is mainly used as a getting started with page with Core 2 Duo/Quad or Harpertown Xeon series hardware running 10.4.11 Tiger on OpenCore.

The hardware that I was using:

**Motherboard:**

- Asus P5K (1201 AHCI BIOS with Xeon microcodes)

**Processor:**

- Intel Xeon E5450 (4 cores, 3.0 GHz)

**RAM:**

- 4x 2 GB Kingston DDR2 800 MHz memory

**GPU:**

- MSI GeForce 6600 GT (128 MB GDDR3)

**Storage:**

- Kingston A400 120 GB SATA SSD

## Installation process

There's nothing unusual, proceed with the [Creating the USB](https://dortania.github.io/OpenCore-Install-Guide/installer-guide/#creating-the-usb) guide, set up the Legacy bootsector, EFI folder, kexts, etc. and install macOS as usual.

## OpenCore Specifics

### config.plist

#### ACPI

##### Add

- [SSDT-EC](https://dortania.github.io/Getting-Started-With-ACPI/Universal/ec-methods/ssdttime.html#fixing-embedded-controllers-ssdttime)
  - Allows AppleBusPowerController to load
- [SSDT-HPET](https://dortania.github.io/Getting-Started-With-ACPI/Universal/irq.html)
  - Used for resolving IRQ conflicts

##### Patch

- Just use all the given patches after generating SSDTs in SSDTTime from `Patches_OC.plist`

##### Quirks

- FadtEnableReset: `True`
  - Needed for proper shutdown/restarts

#### Booter

Disable all quirks related to Booter *except*:

- RebuildAppleMemoryMap: `True`
  - Required to boot 10.6 and older due early kernel panics
- ForceExitBootServices: `True`
  - Helps to avoid `X64 EXCEPTION TYPE` error on OS X 10.6 and older

#### Kernel

##### Kexts

The main ones are as follows:

- [FakeSMC](https://github.com/khronokernel/Legacy-Kexts/blob/master/32Bit-only/Zip/FakeSMC-32.kext.zip)
- [VoodooHDA](https://github.com/khronokernel/Legacy-Kexts/blob/master/FAT/Zip/VoodooHDA.kext.zip)
  - Not needed if you use USB plug-and-play card or headphones
- [ATAPortInjector.kext](https://github.com/khronokernel/Legacy-Kexts/blob/master/Injectors/Zip/ATAPortInjector.kext.zip) (if you don't have AHCI support)

Ethernet gets a bit more complicated as we're going into the depths of legacy hackintosh kexts, so support on Catalina can be a bit sketchy:

- [Khronokernel's list of Ethernet kexts](https://github.com/khronokernel/Legacy-Kexts#32-bit-kexts)
- [Atheros Ethernet kexts](https://code.google.com/archive/p/iats/downloads) (try using Leopard ones, [one for Atheros L1](https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/iats/Atl1kext20080413.zip) works in Tiger)

##### Quirks

- ExternalDiskIcons: `True`
  - Helps in case if either your drives show as external on older OSes or there are some troubles with reading the drive

#### Misc

- SecureBootModel: `Disabled`
  - Note that such an old OS X as 10.4 this setting will not have any effect at all
- Vault: `Optional`
  - Required for booting unless you've signed OpenCore yourself
- ScanPolicy: `0`
  - To show all drives at the OpenCore picker

#### NVRAM

- boot-arg:
  - `nv_disable=1`
    - A temporary bootarg for turning off old NVIDIA acceleration. If you try to boot without that arg and without NVIDIA card patched, you'll get a black screen.
  - `npci=0x2000`
    - Use this bootarg if you have stuck at setting up PCI devices

#### PlatformInfo

Several SMBIOSes that support 10.4 are included here:

- **Core 2 Duo series:**
  - `iMac4,1`
    - Recommended for 10.4.11 if using a 32-bit CPU
  - `iMac7,1`
    - Recommended for 10.4.11 if using either Merom or Penryn LGA775 Core 2 Duo

- **Core 2 Quad / Harpertown Xeon series:**
  - `MacPro2,1`
    - Should be used for 10.4.9 - 10.4.11

### UEFI

**Firmware Drivers(.efi)**:

- [OpenRuntimeServices](https://github.com/acidanthera/OpenCorePkg/releases)
  - Helps with Native NVRAM and other misc. things
- [OpenUsbKbDxe](https://github.com/acidanthera/OpenCorePkg/releases)
  - Needed for the keyboard support in the picker
- [HfsPlusLegacy](https://github.com/acidanthera/OcBinaryData)
  - Needed for seeing HFS volumes

## Credits

- [Apple](https://www.apple.com/ru/) for macOS
- [Acidanthera](https://dortania.github.io/) for creating OpenCore and the original guide and for the SMBIOS table
- [Asus](https://www.asus.com/ru/) for creating the original P5K BIOS
- [xeon-e5450.ru](https://xeon-e5450.ru/socket-775/bios-asus/) for hosting Asus P5K bioses and for information about 771 Xeons compatibility with 775 mobos
- [khronokernel](https://github.com/khronokernel) for the original repo and [legacy kexts repo](https://github.com/khronokernel/Legacy-Kexts)
- [Mactracker](https://mactracker.ca/) and [EveryMac.com](https://everymac.com/) for more accurate macOS release versions on required SMBIOSes
