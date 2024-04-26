# Asus-G771JM-Hackintosh
**Hackintosh Installation Guide  for Asus ROG G771JM**
<p align="center" style="margin:0 auto !important;text-align:center !important;"><img src="https://github.com/ouija/Asus-G771JM-Hackintosh/raw/master/Images/Asus-G771JM-Hackintosh.png"></p>

## Hardware Specs
- Intel Core i7-4710HQ [Haswell] Processor 2.5 GHz (Turbo up to 3.5 GHz)
- NVIDIA GeForce GTX 860M (4GB GDDR5, GM107) Optimus w/Intel HD4600
- Mediatek MT7630E (Ralink) WiFi -> Replaced with [BCM94352HMB](https://www.newegg.ca/broadcom-bcm94352hmb-m-2-mini-pci-e/p/0XM-00BD-00005)
- Realktek HD Audio ALC668
- Realtek RTS5208 PCI Card Reader
- Asus SmartTouch (Synaptics) PS2 Touchpad

## Preface
I seem to be a glutton for punishment, and decided to revive this machine back from the dead -- with plans of upgrading to to Sonoma.  That said, I _first_ had to get OpenCore running properly on this old bitch first, and that wasn't the easiest of tasks, when compared to Clover.

> [!NOTE]
>This isn't an install guide at the moment, but more of a **work in progress** so I can track my steps as I go, and I'll revise this after the fact.



## Clover to OpenCore Conversion Notes
<img src="https://github.com/ouija/Asus-G771JM-Hackintosh/raw/master/Images/OpenCore.png" style="width: 350px; margin-bottom:20px;">

Note that I previously had macOS Monterey 12.6.1 installed on this machine with Clover bootloader, and an older installation guide for that can be found [here](./macOS_12/README.md) if needed.

- Switching from Clover to OpenCore v0.9.9
- Followed the [Dortanio Laptop Haswell](https://dortania.github.io/OpenCore-Install-Guide/config-laptop.plist/haswell.html#acpi) guide to get OpenCore configured properly _(note: the guide is a little outdated and some guesswork is involved)_
- Generated all SSDTs  _(under Windows)_ via [SSDTTime](https://github.com/corpnewt/SSDTTime), which included `FixHPET`, `FakeEC Laptop`, `PluginType`, `USB Reset`, `PCI Bridge`, `PNLF`, `XOSI`, and `Fix DMAR`
- Mobile Intel HD 4600 working with 2048MB of VRAM and HDMI output working using `DeviceProperties` listed in the table below, which differed slightly from what I originally had defined with Clover.
- Realtek ALC668 audio working after changing **alcid** to `20` with HDMI output using `DeviceProperties` below and `alcid=20` boot arg.
	- Audio may not work unless IRQ patching via SSDTTime `FixHPET` is in place.
- Generate [USB Map](https://github.com/corpnewt/USBMap) _(under Windows)_ and replace/remove `USBInjectAll.kext` if present
	- Having mapping may have fixed issues with sleep
- Updating [AirportBrcmFixup.kext]() and/or [BrcmPatchRAM](https://github.com/acidanthera/BrcmPatchRAM) to latest version broke wifi support for BCM94352HMB _(Using older version for now)_
- Having issues with "Restart to black screen" with OpenCore, applied the [Fixing Shutdown/Restart](https://dortania.github.io/OpenCore-Post-Install/usb/misc/shutdown.html) patch but did not fix issue with restart, USB sleep issue fixed however, but may be due to USB mapping.
- May need to add [GPRW to XPRW Patch](https://dortania.github.io/OpenCore-Post-Install/usb/misc/instant-wake.html) to fix Instant wake due to USB
- Modified OC to use [BsxDarkFenceLight1](https://github.com/blackosx/BsxDarkFenceLight1) theme
- Still tweaking and improving, will update here accordingly.

## DeviceProperties

The following tables display the added PCI devices and their child keys.

### PciRoot(0x0)/Pci(0x1B,0x0)
Realtek 668 Audio / HDMI

| **Key**                  | **Type** |   **Value**  |
|--------------------------|:--------:|:------------:|
| No-hda-gfx               |  Data    | ``00001B59`` |
| alc-layout-id            |  Number  | ``16590000`` |


### PciRoot(0x0)/Pci(0x2,0x0)

Intel UHD 620 Graphics

| **Key**                  | **Type** |   **Value**  |
|--------------------------|:--------:|:------------:|
| AAPL,ig-platform-id      |   Data   | ``0600260A`` |
| AAPL00,override-no-edid  |   Data   | ``00FFFFFF FFFFFF00 30E46C04 00000000 00180104 95261578 0A0BB5A3 5955A027 0C505400 00000101 01010101 01010101 01010101 01012E36 804A7138 1F403020 35007ED7 1000001A 00000000 00000000 00000000 00000000 00000000 00FE004C 47204469 73706C61 790A2020 000000FE 004C5031 37335746 342D5350 443100D0`` |
| device-id                |   Data   | ``12040000`` |
| framebuffer-cursormem    |   Data   | ``00009000`` |
| framebuffer-patch-enable |   Data   | ``01000000`` |
| framebuffer-unifiedmem   |   Data   | ``00000080`` |