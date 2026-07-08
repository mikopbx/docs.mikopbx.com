# Moving MikoPBX Data to a New Disk

Use this guide when the local MikoPBX storage is full or when the current data storage disk must be replaced. The storage contains call recordings, call history, system logs, additional modules, backups, and system caches. In a standard installation, the storage directory is mounted at `/storage/usbdisk1`.

This guide applies to MikoPBX installed on a virtual machine or a physical server. For Docker installations, move the host directory that is mounted into the container as `/storage` using the host operating system tools.

{% hint style="danger" %}
Partitioning and formatting commands delete all data on the selected disk. Continue only if you have positively identified the new empty disk.
{% endhint %}

## Important Notes

MikoPBX does not use a manually maintained `/etc/fstab` entry as the only source of truth. Disk information is stored in the MikoPBX configuration database, and when storage is connected, the system mounts the partition by UUID and rebuilds `/etc/fstab`.

After copying data, use the standard script to switch the storage disk:

```sh
/etc/rc/connect_storage
```

Do not run `/sbin/freestorage` as the first migration step from an SSH session. This script also stops the SSH server `dropbear`, so the remote session can be disconnected.

## Requirements

* root SSH access to MikoPBX;

{% hint style="info" %}
Instructions for connecting via SSH are available [here](../troubleshooting/connecting-to-a-pbx-using-ssh/).
{% endhint %}

* a new empty disk connected to the server;
* a current backup of the configuration and important data;
* enough free space on the current storage disk for the system to keep working during the copy;
* a planned short telephony downtime while services are stopped and the PBX is rebooted.

## Identify the Current and New Disks

Connect to MikoPBX over SSH and run the diagnostic commands:

```sh
lsblk -f
blkid
df -h
mount
cat /etc/fstab
sqlite3 /cf/conf/mikopbx.db "select id,name,device,uniqid,filesystemtype,media from m_Storage;"
```

Find:

* the current storage partition mounted at `/storage/usbdisk1`;
* the new disk with no mount point, no file system, and no important data.

Example: the current storage is on `/dev/vdb1`, and the new empty disk is `/dev/vdc`.

Check the new disk again:

<pre class="language-sh"><code class="lang-sh">NEW_DISK=<a data-footnote-ref href="#user-content-fn-1">/dev/vdc</a>

lsblk -o NAME,PATH,SIZE,TYPE,FSTYPE,LABEL,UUID,MOUNTPOINTS,MODEL,SERIAL,VENDOR "$NEW_DISK"
blkid "$NEW_DISK" || true
wipefs -n "$NEW_DISK"
parted -s "$NEW_DISK" print
</code></pre>

If the disk has partitions, a file system, a UUID, a mount point, or you are not sure what the disk is used for, stop and confirm that it is really an empty disk.

## Save a Local Copy of the Configuration Database

Before switching the disk, save a copy of the database and the current `/etc/fstab`:

```sh
STAMP=$(date +%Y%m%d-%H%M%S)
cp /cf/conf/mikopbx.db "/cf/conf/mikopbx.db.before-storage-move-$STAMP"
cp /etc/fstab "/cf/conf/fstab.before-storage-move-$STAMP"
```

This copy is not a replacement for a full backup, but it helps you quickly roll back the selected storage disk.

## Stop Services That Write to Storage

Stop PBX workers and the main MikoPBX services:

```sh
pbx-console services stop-all

for svc in asterisk nginx php-fpm fail2ban rsyslogd gnatsd crond; do
    monit stop "$svc"
done

sleep 10
```

Check that no processes still keep files open on the old storage:

```sh
lsof +f -- /storage/usbdisk1
```

If the command prints any processes, do not continue. Find the remaining service, stop it, and repeat the check.

## Partition and Format the New Disk

{% hint style="danger" %}
The following commands completely erase `NEW_DISK`.
{% endhint %}

Create a GPT partition table, one partition using the whole disk, and an `ext4` file system:

```sh
parted --script --align optimal "$NEW_DISK" mklabel gpt
parted --script --align optimal "$NEW_DISK" mkpart primary ext4 0% 100%
sync
blockdev --rereadpt "$NEW_DISK" 2>/dev/null || true
partprobe "$NEW_DISK" 2>/dev/null || true

NEW_PART=$(lsblk -nr -o PATH,TYPE "$NEW_DISK" | awk '$2=="part"{print $1; exit}')
mkfs.ext4 -F "$NEW_PART"
sync

NEW_UUID=$(blkid -s UUID -o value "$NEW_PART")
echo "$NEW_PART $NEW_UUID"
```

Make sure the variables are populated:

```sh
echo "NEW_DISK=$NEW_DISK"
echo "NEW_PART=$NEW_PART"
echo "NEW_UUID=$NEW_UUID"
```

## Copy Data from the Old Storage

Mount the new partition to a temporary directory and copy the contents of the current `/storage/usbdisk1`:

```sh
mkdir -p /mnt/newstorage
mount -t ext4 UUID="$NEW_UUID" /mnt/newstorage

rsync -aHAX --numeric-ids /storage/usbdisk1/ /mnt/newstorage/
sync
```

Check the copy:

```sh
find /storage/usbdisk1 -xdev | wc -l
find /mnt/newstorage -xdev | wc -l
test -d /mnt/newstorage/mikopbx && echo "mikopbx directory OK"
test -f /mnt/newstorage/mikopbx/astlogs/asterisk/cdr.db && echo "cdr.db OK"
```

The object counts must match. If services are started before the migration is finished, the number of files can change because of logs and temporary files, so compare the counts while services are stopped.

Unmount the temporary mount point:

```sh
umount /mnt/newstorage
```

## Connect the New Disk as MikoPBX Storage

Unmount the old storage partition from `/storage/usbdisk1`:

```sh
umount /storage/usbdisk1
```

If you get `target is busy`, check which processes still use the mount point:

```sh
lsof +f -- /storage/usbdisk1
command -v fuser >/dev/null 2>&1 && fuser -vm /storage/usbdisk1
```

Stop the reported processes and repeat `umount`. If `lsof` and, when available, `fuser` show no holders but the partition is still busy, use the fallback option:

```sh
umount -l /storage/usbdisk1
```

Run the standard storage connection script and pass the new disk name without `/dev/`:

```sh
printf "%s\n" "$(basename "$NEW_DISK")" | /etc/rc/connect_storage
```

The script should find the new partition with the `mikopbx` directory, save the UUID in the MikoPBX configuration, mount it at `/storage/usbdisk1`, and rebuild `/etc/fstab`.

## Start Services

Return the services under `monit` control and restart the workers:

```sh
for svc in crond rsyslogd fail2ban php-fpm nginx gnatsd asterisk; do
    monit start "$svc"
done

pbx-console services restart-all
sleep 20
```

Check the state:

```sh
monit summary
pbx-console services status
asterisk -rx "core show uptime"
mount | grep /storage/usbdisk1
df -h /storage/usbdisk1
```

## Check Persistent Mounting After Reboot

Reboot MikoPBX:

```sh
/sbin/pbx_reboot
```

The SSH session may be disconnected. After MikoPBX boots, connect again and check:

```sh
lsblk -f
mount | grep /storage/usbdisk1
df -h /storage/usbdisk1
cat /etc/fstab
sqlite3 /cf/conf/mikopbx.db "select id,name,device,uniqid,filesystemtype,media from m_Storage;"
test -d /storage/usbdisk1/mikopbx && echo "mikopbx directory OK"
test -f /storage/usbdisk1/mikopbx/astlogs/asterisk/cdr.db && echo "cdr.db OK"
monit summary
pbx-console services status
asterisk -rx "core show uptime"
```

In a correct state:

* `/storage/usbdisk1` is mounted from the new partition;
* the UUID of the new partition is listed in `/etc/fstab`;
* the `m_Storage` table points to the new disk;
* `/storage/usbdisk1/mikopbx` and the call history database are accessible;
* the main MikoPBX services have `OK` status in `monit summary`;
* Asterisk responds to `core show uptime`.

## Rollback

If the new disk is not connected correctly and the old disk has not been wiped, you can switch back to the old storage:

```sh
pbx-console services stop-all

for svc in asterisk nginx php-fpm fail2ban rsyslogd gnatsd crond; do
    monit stop "$svc"
done

umount /storage/usbdisk1
printf "vdb\n" | /etc/rc/connect_storage

for svc in crond rsyslogd fail2ban php-fpm nginx gnatsd asterisk; do
    monit start "$svc"
done

pbx-console services restart-all
```

Replace `vdb` with the old disk name without `/dev/`.

If you need to restore the entire storage configuration record from the saved database copy:

```sh
cp /cf/conf/mikopbx.db.before-storage-move-YYYYMMDD-HHMMSS /cf/conf/mikopbx.db
/sbin/pbx_reboot
```

## Troubleshooting

**`/etc/rc/connect_storage` says the disk is already mounted.**

The old storage is still connected to `/storage/usbdisk1`. Stop the services that write to storage and run `umount /storage/usbdisk1`.

**The new disk is not listed as a suitable disk.**

Check `lsblk -f`. The disk must be visible to the system, must not be mounted elsewhere, and must be larger than 2 GB.

**`umount` returns `target is busy`.**

Use `lsof +f -- /storage/usbdisk1` and, if available, `fuser -vm /storage/usbdisk1`. Stop the processes that keep files open. Use `umount -l` only after stopping services and checking that there are no active holders.

**Storage is not mounted after reboot.**

Compare the UUID in `blkid`, `/etc/fstab`, and `m_Storage`. Then connect storage again:

```sh
printf "%s\n" "$(basename "$NEW_DISK")" | /etc/rc/connect_storage
```

**Services stay in `Initializing` or `Waiting` for a while.**

This is normal for some `monit` checks immediately after boot. Wait 1-2 minutes and repeat:

```sh
monit summary
asterisk -rx "core show uptime"
```

[^1]: Specify the path to your disk found in the previous step
