# Conversion of Call History FreePBX -> MikoPBX

{% hint style="success" %}
**PT1C\_cdr** is the table containing the call history from the telephony panel version 1 in FreePBX.
{% endhint %}

### Actions in FreePBX

1.  Let’s navigate to the FreePBX working directory.

    ```
    mkdir -p /usr/src/miko-mysql-to-sqlite;
    cd /usr/src/miko-mysql-to-sqlite
    ```
2.  Download the script to convert the MySQL table to SQLite3. If it’s not available, make sure to download it.

    ```
    curl '' -o 'mysql2sqlite.sh'
    ```
3.  Run the conversion script:

    ```
     ./mysql2sqlite.sh -ufreepbxuser -p123 asteriskcdrdb PT1C_cdr | grep -v 'CREATE INDEX
    ```

### Actions in MikoPBX

{% hint style="info" %}
The archive with the scripts is available via the provided [link](https://files.miko.ru/s/0vzK6bUhcUjqk3M).
{% endhint %}

1. The file **master.db**, obtained in the previous step, must be transferred to MikoPBX in the directory: **/storage/usbdisk1/mikopbx/freepbx-dmp**.
2. Navigate to the MikoPBX working directory:

```
mkdir -p /storage/usbdisk1/mikopbx/freepbx-dmp;
cd /storage/usbdisk1/mikopbx/freepbx-dmp;
```

3. Prepare the list of files for uploading, specifying the date as **"2020-02-07"**.

```
sqlite3 master.db 'SELECT strftime("%Y/%m/%d/", calldate) || recordingfile FROM PT1C_cdr WHERE recordingfile<>"" AND calldate > "2020-02-07"' > file-for-download.txt
```

4. Uploading call recordings and converting them to MP3.

```
curl '' -o ./download-and-convert.sh
sh ./download-and-convert.sh file-for-download.txt 10.70.10.6:443
```

5. Perform a backup of the call history database.

```
cp /storage/usbdisk1/mikopbx/astlogs/asterisk/cdr.db /storage/usbdisk1/mikopbx/astlogs/asterisk/cdr.db.dmp
```

6. Copy the call history database to the current directory.

```
cp /storage/usbdisk1/mikopbx/astlogs/asterisk/cdr.db ./cdr.db 
```

7. Convert the call history to the MikoPBX format. The history in **cdr.db** will be **CLEARED**.

```php
sqlite3 << EOF
ATTACH DATABASE 'cdr.db' as 'new';
ATTACH DATABASE 'master.db' as 'old';

delete from new.cdr_general;

INSERT INTO new.cdr_general (
    src_num,
    dst_num,
    src_chan,
    dst_chan,
    start,
    answer,
    endtime,
    duration,
    billsec,
    disposition,
    UNIQUEID,
    did,
    linkedid,
    recordingfile
)
SELECT src,
       dst,
       channel,
       dstchannel,
       calldate,
       answer,
       "end",
       duration,
       billsec,
       disposition,
       linkedid || id,
       did,
       linkedid,
       "/storage/usbdisk1/mikopbx/voicemailarchive/monitor/" || strftime("%Y/%m/%d/", calldate) || recordingfile || ".mp3"
  FROM old.PT1C_cdr;

DETACH DATABASE 'new';
DETACH DATABASE 'old';
.exit
EOF
```

8. Clear the empty **recordingfile** fields in the database.

```
sqlite3 ./cdr.db 'UPDATE cdr_general SET recordingfile="" where recordingfile LIKE "%/.mp3"'
```

9. There are no call recordings before **"2020-02-07"**. Let’s clear the **recordingfile** field.

<pre><code><strong>sqlite3 ./cdr.db 'UPDATE cdr_general SET recordingfile="" where start&#x3C;"2020-02-07"'
</strong></code></pre>

10. After moving the database file.

```
cp ./cdr.db /storage/usbdisk1/mikopbx/astlogs/asterisk/cdr.db 
```

**Done**! You can now check the call history in the **web** interface.
