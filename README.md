# How to fix Fenvi T919 Wi-Fi on macOS 15.2 Sequoia using OCLP, and how to enable FileVault without errors
[![FenviT919](https://img.shields.io/badge/Fenvi-T919-green)](https://www.fenvi.com/product_detail_16.html)
![MacOS](https://img.shields.io/badge/macOS-15.2-purple.svg)


![SequoiaLogo](https://github.com/chrisdodgers/Fix_Fenvi-T919_FileVault_macOS_Sequoia_15.x/blob/main/Photos/FenviT919%2BFileVault-Sequoia.png)</br>

## About:
This is a simple guide on how to fix Wi-Fi with a Fenvi T919 when running macOS Sequoia. macOS 14+ has broken native support for BCM4360, which is the Wi-Fi chipset on a Fenvi T919. This guide has been tested and works perfectly on macOS 15.2. This guide will also how to setup FileVault, as I was originally running into an "Invalid Password" error when trying to setup FileVault on macOS Sequoia.

## What Works:

| Feature           | Details       |
| ------------------: | :-----------|
|Airdrop         	| Works Perfect |
| AirPlay           | Works Perfect |
| Handoff           | Works Perfect |
| Continuity        | Works Perfect |
| Universal Control |Usually Works (I noticed even on my real Macs this is finicky on Sequoia. Sometimes re-signing in with my Apple ID fixes UC)      
| FileVault         | Works Perfect |


## Pre-Requisites:
- You have already installed macOS 15.x Sequoia on your system following the [Dortania guide](https://dortania.github.io/OpenCore-Install-Guide/).
- You have a Fenvi T919 installed in your system.


## Fixing Wifi:
### Configuring Required Kexts:
You need to download the 2 following kexts which you can find by using [this link.](https://github.com/dortania/OpenCore-Legacy-Patcher/tree/main/payloads/Kexts/Wifi)

- `IOSkywalkFamily-v1.2.0.zip`
- `IO80211FamilyLegacy-v1.0.0.zip`

>[!NOTE]
>*These kexts are older kexts which gives our BCM4360 chipset support again. You can read more detailed information about this [here.](https://github.com/perez987/Broadcom-wifi-back-on-macOS-Sonoma-with-OCLP/blob/main/README.md)*
>

We also need to download AMFIPass. At the time of this guide, we will be using AMFIPass v1.4.1. This is required to use OCLP on Seqouia. You can find and download the latest version of AMFIPass [here.](https://github.com/dortania/OpenCore-Legacy-Patcher/tree/main/payloads/Kexts/Acidanthera). 

>[!NOTE]
>The other alternative instead of installing AMFIPass.kext is to use the boot-arg `amfi=0x80`. However, using this boot-arg could cause some applications or services not to work correctly! So I highly recommend just using AMFIPass.kext instead of using the `amfi=0x80` boot-arg.
>

1. Copy the 3 .kext files you downloaded into your EFI folder. Place them under `EFI -> OC-> Kexts`.

![KextsFolder](https://github.com/chrisdodgers/Fix_Fenvi-T919_FileVault_macOS_Sequoia_15.x/blob/main/Photos/Kexts-Folder.png)</br>

2. Open [ProperTree Editor](https://github.com/corpnewt/ProperTree) and select your config.plist inside `EFI -> OC`.
3. In the top menu, click on `File` and select `OC Snapshot`. This will automatically add and order our new kexts into our config.plist.
4. Navigate to `Kernel -> Block` in your config.plist. You should see an entry with an Identifier name that says `com.apple.IOSkywalkFamily`. Make sure you set `Enabled` = `True`.

![KernelBlock](https://github.com/chrisdodgers/Fix_Fenvi-T919_FileVault_macOS_Sequoia_15.x/blob/main/Photos/Kernel-Block-Config.png)</br>

>[!IMPORTANT]
>If you do not set this to `Enabled`=`True` you will kernel panic at boot! This is because the newly added IOSkywalkFamily kext will conflict with the one we are trying to replace!
>

>[!NOTE]
>If you do not already see this in your config.plist, you need to add this. If you made your own EFI from scratch using the latest version of OpenCore and if you followed the Dortania guide, this entry should have already been included by default with `Enabled` = `False`. It is ALWAYS highly recommended to make your own EFIs from scratch and to follow the [Dortania guide](https://dortania.github.io/OpenCore-Install-Guide/)!

### Configuring Partial-SIP:
OCLP (OpenCore Legacy Patcher) requires a minimum of Partial-SIP (System Integrity Protection) in order to perform root patching, which we have to do in order to have working Wi-Fi again. You can disable SIP entirely, but for security reasons and for application support reasons I do NOT recommend fully disabling SIP. 

- Navigate to `NVRAM -> Add -> 7C436110-AB2A-4BBB-A880-FE41995C9F82 -> csr-active-config`.
- Change the Data value of csr-active-config to `03080000` which sets SIP to Partial.
- Make sure to save your config.plist!

### Applying OCLP Root Patches:

- Since our config.plist has been updated, reboot/boot into macOS. 

>[!NOTE] 
>I highly recommend resetting your NVRAM after making changes to any parameters under NVRAM. You can do this if you have ResetNvramEntry.efi in your `EFI -> OC -> Drivers` folder. In the OpenCore BootPicker - you can access this by pressing the Space Bar and you will see an option to `Reset NVRAM`. 
>

- Download and open the latest version of OCLP [here.](https://github.com/dortania/Opencore-Legacy-Patcher/releases)
- Select `Post-Install Root Patch` and select `Start Root Patching`.
- Reboot your system once prompted to.

![OCLPMenu](https://github.com/chrisdodgers/Fix_Fenvi-T919_FileVault_macOS_Sequoia_15.x/blob/main/Photos/OCLP-Menu.png)</br>

>[!NOTE]
>
>- If you enabled FileVault BEFORE attempting to Root Patch, you will see an error that says `"Cannot patch due to the following reasons: FileVault is enabled"`. Disable FileVault and reboot to fix this error. If you see other reasons listed, this means either you did not set your `csr-active-config` correctly or you need to reset your NVRAM as mentioned in the previous note.
>- If you have verified again and done all the above and still are having trouble root patching, ensure that you have `SecureBootModel` = `Disabled` under `Misc -> Security` in your config.plist. 
>

- At this point you should now have working Wi-Fi in macOS Sequoia with your Fenvi-T919! 

### Enabling FileVault (Optional):
I discovered when I tried to enable FileVault, I would get an "Invalid Password" error no matter how I tried to save my encryption keys. This is due to us being booted with Partial SIP instead of Full SIP. Partial SIP is required for OCLP root-patches to work. However, the only way I was able to setup FileVault was temporarily enabling Full SIP. The good news is FileVault will work just fine once we set SIP from Full to Partial after setting up FileVault!

1. **Enable Full SIP**:
   - Go to `NVRAM -> Add -> 7C436110-AB2A-4BBB-A880-FE41995C9F82 -> csr-active-config` in your config.plist and set the value to `00000000`
   - Reboot system. (I would recommend like earlier in this guide to reset your NVRAM.)
2. **Enable FileVault in System Settings**:
   - At this point before enabling FileVault, in System Settings you could go ahead and sign-in with your Apple ID. Doing so before turning on FileVault will allow you the option to save your encryption key to iCloud.
   - Enable FileVault and be patient while FileVault encrypts your volume.
3. **Enable Full SIP**:
   - Now that FileVault has completed its setup, go ahead and go back to Partial SIP by setting the `NVRAM -> Add -> 7C436110-AB2A-4BBB-A880-FE41995C9F82 -> csr-active-config` value back to `03080000`.
   - Reboot system. (Once again, I would recommend that you reset your NVRAM.)

### Wi-Fi and FileVault (Optional) should now be 100% working on macOS Sequoia 15.2!
- I will try do my best to keep this guide up-to-date and provide further information relating to these fixes when new macOS Sequoia updates arrive and when I've tested them.
- *If necessary, I may add an update proceedure to this guide.* 

## Photos:

![WiFiProof](https://github.com/chrisdodgers/Fix_Fenvi-T919_FileVault_macOS_Sequoia_15.x/blob/main/Photos/Wi-Fi-Proof-Sequoia.png)</br>

![UCProof](https://github.com/chrisdodgers/Fix_Fenvi-T919_FileVault_macOS_Sequoia_15.x/blob/main/Photos/UniversalControl-Proof-Sequoia.png)</br>

![AirDropProof](https://github.com/chrisdodgers/Fix_Fenvi-T919_FileVault_macOS_Sequoia_15.x/blob/main/Photos/AirDrop-Proof-Sequoia.png)</br>

![ContinuityProof](https://github.com/chrisdodgers/Fix_Fenvi-T919_FileVault_macOS_Sequoia_15.x/blob/main/Photos/Continuity-Proof-Sequoia.png)</br>


## Credits and Thanks:
- Apple for macOS
- [perez987](https://github.com/perez987/Broadcom-wifi-back-on-macOS-Sonoma-with-OCLP/blob/main/README.md) for the original guide/kext links for fixing Fenvi T919 on macOS 14+.
- Acidanthera for [OpenCore Bootloader](https://github.com/acidanthera/OpenCorePkg) and countless Kexts
- Dortania for [OpenCore Install Guide](https://dortania.github.io/OpenCore-Install-Guide) and [OpenCore Legacy Pacher](https://dortania.github.io/OpenCore-Legacy-Patcher/)
- [Corpnewt](https://github.com/corpnewt) for SSDTTime, GenSMBIOS, MountEFI, and ProperTree
- jsassu20 for [MacDown](https://macdown.uranusjr.com/) Markdown Editor   
- [5T33Z0](https://github.com/5T33Z0) for a great MD template used for making this guide, and for creating great guides and write-ups in the community.