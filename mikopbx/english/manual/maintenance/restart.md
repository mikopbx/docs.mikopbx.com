---
description: Description of section functions
---

# Reboot

## Rebooting the station via the MikoPBX interface

The system shutdown/reboot menu can be found in MikoPBX by clicking on "**Reboot**" in the "**Maintenance**" section.

<figure><img src="../../.gitbook/assets/1 (9).png" alt=""><figcaption><p>"Maintenance" -> "Reboot" section</p></figcaption></figure>

When you open the page, a list of active calls to the PBX will be displayed. The start date of the call is displayed, **from** whom and **to** whom the call.

{% hint style="warning" %}
As long as there are active calls, the reboot and shutdown will not be available via the web interface.
{% endhint %}

<figure><img src="../../.gitbook/assets/2 (8).png" alt=""><figcaption><p>List of active calls</p></figcaption></figure>

* **Restart the PBX** - the command starts restarting the station.
* **Turn off PBX** - completes all processes and disconnects the station.

<figure><img src="../../.gitbook/assets/3 (8).png" alt=""><figcaption><p>System shutdown/reboot options</p></figcaption></figure>

## Rebooting the station via the console menu

You can restart the station via the console menu. To do this, select the section **"\[3] Reboot the system"**

<figure><img src="../../.gitbook/assets/4 (15).png" alt=""><figcaption><p>MikoPBX console</p></figcaption></figure>

* If you want to restart the station: press **"\[1] Reboot MikoPBX"**
* If you want to turn off the station: press **"\[2] Shutdown"**

The system will reboot.

<figure><img src="../../.gitbook/assets/5 (13).png" alt=""><figcaption><p>Restart/shutdown station</p></figcaption></figure>

## Reboot with disk check

In case of an emergency restart of the PBX (for example, power outage), it may be necessary to check the disk for errors.

In the PBX console menu, enter the command **"\[9] Console(Shell)"** and press **Enter.**

System launch the MikoPBX console.

<figure><img src="../../.gitbook/assets/6 (3).png" alt=""><figcaption><p>Console menu MikoPBX</p></figcaption></figure>

Enter the command `reboot`. Press **Enter**.

The system will reboot with a disk check.

<figure><img src="../../.gitbook/assets/7 (10).png" alt=""><figcaption><p>Reboot command</p></figcaption></figure>
