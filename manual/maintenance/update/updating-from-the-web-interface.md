# Updating from the web interface

In some sections of the interface (e.g., **Extensions**), the current version of MikoPBX is displayed in the lower right corner.

<figure><img src="../../../.gitbook/assets/MikoPBXupgrade_eng_1.png" alt=""><figcaption></figcaption></figure>

In the PBX web interface, go to **Maintenance** → **PBX update**.

<figure><img src="../../../.gitbook/assets/MikoPBXupgrade_eng_2.png" alt=""><figcaption></figcaption></figure>

If there are newer versions of the PBX available, they will be displayed in the **Online updates** **available** table, with the version number in the first field and the list of changes in the second.

{% hint style="warning" %}
We recommend performing updates sequentially without skipping releases.
{% endhint %}

<figure><img src="../../../.gitbook/assets/MikoPBXupgrade_eng_3.png" alt=""><figcaption></figcaption></figure>

There are two update options: **online update** and **update using a downloaded img file**.

### Online upgrade

{% hint style="danger" %}
**Be cautious**! If the system is installed on the same disk where call recordings are stored, there may be difficulties with the update. [See forum](https://qa.askozia.ru/5061/%D0%BF%D1%80%D0%BE%D0%BF%D0%B0%D0%B4%D0%B0%D0%B5%D1%82-%D1%80%D0%B0%D0%B7%D0%B4%D0%B5%D0%BB-%D0%BF%D0%BE%D1%81%D0%BB%D0%B5-%D0%BE%D0%B1%D0%BD%D0%BE%D0%B2%D0%BB%D0%B5%D0%BD%D0%B8%D1%8F-%D0%BD%D0%B0-6-7-7-31)
{% endhint %}

Updates are downloaded to the PBX and applied immediately.\
To update this way, click the button ![](../../../.gitbook/assets/obnov\_ats\_3.png) for the desired version.

<figure><img src="../../../.gitbook/assets/MikoPBXupgrade_eng_4.png" alt=""><figcaption></figcaption></figure>

A warning window will appear. Click **Upgrade**.

<figure><img src="../../../.gitbook/assets/MikoPBXupgrade_eng_5.png" alt=""><figcaption></figcaption></figure>

The PBX will download and apply the updates, and then reboot.

### **Update using a downloaded img file**

{% hint style="info" %}
Please note that this method can also be used to roll back to a previous version.
{% endhint %}

To update using this method, click the button ![](../../../.gitbook/assets/obnov\_ats\_6.png) for the desired version.

<figure><img src="../../../.gitbook/assets/MikoPBXupgrade_eng_6.png" alt=""><figcaption></figcaption></figure>

The img file will start downloading. Wait for the download to complete.

Then click the button ![](../../../.gitbook/assets/obnov\_ats\_8.png) and select the downloaded img file.

<figure><img src="../../../.gitbook/assets/MikoPBXupgrade_eng_7.png" alt=""><figcaption></figcaption></figure>

Then click **Apply the update**, and in the warning window, click **Upgrade**.

<figure><img src="../../../.gitbook/assets/MikoPBXupgrade_eng_8.png" alt=""><figcaption></figcaption></figure>

The updates will be applied, and the PBX will reboot upon completion.

<figure><img src="../../../.gitbook/assets/MikoPBXupgrade_eng_9.png" alt=""><figcaption></figcaption></figure>
