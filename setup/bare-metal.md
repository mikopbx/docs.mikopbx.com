# Bare-metal installation
# Standalone Computer

{% hint style="info" %}
Tested on:

* Intel NUC DCCP847DYE
* Intel NUC D54250WUKH
{% endhint %}

Most modern PCs support booting from a USB device.  
**MikoPBX** can be run from a USB device.

{% hint style="danger" %}
**Note!** The minimum capacity of the USB drive is **600MB**
{% endhint %}

{% hint style="success" %}
* The **Bootable USB** mode is designed for running the PBX from a USB drive (flash drive). Use the **.img** file for installation.
* The **Live USB** mode is intended for system installation or recovery. Use the **.iso** file for installation.
{% endhint %}

### Using Windows <a href="#ispolzuem_windows" id="ispolzuem_windows"></a>

To create a bootable USB drive, we recommend using the **imageUSB** application. You can download it [here](http://www.osforensics.com/tools/write-usb-images.html). Alternatively, use [**balenaEtcher**](https://www.balena.io/etcher/).

1. Download and install the application.  
2. Run **ImageUSB**

<figure><img src="../.gitbook/assets/ImageUSBStartWindow.png" alt=""><figcaption><p>ImageUSB start window</p></figcaption></figure>

3. Perform the "Refresh drives" action. Select the **USB drive**, then select the image file. Perform the "Write" action.

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption><p>Guide for writing the image to a flash drive</p></figcaption></figure>

4. Wait for the writing process to complete, then connect the **USB** drive to the PC. Restart the PC to boot from the drive.

### Using OSX <a href="#ispolzuem_osx" id="ispolzuem_osx"></a>

{% hint style="danger" %}
Be careful when selecting the device for formatting. Changes are irreversible!
{% endhint %}

1. Open the **Terminal** application.
2. Connect the USB drive.
3. Run the following command:
   
```
diskutil list
```


Information about **all** connected drives will be displayed.

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption><p>List of all connected drives</p></figcaption></figure>

In this example, the name of the USB device is "**/dev/disk3**." Compare the output of the `diskutil list` command before and after connecting the device.

4. Format the drive. You will need to enter the administrator password.

```
sudo diskutil eraseDisk FAT32 NONAME  MBRFormat /dev/disk3;
```


5. Unmount the device with the following command:

```
sudo diskutil unmountDisk /dev/disk3;
```

6. Write the image to the **USB drive**:
   
```
sudo dd if=1.0.64-9.0-svn-mikopbx-x86-64-cross-linux.img of=/dev/disk3 bs=1m;
```

Wait for the writing process to complete, then connect the **USB** drive to the PC. Restart the PC to boot from the drive.
