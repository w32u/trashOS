# ASUS P5K Harpertown Tiger build

```
CPU:     Intel Xeon E5450
MOBO:    ASUS P5K
RAM:     8GB (4x 2GB 800MHz)
GPU:     MSI 6600 GT 128MB GDDR3
NET:     Attansic L1 Gigabit
AUDIO:   ALC883 (not working in Tiger)
BOOT:    OpenCore
```

## THIS IS A PROOF-OF-CONCEPT PAGE AND ITS REFERENCE EFI FOLDER. IT IS RECOMMENDED [TO BUILD YOUR EFI FOLDER YOURSELF](https://dortania.github.io/OpenCore-Install-Guide/) UNLESS YOU KNOW WHAT YOU ARE DOING

## What works/doesn't work

Works:

* Tested versions of macOS:
  * Mac OS X 10.4.10
  * Mac OS X 10.4.11
* Ethernet
* External USB Audio
* Hardware Acceleration (QE/CI)
* AHCI, Xeons or both using custom BIOS in the same named folder

Doesn't work:

* Startup Disk/NVRAM

## ACPI

* SSDT-EC.aml
  * Fake EC controller, it's required
* SSDT-HPET.aml
  * Rules out IRQ conflicts also using ACPI -> Patch patches. For compatibility purposes.

## Kexts

These are just the essential ones to get booting and have proper functionality:

* [Lilu.kext](https://github.com/acidanthera/Lilu/releases)
* [VirtualSMC.kext](https://github.com/acidanthera/VirtualSMC/releases)
* [FakeSMC-32](https://github.com/khronokernel/Legacy-Kexts/blob/master/32Bit-only/Zip/FakeSMC-32.kext.zip) (disabled in config.plist)
* [AttansicL1Ethernet.kexts](https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/iats/Atl1kext20080413.zip)

Architecture of these kexts is set to i386.
