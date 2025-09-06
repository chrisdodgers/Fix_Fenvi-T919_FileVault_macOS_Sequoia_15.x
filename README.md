# How to Fix Fenvi T919 Wi-Fi on macOS 15.6.1 Sequoia using OCLP and How to Fix Setting Up FileVault 
[![FenviT919](https://img.shields.io/badge/Fenvi-T919-green)](https://www.fenvi.com/product_detail_16.html)
![MacOS](https://img.shields.io/badge/FileVault-blue.svg)
![MacOS](https://img.shields.io/badge/macOS-15.3.1-purple.svg)


![SequoiaLogo](https://github.com/chrisdodgers/Fix_Fenvi-T919_FileVault_macOS_Sequoia_15.x/blob/main/Photos/FenviT919%2BFileVault-Sequoia.png)</br>

## About:
This is a simple guide on how to fix Wi-Fi with a Fenvi T919 when running macOS Sequoia (This guide also applies to macOS Sonoma). macOS 14+ has broken native support for BCM4360, which is the Wi-Fi chipset used on a Fenvi T919. This guide has been tested and works on macOS 15.3.1. This guide also includes how to fix/setup FileVault, as I was originally running into an "Invalid Password" error when trying to setup FileVault on macOS Sequoia when SIP was partially disabled. 


>[!IMPORTANT]
>Do NOT attempt this guide on the beta of macOS Tahoe as it will fail at the current moment. Please be kind and patient while the great folks behind OCLP work on updating patches to support macOS Tahoe.  
>
## What Works:

| Feature           | Details       |
| ------------------: | :-----------|
| AirDrop         	| ✅ Works great |
| AirPlay           | ✅ Works great |
| Handoff           | ✅ Works great |
| Continuity        | ✅ Works great |
| Universal Control | ✅ Usually works (I noticed even on my real Macs this is sometimes finicky on Sequoia. Sometimes re-signing in with my Apple ID fixes UC.)      
| FileVault         | ✅ Works great once setup |


## Pre-Requisites:
- You have already installed macOS 15.x Sequoia on your system following the [Dortania guide](https://dortania.github.io/OpenCore-Install-Guide/).
- You have a Fenvi T919 installed in your system.


## Configuring our config.plist:
### Downloading and Configuring Required Kexts:
You need to download the 2 following kexts which you can find by using [this link.](https://github.com/dortania/OpenCore-Legacy-Patcher/tree/main/payloads/Kexts/Wifi)

- `IOSkywalkFamily-v1.2.0.zip`
- `IO80211FamilyLegacy-v1.0.0.zip`

>[!NOTE]
>*These kexts are older kexts which gives our BCM4360 chipset support again. You can read more detailed information about this [here.](https://github.com/perez987/Broadcom-wifi-back-on-macOS-Sonoma-with-OCLP/blob/main/README.md)*
>

We also need to download AMFIPass. At the time of this guide, we will be using AMFIPass v1.4.1. This is required to use OCLP on our Seqouia install. [Download](https://github.com/dortania/OpenCore-Legacy-Patcher/tree/main/payloads/Kexts/Acidanthera) the latest version of AMFIPass.

>[!NOTE]
>The other alternative instead of installing AMFIPass.kext is to use the boot-arg `amfi=0x80` which fully disables AMFI. However, using this boot-arg could cause some applications or services not to work correctly! So I highly recommend just using AMFIPass.kext instead of using the `amfi=0x80` boot-arg.
>

1. Copy the 3 kexts you downloaded into your EFI folder. (Place them under `EFI -> OC-> Kexts`)

![KextsFolder](https://github.com/chrisdodgers/Fix_Fenvi-T919_FileVault_macOS_Sequoia_15.x/blob/main/Photos/Kexts-Folder.png)</br>

2. Open [ProperTree Editor](https://github.com/corpnewt/ProperTree) and select your config.plist inside `EFI -> OC`.
3. In the top menu, click on `File` and select `OC Snapshot`. This will automatically add and order our new kexts into our config.plist.
4. Navigate to `Kernel -> Block` in your config.plist. You most likely will see an entry with an Identifier name that says `com.apple.IOSkywalkFamily`. Make sure you set `Enabled` = `True`!

![KernelBlock](https://github.com/chrisdodgers/Fix_Fenvi-T919_FileVault_macOS_Sequoia_15.x/blob/main/Photos/Kernel-Block-Config.png)</br>

>[!IMPORTANT]
>**If you do not set this entry to `Enabled`=`True`, then you will kernel panic at boot! This is because the newly added IOSkywalkFamily kext will conflict with the one we are trying to replace!** 
>
>**If you do not already have this entry in your config.plist:**
>
> - You can easily add this in ProperTree by right-clicking on `Kernel` and in the drop-down menu, select `OpenCore` -> `Kernel -> Block` -> `IOSkywalkFamily (Sonoma+)`. This will add the entry you need.
>
>![ProperTreeAddIOSkywalkFamily](https://github.com/chrisdodgers/Fix_Fenvi-T919_FileVault_macOS_Sequoia_15.x/blob/main/Photos/ProperTree-Add-IOSkywalkFamily-Entry.png)</br>

### Configuring Partial SIP:
OCLP (OpenCore Legacy Patcher) requires a minimum of Partial SIP (System Integrity Protection) in order to perform root patching, which we have to do in order to have working Wi-Fi again. You can disable SIP entirely, but for security reasons and for application support reasons I do NOT recommend fully disabling SIP. 

- Navigate to `NVRAM -> Add -> 7C436110-AB2A-4BBB-A880-FE41995C9F82 -> csr-active-config`.
- Change the Data value of csr-active-config to `03080000` which sets SIP to Partial.

>[!NOTE]
> I would highly recommend adding an entry for `csr-active-config` under `NVRAM -> Delete -> 7C436110-AB2A-4BBB-A880-FE41995C9F82`. Doing this will always ensure the `csr-active-config` value stored in your NVRAM will always be up-to-date with the value you have set in your config.plist. This means we do not have to reset NVRAM each time we change our csr-active-config. (You can do this for other NVRAM variables like boot-args for example. Thank you [corpnewt](https://github.com/corpnewt) for this great tip!) 

![SetSipPartial](https://github.com/chrisdodgers/Fix_Fenvi-T919_FileVault_macOS_Sequoia_15.x/blob/main/Photos/Set-SIP-Partial.png)</br>

### Ensure SecureBootModel is Disabled:

- Navigate to `Misc -> Security -> SecureBootModel`. Make sure this is set to `Disabled`.

![SecureBootModelDisabled](https://github.com/chrisdodgers/Fix_Fenvi-T919_FileVault_macOS_Sequoia_15.x/blob/main/Photos/SecureBootModel-Disabled.png)</br>

>[!NOTE]
>
>- You will have issues with macOS updates and OCLP if this is not set to `Disabled`!
>  - If this was previously NOT set to `Disabled`, then you will need to perform a NVRAM reset for this to fully take effect!
		- You can perform a NVRAM reset by first ensuring `ResetNvramEntry.efi` is in your `EFI -> OC -> Drivers` folder. In the OpenCore BootPicker - you can then presss the Space Bar and you will see an option to `Reset NVRAM`). 


		
### Adding Required Patches/Boot-Args for FileVault (Optional):
You will need to follow this section of the guide only if you plan on using FileVault. If you do not plan on using FileVault, you can skip this section of the guide.

1. We need to add a kernel patch which will allow FileVault to be enabled with SIP lowered: 
  - Clone/download the latest version of [ProperTree Editor](https://github.com/corpnewt/ProperTree) which includes the FileVault patch we need.
  - In ProperTree, navigate to `Kernel -> Patch`.
  - Right-click on 'Patch' and in the drop-down menu, select `OpenCore` -> `Kernel -> Patch` -> `Force FileVault on Broken Seal (11.3+)`. This will add the patch you need.

![AddFVKernelPatch](https://github.com/chrisdodgers/Fix_Fenvi-T919_FileVault_macOS_Sequoia_15.x/blob/main/Photos/Add-FileVault-Kernel-Patch.png)</br>

![FVKernelPatchApplied](https://github.com/chrisdodgers/Fix_Fenvi-T919_FileVault_macOS_Sequoia_15.x/blob/main/Photos/FileVault-Kernel-Patch.png)</br>  
  
>[!NOTE]
> 
> - *(Optionally, if you would like to instead manually add this patch - you can find it [here](https://github.com/chrisdodgers/Fix_Fenvi-T919_FileVault_macOS_Sequoia_15.x/blob/main/Files/Patch_Force-FileVault-on-Broken-Seal.plist)*


2. We need to add a NVRAM variable that allows OCLP to apply root patches with FileVault enabled. *Without this, OCLP will return an error asking you to disable FileVault when attempting to apply root patches.*:
 - Navigate to `NVRAM -> Add -> 4D1FDA02-38C7-4A6A-9CC6-4BCCA8B30102`
 - Right-click on `4D1FDA02-38C7-4A6A-9CC6-4BCCA8B30102` and select `New child under 4D1F...`
 - Rename `New String` to `OCLP-Settings`
 - Set the value of `OCLP-Settings` to `-allow_fv`
 - Navigate to `NVRAM -> Delete -> 4D1FDA02-38C7-4A6A-9CC6-4BCCA8B30102`
 - Right-click on `4D1FDA02-38C7-4A6A-9CC6-4BCCA8B30102` and select `New child under 4D1F...`
 - Set the string value to `OCLP-Settings`

![OCLPSettings](https://github.com/chrisdodgers/Fix_Fenvi-T919_FileVault_macOS_Sequoia_15.x/blob/main/Photos/Add-OCLP-Settings.png)</br>

## Applying OCLP Root Patches:
(Since we have now finished making the needed changes in our config.plist, make sure to reboot your system before applying root patches!)

1. [Download](https://github.com/dortania/Opencore-Legacy-Patcher/releases) and open the latest version of OCLP.
2. Select `Post-Install Root Patch` in the main menu.

![OCLPMenu](https://github.com/chrisdodgers/Fix_Fenvi-T919_FileVault_macOS_Sequoia_15.x/blob/main/Photos/OCLP-Menu.png)</br>

3. Select `Start Root Patching`.

![StartPatching](https://github.com/chrisdodgers/Fix_Fenvi-T919_FileVault_macOS_Sequoia_15.x/blob/main/Photos/OCLP-StartPatching.png)</br>

4. Reboot your system once prompted to.

### Wi-Fi and FileVault (Optional) should now be 100% working on macOS Sequoia 15.6.1!
- I will try to keep this guide up-to-date and provide further information relating to these fixes when new macOS Sequoia updates arrive and once I've tested them.
- *If necessary, I may add an update proceedure and additional helpful information to this guide.* 

## Photos:

![WiFiProof](https://github.com/chrisdodgers/Fix_Fenvi-T919_FileVault_macOS_Sequoia_15.x/blob/main/Photos/Wi-Fi-Proof-Sequoia.png)</br>

![UCProof](https://github.com/chrisdodgers/Fix_Fenvi-T919_FileVault_macOS_Sequoia_15.x/blob/main/Photos/UniversalControl-Proof-Sequoia.png)</br>

![AirDropProof](https://github.com/chrisdodgers/Fix_Fenvi-T919_FileVault_macOS_Sequoia_15.x/blob/main/Photos/AirDrop-Proof-Sequoia.png)</br>

![ContinuityProof](https://github.com/chrisdodgers/Fix_Fenvi-T919_FileVault_macOS_Sequoia_15.x/blob/main/Photos/Continuity-Proof-Sequoia.png)</br>


## Credits and Thanks:
- Apple for macOS
- [perez987](https://github.com/perez987/Broadcom-wifi-back-on-macOS-Sonoma-with-OCLP/blob/main/README.md) for the original guide I read for fixing Fenvi T919 on macOS 14+.
- [Corpnewt](https://github.com/corpnewt) for providing very helpful information/resources which was used in this guide. (And for creating ProperTree, and other great software like MountEFI and SSDTTime).
- Acidanthera for [OpenCore Bootloader](https://github.com/acidanthera/OpenCorePkg) and countless Kexts.
- Dortania for [OpenCore Install Guide](https://dortania.github.io/OpenCore-Install-Guide) and [OpenCore Legacy Pacher](https://dortania.github.io/OpenCore-Legacy-Patcher/).
- [mrlimerunner](https://github.com/mrlimerunner/sonoma-wifi-hacks?tab=readme-ov-file) for an older guide with good information.
- jsassu20 for [MacDown](https://macdown.uranusjr.com/) Markdown Editor   
- [5T33Z0](https://github.com/5T33Z0) for a great MD template used for making this guide, and for creating great guides and write-ups in the community.