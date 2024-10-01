# Конвертация истории звонков FreePBX -> MikoPBX

{% hint style="success" %}
PT1C\_cdr это таблица с историей звонков от панели телефонии версии 1 в FreePBX.
{% endhint %}

### Действия в  FreePBX <a href="#freepbx" id="freepbx"></a>

1.  Перейдем в рабочий каталог FreePBX

    ```
    mkdir -p /usr/src/miko-mysql-to-sqlite;
    cd /usr/src/miko-mysql-to-sqlite
    ```
2.  Скачаем скрипт конвертации таблицы MySQL в Sqlite3. Скачать, если его нет.

    ```
    curl '' -o 'mysql2sqlite.sh'
    ```
3.  Запускаем скрипт конвертации:

    ```
     ./mysql2sqlite.sh -ufreepbxuser -p123 asteriskcdrdb PT1C_cdr | grep -v 'CREATE INDEX' | sqlite3 ./master.db
    ```

### Действия в MikoPBX <a href="#mikopbx" id="mikopbx"></a>

{% hint style="info" %}
Архив со скриптами доступен по [ссылке](https://files.miko.ru/s/0vzK6bUhcUjqk3M)
{% endhint %}

1. Файл **master.db**, полученный на предыдущем этапе необходимо перенести на MIKOPBX в каталог «**/storage/usbdisk1/mikopbx/freepbx-dmp**»
2.  Перейти в рабочий каталог MIKOPBX:

    ```
    mkdir -p /storage/usbdisk1/mikopbx/freepbx-dmp;
    cd /storage/usbdisk1/mikopbx/freepbx-dmp;
    ```
3.  Подготовка списка файлов к загрузке в качестве даты следует указать свою «2020-02-07»:

    ```
    sqlite3 master.db 'SELECT strftime("%Y/%m/%d/", calldate) || recordingfile FROM PT1C_cdr WHERE recordingfile<>"" AND calldate > "2020-02-07"' > file-for-download.txt
    ```
4.  Загрузка записей разговоров и конвертация в MP3

    ```
    curl '' -o ./download-and-convert.sh
    sh ./download-and-convert.sh file-for-download.txt 10.70.10.6:443
    ```
5.  Выполним резервное копирование базы данных истории звонков:

    ```
    cp /storage/usbdisk1/mikopbx/astlogs/asterisk/cdr.db /storage/usbdisk1/mikopbx/astlogs/asterisk/cdr.db.dmp
    ```
6.  Скопируем базу данных истории звонков в текущий каталог:

    ```
    cp /storage/usbdisk1/mikopbx/astlogs/asterisk/cdr.db ./cdr.db 
    ```
7.  Конвертация истории звонков в формат MIKOPBX. История в cdr.db будет ОЧИЩЕНА.

    ```
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
8.  Очистка пустых полей recordingfile в базе данных

    ```
    sqlite3 ./cdr.db 'UPDATE cdr_general SET recordingfile="" where recordingfile LIKE "%/.mp3"'
    ```
9.  До «2020-02-07» нет записей разговоров. Очистим поле recordingfile

    <pre><code><strong>sqlite3 ./cdr.db 'UPDATE cdr_general SET recordingfile="" where start&#x3C;"2020-02-07"'
    </strong></code></pre>
10. Переместив файл базы данных.

    ```
    cp ./cdr.db /storage/usbdisk1/mikopbx/astlogs/asterisk/cdr.db 
    ```

**Готово!** Можно проверять историю звонков в _web_ интерфейсе.
