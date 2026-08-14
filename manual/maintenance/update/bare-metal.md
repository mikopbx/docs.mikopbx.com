---
description: Updating MikoPBX installed on a dedicated physical computer
---

# Updating a physical server

{% hint style="warning" %}
We recommend updating sequentially, without skipping releases and versions.
{% endhint %}

This guide is intended for a dedicated computer or server running MikoPBX.

The primary method is updating through the web interface. If the PBX does not boot normally or the web interface is unavailable, use an ISO image written to a separate USB drive.

{% hint style="danger" %}
Before updating, be sure to create a backup of the settings and store it on another device. Also save a copy of the call recordings.
{% endhint %}

## Preparation

1. Make sure you have physical access to the server or a working console. Create a backup of the MikoPBX settings and call recordings.
2. Make sure at least **400 MB** is free on the Storage.
3. End active calls and schedule a maintenance window for the telephony system.

## Updating through the web interface

### Online update

1. In the web interface, open **"Maintenance"** → **"PBX update"**.

<figure><img src="../../../.gitbook/assets/PBXUpdate_Section.png" alt=""><figcaption><p>The "PBX update" section</p></figcaption></figure>

2. Select the desired version in the list. Review the list of changes and start the update process.

<figure><img src="../../../.gitbook/assets/updatePBXFromWeb.png" alt=""><figcaption><p>Button for updating the PBX from the web interface</p></figcaption></figure>

3. Once the download is complete, enter the phrase **Yes, I have a backup**. Click **Update**.

<figure><img src="../../../.gitbook/assets/MikoPBX_UpdateConfirmation.png" alt=""><figcaption><p>Confirmation that a backup exists</p></figcaption></figure>

### Updating with a local IMG file

1. Download the `.img` file of the required version from the [MikoPBX releases](https://github.com/mikopbx/Core/releases) page.

{% hint style="info" %}
Pay attention to the architecture of your MikoPBX: if you have an ARM MikoPBX, use the update file with "arm64" in its name. If you have an x86 MikoPBX, use the update file with "x86\_64" in its name.
{% endhint %}

<figure><img src="../../../.gitbook/assets/MikoPBXGithubReleases-IMG.png" alt=""><figcaption><p>Downloading an .img for the update</p></figcaption></figure>

2. Open **Maintenance** → **PBX update**.

<figure><img src="../../../.gitbook/assets/PBXUpdate_Section.png" alt=""><figcaption><p>The "PBX update" section</p></figcaption></figure>

3. Select the downloaded `.img` file.

<figure><img src="../../../.gitbook/assets/MikoPBXUpdate_ChooseIMG.png" alt=""><figcaption><p>Selecting the update file</p></figcaption></figure>

4. Click **Apply update**.

<figure><img src="../../../.gitbook/assets/MikoPBX_ApplyUdate.png" alt=""><figcaption><p>Button for applying the update</p></figcaption></figure>

5. Enter the phrase **Yes, I have a backup** and confirm the operation.

<figure><img src="../../../.gitbook/assets/MikoPBX_UpdateConfirmation.png" alt=""><figcaption><p>Confirmation that a PBX backup exists</p></figcaption></figure>

Wait for the PBX to reboot automatically after the update.

<figure><img src="../../../.gitbook/assets/MikoPBXStation2026.3.40.png" alt=""><figcaption><p>The PBX running the new version</p></figcaption></figure>

## Updating from an ISO on a USB drive

This method is suitable for restoring and updating the PBX from the local console.

### Creating bootable media

1. Download the ISO of the required version from the [MikoPBX releases](https://github.com/mikopbx/Core/releases) page.

{% hint style="info" %}
Pay attention to the architecture of your MikoPBX: if you have an ARM MikoPBX, use the update file with "arm64" in its name. If you have an x86 MikoPBX, use the update file with "x86\_64" in its name.
{% endhint %}

<figure><img src="../../../.gitbook/assets/MikoPBXGithubReleases-ISO.png" alt=""><figcaption></figcaption></figure>

2. Connect a separate USB drive with a capacity of at least 1 GB.
3. Write the ISO in disk image mode using Rufus, balenaEtcher, or `dd`. Instructions for writing an ISO image can be found [here](../../../setup/bare-metal/live-usb.md#writing-the-image-to-a-usb-drive).

{% hint style="danger" %}
Writing the ISO will erase all data on the selected USB drive. Double-check the selected device.
{% endhint %}

### Running the update

1. Connect the prepared USB drive to the server. In the BIOS/UEFI or Boot Menu, select booting from USB.
2. Wait for MikoPBX to start in Recovery mode.

<figure><img src="../../../.gitbook/assets/MikoPBX-RecoveryMode3.40.png" alt=""><figcaption><p>MikoPBX in Recovery mode</p></figcaption></figure>

3. Press any key to open the console menu. Then open **"[4] Install or recover"**.

<figure><img src="../../../.gitbook/assets/MikoPBX-3.40-InstallOrRecover.png" alt=""><figcaption><p>The "Install or recover" section in the MikoPBX console menu</p></figcaption></figure>

4. Select **"2) Update to version..."**

{% hint style="info" %}
The **Install** option is intended for a new installation and erases the data on the selected device. To update while keeping the settings, only choose **"Update to version..."**
{% endhint %}

<figure><img src="../../../.gitbook/assets/MikoPBXConsole-UpdateTo3.40.png" alt=""><figcaption><p>Updating to version 2026.3.40</p></figcaption></figure>

5. Wait for the writing to finish and the reboot. Remove the USB drive or restore the system disk as the first boot device.

<figure><img src="../../../.gitbook/assets/UpdatedMikoPBX2026.3.40.png" alt=""><figcaption><p>The PBX successfully updated to version 2026.3.40</p></figcaption></figure>
