---
description: Overview of methods for transferring MikoPBX to another server
---

# Migrating MikoPBX to Another Server

There are several ways to transfer MikoPBX to a different host (server). Each method has its advantages and special considerations. Below is a brief overview of each option, and you can refer to the detailed guides in this section.

## Option #1: Transfer Using Backup

**Description:**

A backup of the current MikoPBX configuration is created and then uploaded to the new server. This method is suitable for smaller amounts of data.

**Pros:**

* Easy to set up.
* Allows you to preserve the current configuration.

**Considerations:**

* May be less reliable for large amounts of data.
* Requires intermediate storage for the backup (e.g., local disk or cloud).

{% content-ref url="transfer-using-backup.md" %}
[transfer-using-backup.md](transfer-using-backup.md)
{% endcontent-ref %}

***

## Option #2: Transfer Using SFTP and Scheduled Backups

**Description:**

A backup is automatically created and saved directly to the target server via the SFTP protocol. This method is especially effective for large amounts of data.

**Pros:**

* Suitable for large amounts of data.
* Minimizes manual actions.
* Provides direct data transfer between servers.

**Considerations:**

* Requires SFTP configuration on both servers.
* Needs correct SSH user settings for proper operation.

{% content-ref url="transfer-using-scheduled-backup.md" %}
[transfer-using-scheduled-backup.md](transfer-using-scheduled-backup.md)
{% endcontent-ref %}

***

## Option #3: Transfer Using rsync

**Description:**

The `rsync` command is used to directly synchronize data between the old and new servers. This method is convenient for experienced users.

**Pros:**

* Fast synchronization, even for large data volumes.
* Preserves access rights and directory structure.
* Does not require creating intermediate backups.

**Considerations:**

* Requires basic command-line knowledge.
* Possible errors if configurations (e.g., paths) are incorrect.
* Both servers must be accessible on the network at the same time.

{% content-ref url="transfer-using-rsync.md" %}
[transfer-using-rsync.md](transfer-using-rsync.md)
{% endcontent-ref %}

***
