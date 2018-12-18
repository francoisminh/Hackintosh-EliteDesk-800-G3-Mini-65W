# Hackintosh-EliteDesk-800-G3-Mini-65W
Here a the files and guides I used for the installation of OS X Mojave on HP EliteDesk 800 G3 Mini

## Description
Product Name : HP EliteDesk 800 G3 Mini 65W / HP EliteDesk 800 G3 Desktop Mini 65 W

Product N° 1ND88ET#ABF

- CPU : Intel Core i5-6500 3.2 GHz
- GPU : Intel HD Graphics 530 (Dual Display Port, HDMI)
- Memory : 2 Slots DDR4-2133
- Storage : PCIe NVMe 256 GB, PCI-E Gen3 x 4, Turbo Drive G2 Toshiba KXG50ZNV256G 
- Ethernet : Intel I219LM Gigabit Network Connection LOM
- Wireless LAN : Intel 7265 802.11ac 2x2 Wi-Fi with Bluetooth M.2 Combo Card 
- Audio : Conexant CX20632 (Internal Speaker, 2 audio out front ports)

>> Wireless LAN : Replaced Intel M.2 Combo Card with M.2 Combo Card for Lenovo Y50-70 with Broadcom Chipset BCM94352Z NGFF 802.11ac Wifi + Bluetooth 

## Known Issues
Conexant CX20632 requires VoodooHDA.kext which is incompatible with HDMI Audio. If you wish to use HDMI Audio output replace VoodooHDA.kext with AppleALC.kext and inject audio channel 11
Skylake Sleep issue cause reboot on wake. Disable sleep in Engergy Settings.

## Installation
### Important Note :
This guide isn't a Vanilla install guide since I couldn't get the BCM94352Z to work with kexts installed on the EFI partition
For a Vanilla install, simply skip the Wireless LAN part and use a Wifi-Bluetooth USB Key instead

### BIOS Settings
- Disable Fast Boot
- Enable USB boot
- UEFI Boot order, make sure your USB device is first to boot
- Secure Boot : Legacy Support Enable and Secure Boot Disable
- Disable : Virtualization Technology for Directed I/O (VTd)
- Enable : M.2 SSD (if you use a NVMe SSD)
- Uncheck M.2 WLAN/BT (if you don't have a compatible M.2 Combo Card)
- Disable Wake on lan
- Set Video Memory to 64 MB

### Installation USB Key
Create an USB Key with Mojave manually or using a software like Diskmaker 

### Installation of Clover on the installation disk
I used Clover V2.4 Release 4798
Customize installation
**NOTE : These are the settings are for motherboards doesn't support UEFI.**

- Uncheck Clover for UEFI booting only
- Uncheck Install Clover in the ESP
- Boot Sectors :
  - Check Install boot0af in MBR
- Clover for BIOS (legacy) booting
  - Clover EFI 64-bits SATA
- Uncheck Bios Drivers, 64 bits
- Uncheck FileVault 2 Bios Drivers, 64 bit
- UEFI Drivers check :
  - DataHubDxe-64
  - FSInject-64
  - SMCHelper-64
  - VBoxHfs-64
  - ApfsDriverLoader-64
  - EmuVariableUefi-64
  - NvmExpressDxe-64 (if you have a nvme SSD)
  - OsxAptioFix3Drv-64
  - OsxFatBinaryDrv-64
- FileVault 2 UEFI Drivers check :
  - AppleImageCodec-64
  - AppleKeyAggregator-64
  - AppleKeyFeeder-64
  - FirmwareVolume-64

Download Clover Configurator and copy it on the installation key.

Boot on USB and install Mojave. I chose to format the disk in APFS.

During the installation the system will reboot a couple of times. 
When the system reboot on the clover screen make sure you **Inject Intel GFX** and set Input **FakeID : 0x19120000.**

If you don't want to enter these settings on clover boot screen you can edit the config.plist from your installation key and set them.

## Post Installation
### Add Clover on Boot disk
Copy Clover Configurator and use it to mount the EFI Partition of your system. 

You can mount manually your EFI Partition.
On a terminal window
Identify the EFI partition on your disk with the command
- diskutil list
Create a mount point 
- mkdir /Volumes/efi
Mount partition
- sudo mount -t msdos /dev/disk0s1 /Volumes/efi

### Add kexts and ssdt patches
Copy the EFI folder on your Installation key on the EFI of your 
Add the following kext in **/EFI/kexts/Other**
- Lilu.kext (https://github.com/acidanthera/Lilu)
- VoodooHDA.kext (https://sourceforge.net/projects/voodoohda/)
- IntelMausiEthernet.kext (https://bitbucket.org/RehabMan/os-x-intel-network/downloads/)
- WhateverGreen.kext (https://github.com/acidanthera/WhateverGreen)
- FakeSMC.kext (https://github.com/RehabMan/OS-X-FakeSMC-kozlek)
- USBInjectAll.kext (https://github.com/RehabMan/OS-X-USB-Inject-All)

Use the following guide to generate a patched ssdt for CPU Power Management
https://www.tonymacx86.com/threads/quick-guide-to-generate-a-ssdt-for-cpu-power-management.177456/

Use the following guide to generate a patched ssdt for injecting USB Port
https://www.tonymacx86.com/threads/guide-creating-a-custom-ssdt-for-usbinjectall-kext.211311/

If you have a compatible M.2 Combo Card use the following guide
https://www.tonymacx86.com/threads/broadcom-wifi-bluetooth-guide.242423/

#### config.plist
**ACPI Section**

Add the following DSDT Patches
Rename HDAS to HDEF 
Find : 48444153 
Replace with : 48444546

Change SAT0 to SATA
Find : 53415430
Replace with : 53415441

Check :
- FixAirport
- FixHDA
- PluginType

SortedOrder
- ssdt.aml (patched ssdt for CPU generated)
- ssdt-UIAC.aml (patched ssdt for USB Injection generated)

**Boot Section**

Check :
- dart=0
- -disablegfxfirmware
- NeverHibernate
- HibernationFixup

Boot argument :
-lilubetaall

If you have a compatible M.2 Combo Card use this boot argument (Replace FR by your country code)

-lilubetaall brcmfx-country=FR

**Graphics Section**
- Inject Intel
- ig-platform-id : 0x193B0005

**Kernel and Kexts patches**
Kexts to patches 

Name : AppleAHCIPort
Find : 45787465 726E616C
Replace with : 496E7465 726E616C

Name : com.apple.driver.AirPort.Brcm4360
Find : 81F952AA 00007529
Replace with : 81F952AA 00006690

**SMBIOS**
use iMac17,1 (Late 2015)

**If you use the attached config.plist generate a new Serial Number and a new SmUUID**
