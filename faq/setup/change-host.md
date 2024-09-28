# Migrating MikoPBX to Another Server

## Option №1 <a href="#variant_1" id="variant_1"></a>

1. Perform a **Backup** of your current MikoPBX following [this guide](../../modules/miko/module-backup.md).
2. Reset all active **licensing settings** on the current PBX following [this guide](https://wiki.mikopbx.ru/licensing#sbros\_i\_ustanovka\_aktivnyx\_privjazok\_licenzii).
3. Install MikoPBX on the **new server**. Follow the steps in [this guide](https://wiki.mikopbx.ru/quickstart\_lancher): [Step 1](https://wiki.mikopbx.ru/quickstart\_lancher#shag\_1\_ustanovka\_askozia\_pbx) and [Step 2](https://wiki.mikopbx.ru/quickstart\_lancher#shag\_2\_podkljuchenie\_diska\_dlja\_xranenija\_zapisej\_razgovorov).
4. **Upload** the previously created MikoPBX backup to the new server following [this guide](https://wiki.mikopbx.ru/module-backup#vosstanovlenie\_iz\_arxiva).
5. Set up licensing on the new server following [this guide](https://wiki.mikopbx.ru/licensing#aktivacija\_registracionnogo\_nomera).

## Option №2 <a href="#variant_2" id="variant_2"></a>

**If the data size is large**, it's practical to save the backup directly on the target machine. Here's the process:

1. Install MikoPBX on the new machine.
2. On the original MikoPBX, set up [Scheduled Backups](https://wiki.mikopbx.ru/module-backup#rezervnoe\_kopirovanie\_po\_raspisaniju).
3. Ensure the connection is via **SFTP**.
4. **Username** and **password** should be those used for [SSH connection](https://wiki.mikopbx.ru/general-settings#ssh).
5. Set the **Path on the server** to **"/storage/usbdisk1/mikopbx/backup/"**.
6. Wait for the backup to complete.
7. **Shut down the original machine**.
8. On the target machine, perform a [Restore from Backup](https://wiki.mikopbx.ru/module-backup#vosstanovlenie\_iz\_arxiva).

## Option №3 <a href="#variant_3" id="variant_3"></a>

Manual data migration script.

```bash
#
# MikoPBX - free phone system for small business
# Copyright © 2017-2024 Alexey Portnov and Nikolay Beketov
#
# This program is free software: you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; either version 3 of the License, or
# (at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License along with this program.
# If not, see <>.
#

SSH_PORT=22;
PBX_HOST='root@PBX-ADDRESS-OR-IP';
CONF_DB_FILE='/cf/conf/mikopbx.db';
STORAGE_PBX_DIR='/storage/usbdisk1/mikopbx';
SYNC_APP='rsync';
type "$SYNC_APP" > /dev/null 2> /dev/null
if [ "$?" = '1' ];then
  SYNC_APP='scp';
fi;

# TODO
# Copy SSH keys to avoid multiple password entries.
#rsaPubKey="$HOME/.ssh/id_rsa.pub";
#if [ ! -f "$rsaPubKey" ];then
#  dropbearkey -y -f /etc/dropbear/dropbear_rsa_host_key | grep "^ssh-rsa" > "$rsaPubKey";
#fi;
#ssh "$PBX_HOST" -p "$SSH_PORT" "echo >> /root/.ssh/authorized_keys && echo '$(cat "$rsaPubKey")' >> /root/.ssh/authorized_keys && nohup sh -c 'killall dropbear && /usr/sbin/dropbear -p $SSH_PORT' 2>&1 &";
#ssh "$PBX_HOST" -p "$SSH_PORT" 'sqlite3 /cf/conf/mikopbx.db .tables';

# Dump.
ssh "$PBX_HOST" -p "$SSH_PORT" "sqlite3 $CONF_DB_FILE .dump > $STORAGE_PBX_DIR/tmp/mikopbx.db.dmp";
# Copy dump to local machine.
"$SYNC_APP" "$PBX_HOST":"$STORAGE_PBX_DIR/tmp/mikopbx.db.dmp" "$STORAGE_PBX_DIR/tmp/mikopbx.db.dmp"
# Restore the database from dump.
sqlite3 "$STORAGE_PBX_DIR/tmp/mikopbx.db" < "$STORAGE_PBX_DIR/tmp/mikopbx.db.dmp";
# Move the restored database.
mv "$STORAGE_PBX_DIR/tmp/mikopbx.db" "$CONF_DB_FILE";
# Clean up temporary files.
rm -rf "$STORAGE_PBX_DIR/tmp/mikopbx.db";
ssh "$PBX_HOST" -p "$SSH_PORT" "rm -rf $STORAGE_PBX_DIR/tmp/mikopbx.db";

# Disable providers on the local machine.
sqlite3 "$CONF_DB_FILE" "UPDATE m_Sip SET disabled='1' WHERE type='friend'"

# Copy media files from the remote machine.
"$SYNC_APP" -r "$PBX_HOST":"$STORAGE_PBX_DIR"/media/* "$STORAGE_PBX_DIR"/media
# Copy custom modules from the remote machine.
"$SYNC_APP" -r "$PBX_HOST":"$STORAGE_PBX_DIR"/custom_modules/* "$STORAGE_PBX_DIR"/custom_modules

# Copy call history.
# Dump.
ssh "$PBX_HOST" -p "$SSH_PORT" "sqlite3 $STORAGE_PBX_DIR/astlogs/asterisk/cdr.db .dump > $STORAGE_PBX_DIR/astlogs/asterisk/cdr.db.dmp";
# Copy dump to local machine.
"$SYNC_APP" -r "$PBX_HOST":"$STORAGE_PBX_DIR"/astlogs/asterisk/cdr.db.dmp "$STORAGE_PBX_DIR"/astlogs/asterisk/cdr.db.dmp;
# Restore the call history database from dump.
sqlite3 "$STORAGE_PBX_DIR"/astlogs/asterisk/cdrdb.tmp < "$STORAGE_PBX_DIR"/astlogs/asterisk/cdr.db.dmp;
# Remove the current call history database.
rm -rf "$STORAGE_PBX_DIR"/astlogs/asterisk/cdr.db*;
# Move the restored database.
mv "$STORAGE_PBX_DIR"/astlogs/asterisk/cdrdb.tmp "$STORAGE_PBX_DIR"/astlogs/asterisk/cdr.db;

# Clean up temporary files.
rm -rf "$STORAGE_PBX_DIR"/astlogs/asterisk/cdr.db.dmp;
ssh "$PBX_HOST" -p "$SSH_PORT" "rm -rf $STORAGE_PBX_DIR/astlogs/asterisk/cdr.db.dmp";

# Update SQLite3 database structure.
php -r 'require_once "Globals.php"; use MikoPBX\Core\System\Upgrade\UpdateDatabase; $dbUpdater = new UpdateDatabase(); $dbUpdater->updateDatabaseStructure();'

# Copy call recordings.
"$SYNC_APP" -r "$PBX_HOST":"$STORAGE_PBX_DIR"/astspool/monitor "$STORAGE_PBX_DIR"/astspool/monitor
```
