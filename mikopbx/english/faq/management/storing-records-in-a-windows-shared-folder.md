# Storing Recordings in a Shared Windows Folder

In some cases, it is necessary to save call recordings on a network drive. This example shows how to connect a shared Windows folder to MikoPBX.

{% hint style="warning" %}
**Note**: If the network folder becomes unavailable, it may cause disruptions in PBX operation.
{% endhint %}

1. Create a directory for the script:

```bash
mkdir /storage/usbdisk1/mikopbx/custom_modules/shared-folder-script
```

2. Create the script file:

```
/cat > /storage/usbdisk1/mikopbx/custom_modules/shared-folder-script/mount-shared-folder.sh
```

3. Insert the script content:

```php
#!/bin/sh
HOST='//172.16.32.38/SharedFolder';
USER='';
PASS='';
REC_DIR='autorecords';
mountPoint='/storage/win-shared';

createLink ()
{
  subDir="$(date '+%Y/%m')";
  monitorDir="$(/bin/busybox grep monitordir < /etc/inc/mikopbx-settings.json | /bin/busybox cut -f 4 -d '"')";
  if [ "${monitorDir}x" = 'x' ];then
    echo "Empty monitor dir";
    return;
  fi;
  /bin/busybox mkdir -p "$monitorDir" "$mountPoint/$REC_DIR/$subDir";
  if [ ! -L "$monitorDir/$subDir" ] && [ -d "$monitorDir/$subDir" ];then
    cp -r "$monitorDir/$subDir/"*  "$mountPoint/$REC_DIR/$subDir";
    rm -rf "${monitorDir:?}/$subDir/";
    ln -s "$mountPoint/$REC_DIR/$subDir" "${monitorDir:?}/$subDir";
  fi;

  if [ ! -L "$monitorDir/$subDir" ] && [ ! -f "${monitorDir:?}/$subDir" ]; then
    ln -s "$mountPoint/$REC_DIR/$subDir" "${monitorDir:?}/$subDir";
  fi;
}

/bin/busybox mount | /bin/busybox grep "$HOST";
resGrep="$?";
if [ "$resGrep" = "0" ]; then
  echo "Disk is mounted..."
  createLink;
  exit 2;
fi;

mkdir -p "$mountPoint";
/bin/busybox mount -t cifs "$HOST" "$mountPoint" -o "username=$USER,password=$PASS,vers=2.0"
resMount="$?";

if [ "$resMount" != '0' ];then
  echo "Error mount $HOST"
  exit 1;
fi;

createLink;

```

4. Press "CTRL+D" twice to finish creating the file.
5. Grant execution permissions:

```
chmod +x /storage/usbdisk1/mikopbx/custom_modules/shared-folder-script/mount-shared-folder.sh
```

6. In the script variables "**HOST, USER, PASS**", specify the connection parameters to the shared folder.
7. Add the script to cron for automatic connection of the shared folder.
8. Go to the **System** → **Customizing System Files** section.
9. Add the following rule to the end of the **/var/spool/cron/crontabs/root** file:

```
*/1 * * * * /storage/usbdisk1/mikopbx/custom_modules/shared-folder-script/mount-shared-folder.sh > /dev/null 2> /dev/null
```

Test the PBX operation to ensure that call recordings are being saved to the network drive.
