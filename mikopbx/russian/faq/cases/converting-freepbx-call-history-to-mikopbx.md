# Конвертация истории звонков FreePBX -> MikoPBX

{% hint style="success" %}
Перенос истории звонков выполняется одним универсальным скриптом **`miko-cdr-migrate.sh`**. Один и тот же файл запускается и на FreePBX, и на MikoPBX — тип станции определяется автоматически. На FreePBX он выгружает историю в файл `master.db`, на MikoPBX — загружает её в рабочую базу `cdr.db`.
{% endhint %}

{% hint style="info" %}
`PT1C_cdr` — таблица с историей звонков от панели телефонии версии 1 в FreePBX. Если модуль установлен, скрипт берёт данные из неё (там есть время ответа, время завершения и `id`), иначе — из родной таблицы `cdr`.
{% endhint %}

### Что переносится

* Переносится **только история звонков** (записи базы данных).
* **Файлы записей разговоров не копируются и не перекодируются.** В базу прописываются пути к `.mp3`, но сами файлы нужно перенести и при необходимости перекодировать отдельно — см. раздел [«Перенос файлов записей разговоров»](#recordings).

### Загрузка скрипта

Скрипт поддерживается в репозитории MikoPBX Core. Скачайте его на рабочую машину:

```
curl -o miko-cdr-migrate.sh \
  'https://raw.githubusercontent.com/mikopbx/Core/develop/tools/migration/miko-cdr-migrate.sh'
```

### Шаг 1. На FreePBX — экспорт истории <a href="#freepbx" id="freepbx"></a>

1.  Скопируйте скрипт на станцию FreePBX и подключитесь к ней:

    ```
    scp miko-cdr-migrate.sh root@FREEPBX_IP:/usr/src/
    ssh root@FREEPBX_IP
    ```
2.  Запустите скрипт. Логин и пароль MySQL он берёт из `/etc/freepbx.conf` сам:

    ```
    cd /usr/src && sh miko-cdr-migrate.sh
    ```

Скрипт определит, что это FreePBX, выберет таблицу-источник (`PT1C_cdr`, иначе `cdr`) и создаст файл `/usr/src/miko-mysql-to-sqlite/master.db`.

### Шаг 2. Перенос файла `master.db`

Скопируйте полученный `master.db` на MikoPBX в каталог выгрузки:

```
scp root@FREEPBX_IP:/usr/src/miko-mysql-to-sqlite/master.db \
    root@MIKOPBX_IP:/storage/usbdisk1/mikopbx/freepbx-dmp/
```

### Шаг 3. На MikoPBX — импорт <a href="#mikopbx" id="mikopbx"></a>

{% hint style="warning" %}
Импорт **полностью очищает** таблицу `cdr_general` рабочей базы перед загрузкой. Перед запуском на боевой станции сначала выполните прогон с `--dry-run` (ничего не меняет, печатает готовый SQL). Перед заменой скрипт автоматически делает бэкап `cdr.db.dmp`.
{% endhint %}

1.  Скопируйте скрипт на MikoPBX в каталог выгрузки и подключитесь:

    ```
    scp miko-cdr-migrate.sh root@MIKOPBX_IP:/storage/usbdisk1/mikopbx/freepbx-dmp/
    ssh root@MIKOPBX_IP
    ```
2.  Сначала посмотрите, что будет сделано, ничего не меняя:

    ```
    cd /storage/usbdisk1/mikopbx/freepbx-dmp && sh miko-cdr-migrate.sh --dry-run
    ```
3.  Если всё устраивает — выполните импорт:

    ```
    sh miko-cdr-migrate.sh
    ```

Скрипт определит станцию и таблицу внутри `master.db`, сделает бэкап рабочей базы (`cdr.db.dmp`), очистит и заполнит `cdr_general` с правильным маппингом и вернёт `cdr.db` на место.

**Готово!** История появится в _web_-интерфейсе MikoPBX.

### Перенос файлов записей разговоров <a href="#recordings" id="recordings"></a>

{% hint style="info" %}
Этот шаг не выполняется скриптом миграции и нужен, только если требуется прослушивать записи старых разговоров.
{% endhint %}

Пути к файлам записей скрипт прописывает в базу в формате MikoPBX (`/storage/usbdisk1/mikopbx/voicemailarchive/monitor/ГГГГ/ММ/ДД/<имя>.mp3`), но сами файлы остаются на FreePBX. Чтобы перенести и перекодировать их в MP3:

1.  На MikoPBX подготовьте список файлов к загрузке (в качестве даты укажите свою):

    ```
    sqlite3 /storage/usbdisk1/mikopbx/freepbx-dmp/master.db \
      'SELECT strftime("%Y/%m/%d/", calldate) || recordingfile
         FROM PT1C_cdr
        WHERE recordingfile<>"" AND calldate > "2020-02-07"' > file-for-download.txt
    ```
2.  Загрузите записи с FreePBX и перекодируйте их в MP3 (вторым аргументом — адрес FreePBX):

    ```
    curl 'http://files.miko.ru/s/zYyMAyhpqtLZ7Qf/download' -o ./download-and-convert.sh
    sh ./download-and-convert.sh file-for-download.txt FREEPBX_IP:443
    ```

### Полезные опции

| Команда | Назначение |
| --- | --- |
| `sh miko-cdr-migrate.sh --dry-run` | Показать, что будет сделано, ничего не меняя (на MikoPBX печатает готовый SQL) |
| `sh miko-cdr-migrate.sh --mode=freepbx` | Форсировать режим FreePBX, пропустив авто-детект |
| `sh miko-cdr-migrate.sh --mode=mikopbx` | Форсировать режим MikoPBX |
| `sh miko-cdr-migrate.sh --master=/путь/master.db` | Нестандартный путь к `master.db` |
| `sh miko-cdr-migrate.sh -h` | Справка |

Если пути или реквизиты нестандартные, их можно переопределить через переменные окружения:

```
# FreePBX — логин/пароль MySQL и имя базы CDR
DBUSER=... DBPASS=... CDRDB=asteriskcdrdb sh miko-cdr-migrate.sh

# MikoPBX — путь к рабочей базе и каталогу записей
MIKO_CDR_DB=/path/cdr.db MIKO_REC_BASE=/path/monitor sh miko-cdr-migrate.sh
```

### Откат на MikoPBX

Если результат не устроил — верните бэкап, сделанный при импорте:

```
cp /storage/usbdisk1/mikopbx/astlogs/asterisk/cdr.db.dmp \
   /storage/usbdisk1/mikopbx/astlogs/asterisk/cdr.db
```
