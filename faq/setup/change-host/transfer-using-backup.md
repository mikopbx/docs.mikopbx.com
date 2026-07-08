---
description: A method to transfer MikoPBX to another host using backup
---

# Transfer Using Backup

This method involves creating a backup of your current MikoPBX configuration, transferring it, and restoring it on the new server. It’s simple to implement and suitable for small systems. This approach is convenient for users with minimal technical experience.

1. First, create a backup of your previous system. You can find detailed instructions in [this article](../../../modules/miko/module-backup.md).

<figure><img src="../../../.gitbook/assets/createBackupCopy (2).png" alt=""><figcaption><p>Creating a new backup copy</p></figcaption></figure>

2. Select the data you want to transfer and wait for the process to complete.

<figure><img src="../../../.gitbook/assets/selectingData.png" alt=""><figcaption><p>Selecting data for backup copy</p></figcaption></figure>

3. Download your archive by clicking the corresponding button in the **"Backup Module"** section:

<figure><img src="../../../.gitbook/assets/selectingData2.png" alt=""><figcaption><p>Button to download archive</p></figcaption></figure>

4. On the new host (server) with your MikoPBX installation, restore from the archive by clicking **"Upload backup file"**:

<figure><img src="../../../.gitbook/assets/uploadBackupFile.png" alt=""><figcaption><p>"<strong>Upload backup file</strong>" button</p></figcaption></figure>

After this, your system will be restored from the archive. This method is ideal for transferring small amounts of data.
