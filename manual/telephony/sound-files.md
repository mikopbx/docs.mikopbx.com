---
description: Adding/Creating Audio Files in MikoPBX
---

# Sound files

## Uploading a sound file to the PBX

{% hint style="warning" %}
Supported file formats: **wav**, **mp3**, **ogg**, **m4a**, **aac**.
{% endhint %}

Audio files in MikoPBX are used in various call scenarios and interactive voice menus **(**&#x49;n **IVR menu**, during **non-working hours**, in **call queues**, for various system notifications, and in **hold music**.) to play voice greetings or notify customers.

The list of available sound files is displayed in the "**Telephony"** -> "**Sound files".**

<figure><img src="../../.gitbook/assets/1 (8).png" alt=""><figcaption><p>"<strong>Sound files</strong>" section</p></figcaption></figure>

To add a new sound file, click "**Add a new sound file"**.

<figure><img src="../../.gitbook/assets/2 (16).png" alt=""><figcaption><p>"add a new sound file" button</p></figcaption></figure>

Click "**Upload a new file**" and select a sound file.

<figure><img src="../../.gitbook/assets/3 (3).png" alt=""><figcaption><p>"Upload a new file" button</p></figcaption></figure>

Correct the file name if necessary.

<figure><img src="../../.gitbook/assets/4 (22).png" alt=""><figcaption><p>The name of the recording file</p></figcaption></figure>

Save settings

<figure><img src="../../.gitbook/assets/5 (3).png" alt=""><figcaption><p>"<strong>Save settings</strong>" button</p></figcaption></figure>

{% hint style="info" %}
When working over the **https** protocol, it is possible to record an audio file using a microphone.
{% endhint %}

<figure><img src="../../.gitbook/assets/6 (13).png" alt=""><figcaption><p>"<strong>Start recording</strong>" button</p></figcaption></figure>

Custom sound files are stored on the PBX along the path **/storage/usbdisk1/mikopbx/media/custom**. Music on hold files are stored in **/storage/usbdisk1/mikopbx/media/moh**.

## Music on hold <a href="#muzyka_na_uderzhanii" id="muzyka_na_uderzhanii"></a>

If a client gets into a queue during a call or is waiting for redirection, the PBX plays a melody for him. It is possible to download your own tunes for listening while waiting.&#x20;

You can do this on the "**Music on Hold"** tab as described above.

<figure><img src="../../.gitbook/assets/7 (8).png" alt=""><figcaption><p>"Music on hold" tab</p></figcaption></figure>
