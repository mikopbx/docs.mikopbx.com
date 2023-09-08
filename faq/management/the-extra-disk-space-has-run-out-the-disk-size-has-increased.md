# The extra disk space has run out, the disk size has increased

{% hint style="success" %}
Some virtual machines allow you to increase the disk size
{% endhint %}

{% hint style="warning" %}
Be sure to back up your data before you work!
{% endhint %}

To execute the following commands, you will need to [connect to the PBX using an SSH client](../troubleshooting/connecting-to-a-pbx-using-an-ssh-client.md).

## Control of free disk space

```php
~ # df -h
Filesystem                Size      Used Available Use% Mounted on
none                    281.3M    324.0K    281.0M   0% /dev
/dev/sda2               392.3M    384.6M      3.8M  99% /offload
/dev/sda3                14.1M    915.0K     12.9M   6% /cf
/dev/sdb1                 4.9G     71.0M      4.5G   2% /storage/usbdisk1
```

{% hint style="success" %}
The data storage disk is usually mounted in the "/storage/usbdisk1" directory. From the example above, it can be seen that 4.5G of 4.9G is currently available.
{% endhint %}

## Disabling the disk

Before starting work, you should unmount the disk. To do this, run the script:

```
~ # /sbin/freestorage
```

Make sure that the data storage disk is no longer mounted:

```php
~ # df -h
Filesystem                Size      Used Available Use% Mounted on
none                    281.3M    324.0K    281.0M   0% /dev
/dev/sda2               392.3M    388.3M         0 100% /offload
/dev/sda3                14.1M    915.0K     12.9M   6% /cf
```

## Editing the Partition table

### Deleting a partition

First, delete the existing partition. This operation **does NOT delete data on the disk**, just edits the partition table.

Launching the Section Editor:

```
# fdisk /dev/sdb
```

The system will prompt you to enter a command, enter "d" and press Enter:

```
Command (m for help): d
```

Система запросит выбрать раздел к удалению, он один, вводим номер раздела «1» и жмем Enter:

```
Selected partition 1
```

Сохраняем таблицу разделов, вводим команду «w» и жмем Enter:

```
Command (m for help): w
```

### Adding a larger section

Launching the Section Editor:

```
# fdisk /dev/sdb
```

The system will prompt you to enter a command, enter "n" and press Enter:

```
Command (m for help): n
```

Next, specify the command "p", the section will be primary, press Enter:

```
Command action p
```

Enter the number of the created section "1", press Enter:

```
Partition number (1-4): 1
```

Next, the system will ask you to enter the numbers of the first and last sector "**First sector" / "Last sector**", wait for Enter, do not enter anything and agree with the "**default**" values.

### Checking a new partition

{% hint style="warning" %}
The size of the partition must match the size of the disk.
{% endhint %}

```php
~ # fdisk -l 
Disk /dev/sdb: 10 GB, 10737418240 bytes, 20971520 sectors
1305 cylinders, 255 heads, 63 sectors/track
Units: cylinders of 16065 * 512 = 8225280 bytes

Device  Boot StartCHS    EndCHS        StartLBA     EndLBA    Sectors  Size Id Type
/dev/sdb1    0,1,1       1023,254,63         63   20964824   20964762  9.9G 83 Linux
```

### Checking the section for errors

Run the verification command:

```
e2fsck -f /dev/sdb1
```

Example of the result of the team's work:

```php
e2fsck 1.43.4 (31-Jan-2017)
Pass 1: Checking inodes, blocks, and sizes
Pass 2: Checking directory structure
Pass 3: Checking directory connectivity
Pass 4: Checking reference counts
Pass 5: Checking group summary information
/dev/sdb1: 35/655360 files (11.4% non-contiguous), 63423/2620595 blocks
```

### Partition file system size

Run the command:

```
resize2fs /dev/sdb1
```

Example of command output:

```php
resize2fs 1.43.4 (31-Jan-2017)
The filesystem is already 2620595 (4k) blocks long.  Nothing to do!
```

### Rebooting and mounting

When booting, the system will automatically mount a disk for data storage:

```php
~ # df -h
Filesystem                Size      Used Available Use% Mounted on
none                    281.3M    324.0K    281.0M   0% /dev
/dev/sda2               392.3M    384.6M      3.8M  99% /offload
/dev/sda3                14.1M    915.0K     12.9M   6% /cf
/dev/sdb1                 9.8G     73.3M      9.2G   1% /tmp/123
```
