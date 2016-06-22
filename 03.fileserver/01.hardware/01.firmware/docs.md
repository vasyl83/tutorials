---
title: LSI IT mode firmware
taxonomy:
    category: docs
---

In order to fully use zfs or to make sure that S.M.A.R.T data is correctly passed from RAID adapter to the operating system, LSI 9240-8i must be put into IT mode. What IT mode does is turn the RAID card into a simple HBA card. Since LSI 9240-8i doesnt have IT mode firmware available for it, the card needs to be flashed with 9211-8i firmware.

The process is quite easy. All thas is needed is a bootable USB stick with FreeDOS and a few of LSI tools. The motherboard is important as well, since these instructions are designed to take advantage of UEFI shell.

One of the easier ways to continue is to create a FreeDOS bootable USB stick with [Rufus](https://rufus.akeo.ie/) so that if anything fails DOS can easily be started to clear the card's NVRAM.

Latest version of LSI 9211-8i firmware, as of moment of the writing of this tutorial, is P20, if a newer firmware is released get the latest one.

Get **9211-8i_Package_P20_IR_IT_Firmware_BIOS_for_MSDOS_Windows** and **Installer_P20_for_UEFI** on the download tab of [LSI 9211-8i page](http://www.lsi.com/products/host-bus-adapters/pages/lsi-sas-9211-8i.aspx#tab/tab4).

From **9211-8i_Package_P20_IR_IT_Firmware_BIOS_for_MSDOS_Windows** copy **2118it.bin** from **Firmware\HBA_9211_8i_IT** onto the USB stick. Also get **mptsas2.rom** from **sasbios_rel** folder (only needed if the firmware version differs from the one flashed onto the card)

From **Installer_P20_for_UEFI** get **sas2flash.efi** from ** sas2flash_efi_ebc_rel** folder.

Download [sas2008.zip](http://www.files.laptopvideo2go.com/hdd/sas2008.zip) and unpack its contents into a separate folder on the USB stick. This package is needed incase **sas2flash.efi** fails by not finding the card or giving any kind of error.

Lastly, an EFI shell executable is needed in order for UEFI BIOS to start an EFI shell (it may not be the case for newer motherboards, but Asus P8Z68-V LX needed it). There are now two versions of EFI shell, old deprecated [version 1](https://github.com/tianocore/edk2/tree/master/EdkShellBinPkg/FullShell/X64) and new [version 2](https://github.com/tianocore/edk2/tree/master/ShellBinPkg/UefiShell/X64). Asus P8Z68-V LX worked with version 1. EFI shell must be named **shell.efi** in order to be started from UEFI BIOS, so rename the downloaded file and put it in the root of the USB stick.

Write down the SAS address of the card, it's on the back of the PCB on a green sticker (ie 500605B0xxxxxxxx), also as a precaution, remove all the storage and any cards other than LSI RAID card for the PC. Boot into UEFI BIOS with the USB stick plugged in.

Inside UEFI BIOS find the option to **Launch EFI Shell** or **Launch EFI Shell from filesystem device** and select it. Once inside EFI shell, mount the USB stick (EFI shell should list all the devices on startup, the USB stick should usually be fs0)
```
mount fs0:
```
Once the USB stick is mounted switch to it.
```
fs0:
```
To navigate use usual Linux commands: `ls`,`cd` etc.

Next execute the following commands:

To erase the current firmware `sas2flash.efi -o -e 6`.

To flash new one `sas2flash.efi -o -f 2118it.bin -b mptsas2.rom`.

Finally `sas2flash.efi -list` to see if it worked. You should see `Firmware Product ID:     0x2213 (IT)` on one of the lines.

In case the erase failed, boot from the USB stick into DOS navigate into the directory where the contents of [sas2008.zip](http://www.files.laptopvideo2go.com/hdd/sas2008.zip) are located and then run the following commands in order to clean the firmware:
```
megarec -writesbr 0 sbrempty.bin
megarec -cleanflash 0
```
Afterwards boot into EFI Shell and execute `sas2flsh -o -f 2118it.bin -b mptsas2.rom` in order to flash the new firmware and then `sas2flsh -o -sasadd 500605bxxxxxxxxx`(x= numbers for SAS address from the green sticker) to restore SAS address.

Run `sas2flash.efi -list` to see if it worked. Reboot and enjoy.