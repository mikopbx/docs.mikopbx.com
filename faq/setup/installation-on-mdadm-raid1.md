# Installation on MDADM RAID1

## Preparation

{% hint style="danger" %}
**WARNING:** All data on the disks will be erased.
{% endhint %}

1. Prepare a PC with two disks of the same size.
2. Boot the machine in[ **LiveCD** or **LiveUSB** mode](../../setup/bare-metal.md).
3. Connect to the machine via [**SSH**](../troubleshooting/connecting-to-a-pbx-using-an-ssh-client.md).

## RAID 1 Assembly

1. Run the following command in the console to display disk names:

```bash
fdisk -l
```

2. The disk names will be displayed. In this example, the disk names are:

```
/dev/sda
/dev/sdb
```

3. Clear the superblocks on the disks:

```bash
bashCopy codemdadm --zero-superblock --force /dev/sd{b,a}
```

4. Clean the old metadata:

```bash
bashCopy codewipefs --all --force /dev/sd{b,a}
```

5. Create the RAID1 array:

```bash
bashCopy codemdadm --create --metadata=0.90 --verbose /dev/md0 -l 1 -n 2 /dev/sd{b,a}
```

6. When prompted with "Continue creating array?", confirm by entering "**y**".
7. You can now proceed with the installation as per the [installation guide](../../setup/bare-metal.md). When selecting the disk during installation, choose `md0`.

### Grub

**TODO:** You may need to modify the `grub.cfg` file. Otherwise, there is no guarantee that the system will boot if one of the disks fails.
