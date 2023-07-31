# Reboot

## Rebooting the station via the MikoPBX interface

The system shutdown/reboot menu can be found in MikoPBX by clicking on "**Reboot**" in the "**Maintenance**" section

<figure><img src="../../.gitbook/assets/1 (9).png" alt=""><figcaption></figcaption></figure>

When you open the page, a list of active calls to the PBX will be displayed. The start date of the call is displayed, **from** whom and **to** whom the call

<figure><img src="../../.gitbook/assets/2 (8).png" alt=""><figcaption></figcaption></figure>

* **Restart the PBX** - the command starts restarting the station.&#x20;
* **Turn off PBX** - completes all processes and disconnects the station.

<figure><img src="../../.gitbook/assets/3 (8).png" alt=""><figcaption></figcaption></figure>

{% hint style="warning" %}
As long as there are active calls, the reboot and shutdown will not be available via the web interface.
{% endhint %}

## Rebooting the station via the console menu

You can restart the station via the console menu. To do this, select the section **"\[3] Reboot the system"**

<figure><img src="../../.gitbook/assets/4 (16).png" alt=""><figcaption></figcaption></figure>

If you want to restart the station: press **"\[1] Reboot MikoPBX"**

If you want to turn off the station: press **"\[2] Shutdown"**

<figure><img src="../../.gitbook/assets/5 (17).png" alt=""><figcaption></figcaption></figure>

The system will reboot.

## Reboot with disk check

In case of an emergency restart of the PBX (for example, power outage), it may be necessary to check the disk for errors.

In the PBX console menu, enter the command **"\[9] Console(Shell)"** and press **Enter**

<figure><img src="../../.gitbook/assets/6 (3).png" alt=""><figcaption></figcaption></figure>

System launch the Miko PBX console

Enter the command `reboot`

Press **Enter**.

<figure><img src="../../.gitbook/assets/7 (10).png" alt=""><figcaption></figcaption></figure>

The system will reboot with a disk check.
