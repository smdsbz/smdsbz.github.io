---
layout: post
title: The Possibly General Hackintosh Guide
---

General purpose tutorial on hackintosh, that comes with **absolutely no
warranty** :smirk:

### Current Specs.

- Model: Asus U3000ua
- CPU: Intel Core i7-6500U
- RAM: 2 * 4G DDR3L
- Graphics: Intel Integrated Graphics HD520
- Wireless: Intel 7265NGW (WiFi and Bluetooth)

### Currently Not Working

- WiFi connection achieved via external USB utility
- Battery capacity display
- `No Arguments ... _BIX`

### My EFI

[GitHub Repo](https://github.com/smdsbz/Asus_U3000)

### HOWTO

*Hackintosh is not for the weak.*

If you are not scared-off by the quote above, you're welcome to proceed. Just to
be sure, this guide comes with **absolutely no warranty**, but I will be glad
if you share the obstacles you've come across with other hackintosh-ers
like me.  

### Notes

- AMD cpus are not welcomed here.
- Never think about your dedicated graphics card (if you are using a laptop).
- Intel Network cards never worked.

Now, shall we begin!  

### Get Started

#### ACPI, DSDT and SSDT

ACPI, or *Advanced Configuration and Power Management*, gives hackers the
shimmering light to run macOS on non-peripheral hardwares. TL DR, it's the
way you can fool macOS to believe that your computer is a true Mac.  

DSDTs and SSDTs are the files unix-like systems use to achieve ACPI managing.
They could be very different between different specs, models. I will show you
how to get your own DSDT files, it's quite easy.  

#### EFI and UEFI

The EFI partition stores the bootloaders, systems are booted only after the
bootloader in the EFI partition boots up. You will be using *CLOVER* as your
bootloader. Search, download it, and put it into your EFI partition.  

#### Get Your DSDT

Change boot sequence in BIOS / UEFI, boot into CLOVER, press F4, and
you're golden! The DSDT files will be in your `EFI/CLOVER/ACPI/original`
folder.  

Make another folder `EFI/CLOVER/ACPI/patched`, and copy `DSDT.aml`,
`SSDT-xxx.aml` into the `patched` folder. Rename `SDT-xxx.aml`s in the fashion
`SDT-1.aml`, `SDT-2.aml`... where the numbers should remain untouched.  

#### `config.plist`

The `config.plist` file is a XML-like file, which contains all the
informations CLOVER needs to know, in order to "patch" the original macOS
system into one that runs on your computer.  

You can download the one that works best for you at
[here](https://github.com/RehabMan/OS-X-Clover-Laptop-Config).  


#### `drivers64UEFI` *64-bit machine*

`.efi` files are loaded before kernel is up.  

Some necessary files are `OsxAptioFixDrv-64.efi` (make hardware ID injections
possible), `apfs.efi` (enabling Apple's latest file system) and various
other things.  

It's kinda hard to explain what it actually does, but a copy-paste should do
the trick.  

#### `*.kext`

Kext, or *Kernel Extension*, are loaded after the macOS system kernel is up.
Heres where you can perform all your debug skills.  

Heres a list of kexts I use, to make certain parts of my machine works:  

- `FakePCIID-xxxxx.kext` - inject hardware IDs
- `FakeSMC.kext` - fake system management controller
- `Lilu.kext` - lots of other kexts concerning graphics and audio
  depends on this
- `CoreDisplayFixup.kext` - fix HiDPI issues, in some other cases fixes
  graphics memory issues too
- `VoodooHDA.kext` - try this if macOS doesn't recognize your speakers
- `VoodooPS2Controller.kext` - try this if macOS doesn't recognize your
  mouse / trackpad / keyboard

All of the kext above can be downloaded from GitHub, BitBucket.  

#### Boot Arguments

You can use boot arguments in CLOVER to enable...

- `-v` - to show verbose logs, very useful when you can't enter the installation
  view
- `nv_disable=1` - to disable dedicated graphics, since laptop nVidia GPUs are
  not available now
- `keepsyms=1` - to keep logs on *kernel panic*
- `debug=0x100` - to show more debug logs

Some are useful for debugging, some are crutial for booting up. The choice is
up to you.

#### HACK! HACK! HACK!

Good luck and break your leg!  

Try out all the possible combinations of `kext`s, `SSDT`s, `config.plist`s.
And don't forget to Google / Bing.  

