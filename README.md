# Asus-G771JM-Hackintosh
**Hackintosh Installation Guide  for Asus ROG G771JM**
<p align="center" style="margin:0 auto !important;text-align:center !important;"><img src="https://github.com/ouija/Asus-G771JM-Hackintosh/raw/master/Images/Asus-G771JM-Hackintosh.png"></p>

## Preface
It's been a very long time since I've done a *fresh install* on this unit, as I hackintoshed it with when first getting this unit 6 or 7 years ago with **Mavericks**, and have only done upgrades since then, so this is more for my own notes than an actual installation guide, and currently is in relation to supporting installation of **macOS Big Sur** and then upgrading to **Monterey**.

## Hardware Specs
- Intel Core i7-4710HQ [Haswell] Processor 2.5 GHz (Turbo up to 3.5 GHz)
- NVIDIA GeForce GTX 860M (4GB GDDR5, GM107) Optimus w/Intel HD4600
- Mediatek MT7630E (Ralink) WiFi -> Replaced with [BCM94352HMB](https://www.newegg.ca/broadcom-bcm94352hmb-m-2-mini-pci-e/p/0XM-00BD-00005)
- Realktek HD Audio ALC668
- Realtek RTS5208 PCI Card Reader
- Asus SmartTouch (Synaptics) PS2 Touchpad

## Quick Installation Notes

 - Use one of the [guides](https://www.tonymacx86.com/threads/how-to-create-a-macos-big-sur-public-beta-installation-usb.300100/) available on the internet to create USB installer for Big Sur and complete installation. **UPDATE**: Follow a [newer guide](https://manjaro.site/how-to-create-macos-monterey-usb-installer-for-hackintosh/) to create USB installer for Monterey if preferred
 - Note that I preferred to use **Clover** *(v5123+ for Big Sur, v5142+ for Monterey)*, as OpenCore seems to have an issue with rebooting on this machine *(shutdown works fine, but restart hangs on a blank screen; I was also having issues with sleep)*; There are notes at the end of this document about configuring OpenCore for reference.
 - I have also installed macOS to an internal NVMe drive and am booting from the Windows EFI partition with `NmeExpressDxe.efi` enabled *(because the bios lacks NVMe boot support)* and originally thought OpenCore was failing to detect macOS due to this, but further testing found that this was due to originally upgrading Big Sur via the main installation partition instead of the preboot partition (see below), which was another reason I originally stuck with Clover.
 - You can [build a custom BIOS](https://rog.asus.com/forum/showthread.php?110850-G771JM-Custom-BIOS-with-NVMe-Support) to enable support for booting directly from the NVMe controller/drive instead, but I was getting random BSOD in Windows and reverted back to this more stable method.
 - You need to enable some DSDT patches in Clover to support Catalina or greater, specifically the `EC0 to EC` patch *(see more detailed installation notes below)* but essentially can use the same `config.plist` for Clover on your installation USB if installed for the same `Asus G771JM` model.
 - **Improtant:** Ensure you select the `Preboot` partition after the second reboot during installation to avoid issues with AppStore downloads failing post-upgrade. (read more [here](https://github.com/CloverHackyColor/CloverBootloader/issues/300#issuecomment-731096921) and [here](https://www.reddit.com/r/hackintosh/comments/jtv7cn/clover_big_sur_app_store_download_stuck_on/)) 
	 - Edit the Clover `config.plist` file *(if using the one provided here)* and comment out or remove the line that hides this `Preboot` partition prior to installing Big Sur!
	 - [Signing of Apple ID and clearing cache](https://github.com/CloverHackyColor/CloverBootloader/issues/300#issuecomment-729718708), as well as booting once from preboot with [SIP enabled](https://hackintosher.com/forums/thread/enable-disable-system-integrity-protection-sip-on-a-hackintosh.53/) seemed to finally make it work again.
 - **2021 / Monterey Update:** Came across [improved properties](https://www.tonymacx86.com/threads/success-big-sur-11-1-on-asus-rog-g771jw-opencore-0-6-5-and-clover-5128.307754/) for the Intel HD4600, which have been updated below and include fixing the low resolution / CSM boot issue AND fixed VRAM to use all 2048 MB of memory that the video card is capable of _(instead of 1536 MB)_
	 - To upgrade to Monterey, I first updated Clover to the latest release, then updated all custom kexts to latest versions, then I had to change/set the SMBIOS definition to *MacBookPro11,5* to download and install the update, which worked flawlessly without issues.
	 - IMPORTANT NOTE: Remember that when updating the [AirportBrcmFixup.kext](https://github.com/acidanthera/AirportBrcmFixup) to delete/remove the `AirPortBrcm4360_Injector.kext` from plugins folder within kext to be able to support Big Sur / Monterey <i>(or there will be kernel panics becasue of this)</i>, and utilize `BlueToolFixup.kext` instead.
	 - Updated this guide with notes on how to properly enable brightness support in Big Sur/Monterey!
	 - Now using [USBToolBox](https://github.com/USBToolBox/tool) method for mapping USB ports as of Monterey 12.3.1, which seemed to introduce mapping issues with older USBMap method.

## Detailed Installation and Configuration Notes
**BIOS RELATED:**

Ensure BIOS has Display Memory set to 64MB and that both Secure Boot and CSM mode is disabled.   If VT-d left enabled, ensure to enable `DsiableIoMapper` quirk in Clover.

**CLOVER RELATED:**

 - **ACPI/DSDT/Patches**  include: `change _OSI to XOSI`, `change _DSM to XDSM`, `change EC0 to EC`, `change GFX0 to IGPU`, `change SAT0 to SATA`, `change EHC1 to EH01`, `change EHC2 to EH02`, and `change B0D3 to HDAU`
 
 - **ACPI/Fixes**:  Ensure that **NONE** of these fixes are checked/enabled in Clover, specifically the `AddPNLF` fix, or it will interfer with the `SSDT-PNLF.aml` used to inject the PNLF device used for enabling backlight/brightness slider controls.

  - **ACPI/SSDT/Generate/PluginType** is enabled (true) to take advantage of CPU scaling SSDT generated by **[ssdtPRGen](https://github.com/Piker-Alpha/ssdtPRGen.sh)**.

 - SMBIOS Definition set for <strike>MacBookPro11,2</strike> which closely matches the hardware of the Asus G771JM
	 - Update: macOS Monterey requires a minimum definition of **MacBookPro11,4** to be used

 - Enable the following Quicks in Clover v5123 or greater to enable proper OpenRuntime.efi support:
	 - AvoidRuntimeDefrag
	 - EnableSafeModeSlide
	 - EnableWriteUnprotector
	 - ProvideCustomSlide
	 - SetupVitrualMap
	 - DsiableIoMapper *(if VT-d enabled in BIOS)* 
	 - XhciPortLimit

- `Devices` has  `IntelGFX` set to `0x04128086`, `Audio` -> `Inject`set to 3, and ensure that `SetIntelBacklight` and `SetIntelMaxBacklight` are DISABLED/UNCHECKED _(these can interfere with backlight/brightness controls)_
	- `PciRoot(0x0)/Pci(0x1B,0x0)` is defined with the following properties: 
		- key: `No-hda-gfx` value: `00000000 00000000 00` type: `DATA`
		- key: `alc-layout-id` value: `3` type: `NUMBER`
	- `PciRoot(0x0)/Pci(0x2,0x0)` is defined with the following properties:
		- key: `AAPL,ig-platform-id` value: `0600260A` type: `DATA`
		- key: `AAPL00,override-no-edid` value: `00FFFFFF FFFFFF00 30E46C04 00000000 00180104 95261578 0A0BB5A3 5955A027 0C505400 00000101 01010101 01010101 01010101 01012E36 804A7138 1F403020 35007ED7 1000001A 00000000 00000000 00000000 00000000 00000000 00FE004C 47204469 73706C61 790A2020 000000FE 004C5031 37335746 342D5350 443100D0` type: `DATA`
		- key: `device-id` value: `12040000` type: `DATA`
		- key: `device_type` value: `VGA compatible controller` type: `STRING`
		- key: `enable-hdmi20` value: `1` type: `NUMBER`
		- key: `framebuffer-con1-enable` value: `1` type: `NUMBER`
		- key: `framebuffer-con1-flags` value: `06000000` type: `DATA`
		- key: `framebuffer-con1-pipe` valuue: `12000000` type: `DATA`
		- key: `framebuffer-con1-type` valuue: `00080000` type: `DATA`
		- key: `framebuffer-cursormem` value: `00009000` type: `DATA`
		- key: `framebuffer-patch-enable` value: `1` type: `NUMBER`
		- key: `framebuffer-patch0-enable` value: `1` type: `NUMBER`
		- key: `framebuffer-patch0-find` value: `02040900 00040000 87000000` type: `DATA`
		- key: `framebuffer-patch0-replace` value: `FF000900 00040000 87000000` type: `DATA`
		- key: `framebuffer-patch1-enable` value: `1` type: `NUMBER`
		- key: `framebuffer-patch1-find` value: `01000000 40000000 1E020000 05050901` type: `DATA`
		- key: `framebuffer-patch1-replace` value: `01000000 40000000 1F020000 05050901` type: `DATA`
		- key: `framebuffer-patch2-enable` value: `1` type: `NUMBER`
		- key: `framebuffer-patch2-find` value: `0300220D 00030303 00000002 00003001 00000000 00000060 9914` type: `DATA`
		- key: `framebuffer-patch2-replace` value: `0300220D 00030303 00000008 00000002 00000000 00000080 9914` type: `DATA`
		- key: `framebuffer-unifiedmem` value: `00000080` type: `DATA`
		- key: `model` value: `Intel HD Graphics 4600` type: `STRING`

- <strike>Note that it's better NOT to use the `CsmVideoDxe.efi` to enable higher resolutions if/when using external monitors and keep Clover boot resolution at 1024x768.</strike>
	- Update: This issue **should** now be resolved using these newer device properties for the `HD4600`, see note above _(not yet tested or verified)_

**KEXT RELATED:**
- Using **[Lilu.kext](https://github.com/acidanthera/lilu/releases)** and **[WhateverGreen.kext](https://github.com/acidanthera/whatevergreen/releases)** to enable Intel HD4600 _(with FakeID injection and device properties defined in Clover as outlined above)_, as well as `-wegnoegpu` boot argument to disable any external GPUs
- Using  **[VirtualSMC](https://github.com/acidanthera/virtualsmc/releases)**  _(instead of  [FakeSMC](https://bitbucket.org/RehabMan/os-x-fakesmc-kozlek/downloads/)  and  [ACPIBatteryManager.kext](https://bitbucket.org/RehabMan/os-x-acpi-battery-driver/downloads/))_ to enable battery status
	- Include all VirtualSMC kexts except for `SMCDellSensors.kext`
	- Patch DSDT with Rehabaman's `Asus G75VW`  battery patch
- Using **[AsusSMC](https://github.com/hieplpvip/AsusSMC)** with `[als] Fake ALS` and `[kbdl] Haswell/Broadwell/Skylake` and `[fn] F5 key` and `[fn] F6 key` patches _(not using AsusSMCDaemon)_ to enable Asus Function Keys and Keyboard Backlight Control _(instead of  [AsusNBFnKeys.kext](https://osxlatitude.com/forums/topic/1968-fn-hotkey-and-als-sensor-driver-for-asus-notebooks/))_
- Using acidanthera's [VoodooPS2.kext](https://github.com/acidanthera/VoodooPS2) instead of emlydinesh's [ApplePS2SmartTouchPad.kext](https://osxlatitude.com/forums/topic/1948-elan-focaltech-and-synaptics-smart-touchpad-driver-mac-os-x) which supports AsusSMC and F9 key to disable trackpad
- Using acidanthera's [AirportBrcmFixup.kext](https://github.com/acidanthera/AirportBrcmFixup) to enable wireless but need to set `brcmfx-driver=2` boot argument to enable, as well as removing `AirPortBrcm4360_Injector.kext` from plugins folder within kext to support Big Sur.
- Using acidanthera's [BrcmPatchRAM](https://github.com/acidanthera/BrcmPatchRAM) to enable bluetooth
	-  `BrcmPatchRAM3.kext`, `BrcmFirmwareData.kext`, and `BlueToolFixup.kext` _(prior to Monterey, `BrcmBluetoothInjector.kext` was used instead of `BlueToolFixup.kext`)_
- <del>Using custom **[USBMap.kext](https://github.com/corpnewt/USBMap)** to properly enable USB ports/hubs,  instead of FakePCIID.kexts *(which will cause slow startup and wifi issues in Big Sur)*</del>
		- Now using [USBToolBox](https://github.com/USBToolBox/tool) method for mapping USB ports as of Monterey 12.3.1, which seemed to introduce mapping issues with older USBMap method.
	- Note this enables camera, and is also needed to enable wireless and sdcard support
- Using cholonam's [Sinetek-rtsx.kext](https://github.com/cholonam/Sinetek-rtsx/releases) to enable SD card reader *([original version](https://github.com/sinetek/Sinetek-rtsx) is causing kernel panic in Big Sur when mounting SD card)*
- Using Mieze's  [RealtekRTL8111.kext](https://github.com/Mieze/RTL8111_driver_for_OS_X/releases) to enable LAN
- Using modified version of Rehabman's [CodecCommander.kext](https://bitbucket.org/RehabMan/os-x-eapd-codec-commander/downloads/) for `ALC668` to resolve [audio issue](https://www.tonymacx86.com/threads/alc1150-dual-boot-with-windows-and-10-10-3-no-sound-solved.162380/) when dual-booting with Windows.
- NOTE: If supported, consider using acidanthera's [NVMeFix](https://github.com/acidanthera/NVMeFix) to improve support for NVMe drive and enable TRIM *(optional - this can prevent startup if unsupported, which is the case with my NVMe and Monterey 12.2.1)*

**DSDT RELATED:**

 - [This guide](https://www.tonymacx86.com/threads/guide-patching-laptop-dsdt-ssdts.152573/)  by RehabMan is still *one of the best* by far when it comes to disassembling and modifying the DSDT.

 - All `Common Patches` from the RehabMan guide have been applied to the DSDT, including the `Fix PNOT/PPNT` patch, as I am <ins>not</ins> including OEM SSDT *(only ssdtPRGen SSDT)* and also apply  `Add IMEI` patch as DSDT does **not** contain HECI/IMEI device.
 - Applied `USB3_PRW 0x0D (instant wake)` patch to fix USB causing wake after sleep.
 - Applied `[bat] Asus G75VW` patch for battery status support.
 - Applied `[sys] Haswell LPC` patch for Haswell LPC support.
 - Applied [patches to disable Nvidia Optimus](https://www.insanelymac.com/forum/topic/295584-disabling-nvidia-optimus-card-on-all-laptops/) and improve battery life.
 - Using [SSDT-PNLF.aml](https://dortania.github.io/Getting-Started-With-ACPI/Laptops/backlight-methods/manual.html) to enable brightness controls _(without using **any** additional kexts like `BrightnessKeys.kext`)_ which I manually compiled from the .dsl file and applied Rehabman's `[igpu] Rename GFX0 to IGPU` patch to ensure it would function with the other `GFX0 to IGPU` patches being applied by Clover

---

**OpenCore Related:**

 - Note that I experienced issues with restart failing while using OpenCore and for that reason chose to stick with Clover for now, but OpenCore is pretty much the standard bootloader now when it comes to macOS.
 - Follow [Dortania's OpenCore Install Guide](https://dortania.github.io/OpenCore-Install-Guide/) specific to Haswell Laptops to get started. 
 - Note that (like with Clover), I had to set the bootloader resolution to 1024x768 in OpenCore or display output was garbled;  To do this edit `UEFI -> Output -> Resolution` in the OpenCore `config.plist` and set it to 1024x768.
 - There were also issues with `AppleALC` causing the system to fail on boot when `alcid` boot arg was present (or having DeviceProperties set for audio);  I believe this is due to needing the GFX0 -> IGPU and B0D3 -> HDAU patches in place  (I quickly tested using a patched DSDT and this fixed it; Still need to test with OpenCore hotpatch instead).
 - Getting Battery Status and AsusSMC/Keyboard Backlight requires DSDT edits _(since I cannot find SSDT hotpatch equivelents)_, however using a DSDT with OpenCore is not recommended, since it applies the ACPI to _all operating systems_ in a multiboot setup, so be careful when going this route!
 - If migrating from Clover, ensure you've upgraded to Big Sur via the suggested `Preboot` method above, or OpenCore will fail to detect the macOS partition.
 - Note on this unit, the BIOS has [CFG Lock](https://dortania.github.io/OpenCore-Post-Install/misc/msr-lock.html) enabled, which you can disable via [these instructions](https://dortania.github.io/OpenCore-Post-Install/misc/msr-lock.html#disabling-cfg-lock) and using this command: `setup_var_3 0x8A 0x00`
