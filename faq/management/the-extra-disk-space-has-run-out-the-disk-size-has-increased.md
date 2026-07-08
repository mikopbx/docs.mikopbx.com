# Increasing the MikoPBX Storage Disk Size

Use this guide when the disk that contains the local MikoPBX storage has been expanded in the hypervisor or on the physical server, but the `/storage/usbdisk1` partition still shows the old size.

The local storage contains call recordings, call history, system logs, additional modules, backups, and system caches. In a standard installation, the storage is mounted at `/storage/usbdisk1`.

{% hint style="danger" %}
Before changing partitions, create a MikoPBX backup and a virtual machine snapshot or disk backup using your hypervisor tools.
{% endhint %}

{% hint style="warning" %}
You can run diagnostics over SSH. Run the actual resize operation from the local console, hypervisor console, or serial console. The standard resize script calls `/sbin/freestorage`, which stops the SSH server `dropbear`, so the SSH session will be disconnected.
{% endhint %}

## When to Use This Guide

Use this scenario if:

* MikoPBX is installed on a virtual machine or a physical server;
* the storage disk has already been expanded at the hypervisor, RAID controller, or storage subsystem level;
* `df -h /storage/usbdisk1` still shows the old file system size;
* the disk has unallocated space after the current storage partition.

This guide does not apply to Docker installations. In Docker, the size of the `/storage` directory is managed by the host operating system and volume settings.

## Check the Current Storage Disk

Connect to MikoPBX over SSH for diagnostics only, or open the MikoPBX console and run:

{% hint style="info" %}
Instructions for connecting via SSH are available [here](../troubleshooting/connecting-to-a-pbx-using-ssh/).
{% endhint %}

```sh
lsblk -f
df -h /storage/usbdisk1
mount | grep /storage/usbdisk1
cat /etc/fstab
sqlite3 /cf/conf/mikopbx.db "select id,name,device,uniqid,filesystemtype,media from m_Storage;"
```

Find the disk listed in the `m_Storage` table. For example:

```text
1|Storage №1|/dev/vdc|854d0b18-608a-4634-b910-dea3726ae1a5|ext4|1
```

In this example, the storage disk is `/dev/vdc`, and the storage partition is `/dev/vdc1`.

Save the disk name to a variable:

```sh
STORAGE_DISK=$(sqlite3 /cf/conf/mikopbx.db "select device from m_Storage where media='1' limit 1;")
echo "$STORAGE_DISK"
```

Check the disk size and unallocated free space:

```sh
blockdev --getsize64 "$STORAGE_DISK"
parted -s "$STORAGE_DISK" unit MiB print free
```

After the disk has been expanded in the hypervisor, the output must show that the disk itself is larger and that `Free Space` appears after the existing partition.

Example state before resizing:

```text
Disk /dev/vdc: 12288MiB

Number  Start     End       Size      File system  Name     Flags
        0.02MiB   1.00MiB   0.98MiB   Free Space
 1      1.00MiB   10239MiB  10238MiB  ext4         primary
        10239MiB  12288MiB  2049MiB   Free Space
```

If there is no free space, first expand the disk in the hypervisor or check that you selected the correct disk.

## Fix GPT After Expanding the Disk

On GPT disks, after expanding a virtual disk, `parted` may show this warning:

```text
Warning: Not all of the space available to /dev/vdc appears to be used
```

In this case, first move the backup GPT header to the end of the new disk size:

```sh
sgdisk -e "$STORAGE_DISK"
sync
blockdev --rereadpt "$STORAGE_DISK" 2>/dev/null || true
partprobe "$STORAGE_DISK" 2>/dev/null || true
```

Then repeat the check:

```sh
parted -s "$STORAGE_DISK" unit MiB print free
```

The `Not all of the space...` warning should disappear. The free space after the partition should still be visible.

## Run Storage Resize

Open the local MikoPBX console or the virtual machine console.

In the console menu, select:

```text
Storage -> Resize storage
```

Confirm stopping processes:

```text
All processes will be completed. Continue? (y/n):
```

Enter `y`.

MikoPBX will run the standard scenario:

* stop services that use storage;
* unmount `/storage/usbdisk1`;
* expand the storage partition to the end of the disk;
* run `e2fsck`;
* run `resize2fs`;
* reboot the system.

If you need to run the same scenario as a command from the console, use:

```sh
/etc/rc/resize_storage_part "$STORAGE_DISK"
```

{% hint style="warning" %}
Do not run this command from an SSH session. During execution, `dropbear` will be stopped and the connection will be disconnected.
{% endhint %}

## Check the Result After Reboot

After MikoPBX boots, identify the storage disk again and run the checks:

```sh
STORAGE_DISK=$(sqlite3 /cf/conf/mikopbx.db "select device from m_Storage where media='1' limit 1;")
echo "$STORAGE_DISK"

lsblk -f
df -h /storage/usbdisk1
parted -s "$STORAGE_DISK" unit MiB print free
mount | grep /storage/usbdisk1
cat /etc/fstab
sqlite3 /cf/conf/mikopbx.db "select id,name,device,uniqid,filesystemtype,media from m_Storage;"
```

In a correct state:

* `/storage/usbdisk1` is mounted from the same storage partition;
* the size of `/storage/usbdisk1` has increased;
* there is no free space after the storage partition, or only a minimal technical gap remains;
* the UUID in `/etc/fstab` matches the UUID of the storage partition;
* the `m_Storage` table points to the same storage disk.

Example result after a successful resize:

```text
Filesystem                Size      Used Available Use% Mounted on
/dev/vdc1                11.7G      1.1G     10.0G  10% /storage/usbdisk1
```

Check that the data is accessible:

```sh
test -d /storage/usbdisk1/mikopbx && echo "mikopbx directory OK"
test -f /storage/usbdisk1/mikopbx/astlogs/asterisk/cdr.db && echo "cdr.db OK"
sqlite3 /storage/usbdisk1/mikopbx/astlogs/asterisk/cdr.db "pragma integrity_check;"
```

The correct response for the call history database is:

```text
ok
```

Check services:

```sh
monit summary
pbx-console services status
asterisk -rx "core show uptime"
```

Immediately after reboot, some `monit` checks may be in the `Initializing` state. Wait 1-2 minutes and repeat the check.

## If the Size Did Not Change

If `Resize storage` reported success but `df -h /storage/usbdisk1` still shows the old size, check GPT and free space:

```sh
STORAGE_DISK=$(sqlite3 /cf/conf/mikopbx.db "select device from m_Storage where media='1' limit 1;")
parted -s "$STORAGE_DISK" unit MiB print free
```

If the `Not all of the space...` warning is present, run:

```sh
sgdisk -e "$STORAGE_DISK"
sync
blockdev --rereadpt "$STORAGE_DISK" 2>/dev/null || true
partprobe "$STORAGE_DISK" 2>/dev/null || true
```

Then run the resize operation again from the console:

```sh
/etc/rc/resize_storage_part "$STORAGE_DISK"
```

If the free space is less than 5% of the disk size, the standard script may finish without resizing. In this case, check whether the disk was actually expanded enough.

## Rollback

If the system does not boot or storage is not mounted after resizing, use the backup or snapshot created before the operation.

If MikoPBX boots but storage is not mounted, check the UUID and database record:

```sh
blkid
cat /etc/fstab
sqlite3 /cf/conf/mikopbx.db "select id,name,device,uniqid,filesystemtype,media from m_Storage;"
mount | grep /storage/usbdisk1
```

If needed, connect the storage disk again using the standard script:

```sh
printf "%s\n" "$(basename "$STORAGE_DISK")" | /etc/rc/connect_storage
```
