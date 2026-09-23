# Хранение записей в общей папке windows

В ряде случаев необходимо предусмотреть сохранение записей разговоров на сетевой диск. В этом примере покажем как подключить к MikoPBX общий каталог windows.

{% hint style="warning" %}
**Обратите внимание**: если сетевой каталог будет не доступен, возможны сбои в работе АТС.
{% endhint %}

1. Создадим каталог для хранения скрипта

<pre><code><strong>mkdir /storage/usbdisk1/mikopbx/custom_modules/shared-folder-script
</strong></code></pre>

2. Создадим файл скрипта

```
cat > /storage/usbdisk1/mikopbx/custom_modules/shared-folder-script/mount-shared-folder.sh
```

3. Вставьте содержимое скрипта

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

4. Нажмите сочетание клавиш «CTRL+D» два раза для завершения создания файла
5. Предоставьте права на исполнение

```
chmod +x /storage/usbdisk1/mikopbx/custom_modules/shared-folder-script/mount-shared-folder.sh
```

6. В переменных скрипта «**HOST,USER,PASS**» следует описать параметры подключения к общему каталогу
7. Скрипт необходимо доавить в cron для автоматического подключения общей папки
8. Перейдите в раздел «**Система**» - «**Кастомизация системных файлов**»
9. Добавьте в конец файла «**/var/spool/cron/crontabs/root**» следующее правило

```
*/1 * * * * /storage/usbdisk1/mikopbx/custom_modules/shared-folder-script/mount-shared-folder.sh > /dev/null 2> /dev/null
```

10. Протестируйте работу АТС, убедитесь, что записи разговоров сохраняются на сетевой диск.
