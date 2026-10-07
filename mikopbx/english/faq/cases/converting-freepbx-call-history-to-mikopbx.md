# Conversion of Call History FreePBX -> MikoPBX

{% hint style="success" %}
Call history is migrated with a single universal script, **`miko-cdr-migrate.sh`**. The same file is run both on FreePBX and on MikoPBX — the station type is detected automatically. On FreePBX it exports the history into a `master.db` file; on MikoPBX it loads that history into the working `cdr.db` database.
{% endhint %}

{% hint style="info" %}
**PT1C\_cdr** is the table containing the call history from the telephony panel version 1 in FreePBX. If the module is installed, the script takes the data from it (it has the answer time, end time and `id`); otherwise it uses the native `cdr` table.
{% endhint %}

### What is migrated

* Only the **call history** (database records) is migrated.
* **Call recording files are not copied or re-encoded.** Paths to the `.mp3` files are written into the database, but the files themselves must be transferred and, if necessary, re-encoded separately — see [«Transferring call recording files»](#recordings).

### Downloading the script

The script is maintained in the MikoPBX Core repository. Download it onto your working machine:

```
curl -o miko-cdr-migrate.sh \
  'https://raw.githubusercontent.com/mikopbx/Core/develop/tools/migration/miko-cdr-migrate.sh'
```

### Step 1. On FreePBX — exporting the history <a href="#freepbx" id="freepbx"></a>

1.  Copy the script to the FreePBX station and connect to it:

    ```
    scp miko-cdr-migrate.sh root@FREEPBX_IP:/usr/src/
    ssh root@FREEPBX_IP
    ```
2.  Run the script. It reads the MySQL login and password from `/etc/freepbx.conf` on its own:

    ```
    cd /usr/src && sh miko-cdr-migrate.sh
    ```

The script detects that this is FreePBX, selects the source table (`PT1C_cdr`, or `cdr` otherwise) and creates the file `/usr/src/miko-mysql-to-sqlite/master.db`.

### Step 2. Transferring the `master.db` file

Copy the resulting `master.db` to MikoPBX into the upload directory:

```
scp root@FREEPBX_IP:/usr/src/miko-mysql-to-sqlite/master.db \
    root@MIKOPBX_IP:/storage/usbdisk1/mikopbx/freepbx-dmp/
```

### Step 3. On MikoPBX — importing <a href="#mikopbx" id="mikopbx"></a>

{% hint style="warning" %}
The import **completely clears** the `cdr_general` table of the working database before loading. On a production station, first make a trial run with `--dry-run` (it changes nothing and prints the resulting SQL). Before replacing the database the script automatically makes a `cdr.db.dmp` backup.
{% endhint %}

1.  Copy the script to MikoPBX into the upload directory and connect:

    ```
    scp miko-cdr-migrate.sh root@MIKOPBX_IP:/storage/usbdisk1/mikopbx/freepbx-dmp/
    ssh root@MIKOPBX_IP
    ```
2.  First preview what will be done, without changing anything:

    ```
    cd /storage/usbdisk1/mikopbx/freepbx-dmp && sh miko-cdr-migrate.sh --dry-run
    ```
3.  If everything looks fine, run the import:

    ```
    sh miko-cdr-migrate.sh
    ```

The script detects the station and the table inside `master.db`, makes a backup of the working database (`cdr.db.dmp`), clears and fills `cdr_general` with the correct mapping, and puts `cdr.db` back in place.

**Done!** The history will appear in the MikoPBX _web_ interface.

### Transferring call recording files <a href="#recordings" id="recordings"></a>

{% hint style="info" %}
This step is not performed by the migration script and is only needed if you want to play back recordings of old calls.
{% endhint %}

The script writes the recording file paths into the database in the MikoPBX format (`/storage/usbdisk1/mikopbx/voicemailarchive/monitor/YYYY/MM/DD/<name>.mp3`), but the files themselves stay on FreePBX. To transfer and re-encode them to MP3:

1.  On MikoPBX, prepare the list of files to download (specify your own date):

    ```
    sqlite3 /storage/usbdisk1/mikopbx/freepbx-dmp/master.db \
      'SELECT strftime("%Y/%m/%d/", calldate) || recordingfile
         FROM PT1C_cdr
        WHERE recordingfile<>"" AND calldate > "2020-02-07"' > file-for-download.txt
    ```
2.  Download the recordings from FreePBX and re-encode them to MP3 (the FreePBX address is the second argument):

    ```
    curl 'http://files.miko.ru/s/zYyMAyhpqtLZ7Qf/download' -o ./download-and-convert.sh
    sh ./download-and-convert.sh file-for-download.txt FREEPBX_IP:443
    ```

### Useful options

| Command | Purpose |
| --- | --- |
| `sh miko-cdr-migrate.sh --dry-run` | Show what will be done without changing anything (on MikoPBX prints the resulting SQL) |
| `sh miko-cdr-migrate.sh --mode=freepbx` | Force FreePBX mode, skipping auto-detection |
| `sh miko-cdr-migrate.sh --mode=mikopbx` | Force MikoPBX mode |
| `sh miko-cdr-migrate.sh --master=/path/master.db` | Non-standard path to `master.db` |
| `sh miko-cdr-migrate.sh -h` | Help |

If the paths or credentials are non-standard, they can be overridden via environment variables:

```
# FreePBX — MySQL login/password and the CDR database name
DBUSER=... DBPASS=... CDRDB=asteriskcdrdb sh miko-cdr-migrate.sh

# MikoPBX — path to the working database and to the recordings directory
MIKO_CDR_DB=/path/cdr.db MIKO_REC_BASE=/path/monitor sh miko-cdr-migrate.sh
```

### Rollback on MikoPBX

If the result is not satisfactory, restore the backup made during the import:

```
cp /storage/usbdisk1/mikopbx/astlogs/asterisk/cdr.db.dmp \
   /storage/usbdisk1/mikopbx/astlogs/asterisk/cdr.db
```
