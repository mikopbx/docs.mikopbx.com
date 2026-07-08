# Увеличение размера диска хранилища MikoPBX

Эта инструкция нужна, когда диск, на котором находится локальное хранилище MikoPBX, был увеличен в гипервизоре или на физическом сервере, но раздел `/storage/usbdisk1` всё ещё показывает старый размер.

В локальном хранилище находятся записи разговоров, история разговоров, системные логи, дополнительные модули, резервные копии и системные кеши. В обычной установке хранилище смонтировано в `/storage/usbdisk1`.

{% hint style="danger" %}
Перед изменением разделов обязательно сделайте резервную копию MikoPBX и снимок виртуальной машины или backup диска средствами гипервизора.
{% endhint %}

{% hint style="warning" %}
Диагностику можно выполнить по SSH. Само изменение размера запускайте из локальной консоли, консоли гипервизора или serial console. Штатный resize-скрипт вызывает `/sbin/freestorage`, который останавливает SSH-сервер `dropbear`, поэтому SSH-сессия будет разорвана.
{% endhint %}

## Когда применять инструкцию

Используйте этот сценарий, если:

* MikoPBX установлена на виртуальную машину или физический сервер;
* диск хранилища уже увеличен на уровне гипервизора, RAID-контроллера или дисковой подсистемы;
* в `df -h /storage/usbdisk1` размер файловой системы остался прежним;
* на диске есть неразмеченная область после текущего storage-раздела.

Для Docker-установок эта инструкция не подходит. В Docker размер каталога `/storage` управляется хостовой ОС и настройками volume.

## Проверьте текущий диск хранилища

Подключитесь к MikoPBX по SSH только для диагностики или откройте консоль MikoPBX и выполните:

{% hint style="info" %}
Инструкцию для подключения по SSH Вы можете найти [здесь](../troubleshooting/connecting-to-a-pbx-using-ssh/).
{% endhint %}

```sh
lsblk -f
df -h /storage/usbdisk1
mount | grep /storage/usbdisk1
cat /etc/fstab
sqlite3 /cf/conf/mikopbx.db "select id,name,device,uniqid,filesystemtype,media from m_Storage;"
```

Определите диск, который указан в таблице `m_Storage`. Например:

```text
1|Storage №1|/dev/vdc|854d0b18-608a-4634-b910-dea3726ae1a5|ext4|1
```

В этом примере диск хранилища - `/dev/vdc`, а раздел хранилища - `/dev/vdc1`.

Сохраните имя диска в переменную:

```sh
STORAGE_DISK=$(sqlite3 /cf/conf/mikopbx.db "select device from m_Storage where media='1' limit 1;")
echo "$STORAGE_DISK"
```

Проверьте размер диска и свободную неразмеченную область:

```sh
blockdev --getsize64 "$STORAGE_DISK"
parted -s "$STORAGE_DISK" unit MiB print free
```

После увеличения диска в гипервизоре вывод должен показывать, что сам диск стал больше, а после существующего раздела появилась область `Free Space`.

Пример состояния до resize:

```text
Disk /dev/vdc: 12288MiB

Number  Start     End       Size      File system  Name     Flags
        0.02MiB   1.00MiB   0.98MiB   Free Space
 1      1.00MiB   10239MiB  10238MiB  ext4         primary
        10239MiB  12288MiB  2049MiB   Free Space
```

Если свободной области нет, сначала увеличьте диск в гипервизоре или проверьте, что выбран правильный диск.

## Исправьте GPT после увеличения диска

На GPT-дисках после увеличения виртуального диска может появиться предупреждение `parted`:

```text
Warning: Not all of the space available to /dev/vdc appears to be used
```

В этом случае сначала перенесите резервный GPT-заголовок в конец нового размера диска:

```sh
sgdisk -e "$STORAGE_DISK"
sync
blockdev --rereadpt "$STORAGE_DISK" 2>/dev/null || true
partprobe "$STORAGE_DISK" 2>/dev/null || true
```

Затем повторите проверку:

```sh
parted -s "$STORAGE_DISK" unit MiB print free
```

Предупреждение `Not all of the space...` должно исчезнуть. Свободная область после раздела при этом должна остаться видимой.

## Запустите изменение размера хранилища

Откройте локальную консоль MikoPBX или консоль виртуальной машины.

В консольном меню выберите:

```text
Хранилище -> Изменить размер хранилища
```

Подтвердите остановку процессов:

```text
Все процессы будут завершены. Продолжить? (y/n):
```

Ответьте `y`.

MikoPBX выполнит штатный сценарий:

* остановит службы, которые используют хранилище;
* отмонтирует `/storage/usbdisk1`;
* увеличит storage-раздел до конца диска;
* выполнит `e2fsck`;
* выполнит `resize2fs`;
* перезагрузит систему.

Если нужно запустить тот же сценарий командой из консоли, используйте:

```sh
/etc/rc/resize_storage_part "$STORAGE_DISK"
```

{% hint style="warning" %}
Эту команду не следует запускать из SSH-сессии. Во время выполнения будет остановлен `dropbear`, и соединение оборвётся.
{% endhint %}

## Проверьте результат после перезагрузки

После загрузки MikoPBX снова определите storage-диск и выполните проверки:

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

В корректном состоянии:

* `/storage/usbdisk1` смонтирован с прежнего storage-раздела;
* размер `/storage/usbdisk1` увеличился;
* свободной области после storage-раздела больше нет или остался только технический минимум;
* UUID в `/etc/fstab` совпадает с UUID storage-раздела;
* в таблице `m_Storage` указан тот же диск хранилища.

Пример результата после успешного resize:

```text
Filesystem                Size      Used Available Use% Mounted on
/dev/vdc1                11.7G      1.1G     10.0G  10% /storage/usbdisk1
```

Проверьте доступность данных:

```sh
test -d /storage/usbdisk1/mikopbx && echo "mikopbx directory OK"
test -f /storage/usbdisk1/mikopbx/astlogs/asterisk/cdr.db && echo "cdr.db OK"
sqlite3 /storage/usbdisk1/mikopbx/astlogs/asterisk/cdr.db "pragma integrity_check;"
```

Для базы истории разговоров корректный ответ:

```text
ok
```

Проверьте службы:

```sh
monit summary
pbx-console services status
asterisk -rx "core show uptime"
```

Сразу после перезагрузки часть проверок `monit` может быть в состоянии `Initializing`. Подождите 1-2 минуты и повторите проверку.

## Если размер не изменился

Если после выполнения `Изменить размер хранилища` скрипт сообщил об успехе, но `df -h /storage/usbdisk1` показывает старый размер, проверьте GPT и свободную область:

```sh
STORAGE_DISK=$(sqlite3 /cf/conf/mikopbx.db "select device from m_Storage where media='1' limit 1;")
parted -s "$STORAGE_DISK" unit MiB print free
```

Если есть предупреждение `Not all of the space...`, выполните:

```sh
sgdisk -e "$STORAGE_DISK"
sync
blockdev --rereadpt "$STORAGE_DISK" 2>/dev/null || true
partprobe "$STORAGE_DISK" 2>/dev/null || true
```

После этого снова запустите изменение размера из консоли:

```sh
/etc/rc/resize_storage_part "$STORAGE_DISK"
```

Если свободная область меньше 5% от размера диска, штатный скрипт может завершиться без изменения размера. В этом случае проверьте, действительно ли диск был увеличен достаточно заметно.

## Откат

Если после изменения размера система не загружается или хранилище не монтируется, используйте backup или snapshot, сделанный перед работами.

Если MikoPBX загружается, но storage не смонтирован, проверьте UUID и запись в базе:

```sh
blkid
cat /etc/fstab
sqlite3 /cf/conf/mikopbx.db "select id,name,device,uniqid,filesystemtype,media from m_Storage;"
mount | grep /storage/usbdisk1
```

При необходимости подключите storage-диск заново штатным скриптом:

```sh
printf "%s\n" "$(basename "$STORAGE_DISK")" | /etc/rc/connect_storage
```
