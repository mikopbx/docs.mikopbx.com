# Перенос данных MikoPBX на новый диск

Эта инструкция нужна, когда локальное хранилище MikoPBX заполнено или старый диск требуется заменить на новый. В хранилище находятся записи разговоров, история разговоров, системные логи, дополнительные модули, резервные копии и временные файлы. В обычной установке каталог хранилища смонтирован в `/storage/usbdisk1`.

Инструкция рассчитана на установку MikoPBX на виртуальную машину или физический сервер. Для Docker-установки переносите каталог, который проброшен в контейнер как `/storage`, средствами хостовой ОС.

{% hint style="danger" %}
Команды разметки и форматирования удаляют все данные на выбранном диске. Продолжайте только если вы точно определили новый свободный диск.
{% endhint %}

## Что важно знать

MikoPBX не использует постоянную ручную запись в `/etc/fstab` как единственный источник правды. Сведения о диске хранятся в базе конфигурации MikoPBX, а при подключении хранилища система монтирует раздел по UUID и пересобирает `/etc/fstab`.

Для переключения хранилища после копирования данных используйте штатный скрипт:

```sh
/etc/rc/connect_storage
```

Не запускайте `/sbin/freestorage` из SSH-сессии как первый шаг миграции: этот скрипт останавливает в том числе SSH-сервер `dropbear`, поэтому удалённое подключение может оборваться.

## Предварительные требования

* есть доступ к MikoPBX по SSH под `root`;

{% hint style="info" %}
Инструкцию для подключения по SSH Вы можете найти [здесь](../troubleshooting/connecting-to-a-pbx-using-ssh/).
{% endhint %}

* к серверу подключён новый свободный диск;
* сделана актуальная резервная копия конфигурации и важных данных;
* на диске хранилища достаточно места для временной работы системы на время копирования;
* вы готовы к короткому простою телефонии на время остановки служб и перезагрузки.

## Определите текущий и новый диск

Подключитесь к MikoPBX по SSH и выполните диагностические команды:

```sh
lsblk -f
blkid
df -h
mount
cat /etc/fstab
sqlite3 /cf/conf/mikopbx.db "select id,name,device,uniqid,filesystemtype,media from m_Storage;"
```

Найдите:

* текущий storage-раздел, смонтированный в `/storage/usbdisk1`;
* новый диск без точки монтирования, без файловой системы и без важных данных.

Пример: текущий storage находится на `/dev/vdb1`, а новый свободный диск - `/dev/vdc`.

Дополнительно проверьте новый диск:

<pre class="language-sh"><code class="lang-sh">NEW_DISK=<a data-footnote-ref href="#user-content-fn-1">/dev/vdc</a>

lsblk -o NAME,PATH,SIZE,TYPE,FSTYPE,LABEL,UUID,MOUNTPOINTS,MODEL,SERIAL,VENDOR "$NEW_DISK"
blkid "$NEW_DISK" || true
wipefs -n "$NEW_DISK"
parted -s "$NEW_DISK" print
</code></pre>

Если у диска есть разделы, файловая система, UUID, точка монтирования или вы не уверены в его назначении, остановитесь и уточните, что это действительно свободный диск.

## Сохраните локальную копию базы конфигурации

Перед переключением диска сохраните копию базы и текущего `/etc/fstab`:

```sh
STAMP=$(date +%Y%m%d-%H%M%S)
cp /cf/conf/mikopbx.db "/cf/conf/mikopbx.db.before-storage-move-$STAMP"
cp /etc/fstab "/cf/conf/fstab.before-storage-move-$STAMP"
```

Эта копия не заменяет полноценный backup, но помогает быстро откатить изменение выбора storage-диска.

## Остановите службы, которые пишут в хранилище

Остановите фоновые задачи и основные службы MikoPBX:

```sh
pbx-console services stop-all

for svc in asterisk nginx php-fpm fail2ban rsyslogd gnatsd crond; do
    monit stop "$svc"
done

sleep 10
```

Проверьте, что процессы больше не держат файлы в старом хранилище:

```sh
lsof +f -- /storage/usbdisk1
```

Если команда выводит процессы, не продолжайте перенос. Разберите, какая служба осталась активной, остановите её и повторите проверку.

## Разметьте и отформатируйте новый диск

{% hint style="danger" %}
Следующие команды полностью очищают `NEW_DISK`.
{% endhint %}

Создайте на новом диске таблицу GPT, один раздел на весь диск и файловую систему `ext4`:

```sh
parted --script --align optimal "$NEW_DISK" mklabel gpt
parted --script --align optimal "$NEW_DISK" mkpart primary ext4 0% 100%
sync
blockdev --rereadpt "$NEW_DISK" 2>/dev/null || true
partprobe "$NEW_DISK" 2>/dev/null || true

NEW_PART=$(lsblk -nr -o PATH,TYPE "$NEW_DISK" | awk '$2=="part"{print $1; exit}')
mkfs.ext4 -F "$NEW_PART"
sync

NEW_UUID=$(blkid -s UUID -o value "$NEW_PART")
echo "$NEW_PART $NEW_UUID"
```

Убедитесь, что переменные заполнены:

```sh
echo "NEW_DISK=$NEW_DISK"
echo "NEW_PART=$NEW_PART"
echo "NEW_UUID=$NEW_UUID"
```

## Скопируйте данные старого хранилища

Смонтируйте новый раздел во временный каталог и скопируйте содержимое текущего `/storage/usbdisk1`:

```sh
mkdir -p /mnt/newstorage
mount -t ext4 UUID="$NEW_UUID" /mnt/newstorage

rsync -aHAX --numeric-ids /storage/usbdisk1/ /mnt/newstorage/
sync
```

Проверьте копию:

```sh
find /storage/usbdisk1 -xdev | wc -l
find /mnt/newstorage -xdev | wc -l
test -d /mnt/newstorage/mikopbx && echo "mikopbx directory OK"
test -f /mnt/newstorage/mikopbx/astlogs/asterisk/cdr.db && echo "cdr.db OK"
```

Количество объектов должно совпадать. Если после проверки вы запускаете службы до окончания миграции, число файлов может измениться из-за логов и временных файлов, поэтому сравнение выполняйте именно при остановленных службах.

Отмонтируйте временную точку:

```sh
umount /mnt/newstorage
```

## Подключите новый диск как хранилище MikoPBX

Снимите старый storage-раздел с `/storage/usbdisk1`:

```sh
umount /storage/usbdisk1
```

Если получите `target is busy`, проверьте держателей:

```sh
lsof +f -- /storage/usbdisk1
command -v fuser >/dev/null 2>&1 && fuser -vm /storage/usbdisk1
```

Остановите найденные процессы и повторите `umount`. Если `lsof` и, при наличии, `fuser` ничего не показывают, а раздел всё равно занят, допустим крайний вариант:

```sh
umount -l /storage/usbdisk1
```

После этого запустите штатное подключение storage-диска и передайте имя нового диска без `/dev/`:

```sh
printf "%s\n" "$(basename "$NEW_DISK")" | /etc/rc/connect_storage
```

Скрипт должен найти новый раздел с каталогом `mikopbx`, сохранить UUID в конфигурации MikoPBX, смонтировать его в `/storage/usbdisk1` и пересобрать `/etc/fstab`.

## Запустите службы

Верните службы под управление `monit` и перезапустите workers:

```sh
for svc in crond rsyslogd fail2ban php-fpm nginx gnatsd asterisk; do
    monit start "$svc"
done

pbx-console services restart-all
sleep 20
```

Проверьте состояние:

```sh
monit summary
pbx-console services status
asterisk -rx "core show uptime"
mount | grep /storage/usbdisk1
df -h /storage/usbdisk1
```

## Проверьте постоянное монтирование после перезагрузки

Перезагрузите MikoPBX:

```sh
/sbin/pbx_reboot
```

SSH-сессия может оборваться. После загрузки подключитесь снова и проверьте:

```sh
lsblk -f
mount | grep /storage/usbdisk1
df -h /storage/usbdisk1
cat /etc/fstab
sqlite3 /cf/conf/mikopbx.db "select id,name,device,uniqid,filesystemtype,media from m_Storage;"
test -d /storage/usbdisk1/mikopbx && echo "mikopbx directory OK"
test -f /storage/usbdisk1/mikopbx/astlogs/asterisk/cdr.db && echo "cdr.db OK"
monit summary
pbx-console services status
asterisk -rx "core show uptime"
```

В корректном состоянии:

* `/storage/usbdisk1` смонтирован с нового раздела;
* UUID нового раздела указан в `/etc/fstab`;
* в таблице `m_Storage` указан новый диск;
* каталог `/storage/usbdisk1/mikopbx` и база истории разговоров доступны;
* основные службы MikoPBX в `monit summary` имеют статус `OK`;
* Asterisk отвечает на `core show uptime`.

## Откат

Если новый диск не подключился, а старый диск не очищался, можно вернуться на старый storage:

```sh
pbx-console services stop-all

for svc in asterisk nginx php-fpm fail2ban rsyslogd gnatsd crond; do
    monit stop "$svc"
done

umount /storage/usbdisk1
printf "vdb\n" | /etc/rc/connect_storage

for svc in crond rsyslogd fail2ban php-fpm nginx gnatsd asterisk; do
    monit start "$svc"
done

pbx-console services restart-all
```

Замените `vdb` на имя старого диска без `/dev/`.

Если нужно вернуть всю запись конфигурации storage из сохранённой копии базы:

```sh
cp /cf/conf/mikopbx.db.before-storage-move-YYYYMMDD-HHMMSS /cf/conf/mikopbx.db
/sbin/pbx_reboot
```

## Частые проблемы

**`/etc/rc/connect_storage` пишет, что диск уже смонтирован.**

Старое хранилище ещё подключено к `/storage/usbdisk1`. Остановите пишущие службы и выполните `umount /storage/usbdisk1`.

**Новый диск не появляется в списке подходящих дисков.**

Проверьте `lsblk -f`. Диск должен быть виден системе, не должен быть смонтирован как другой каталог и должен быть больше 2 ГБ.

**После `umount` появляется `target is busy`.**

Используйте `lsof +f -- /storage/usbdisk1` и, если команда есть в системе, `fuser -vm /storage/usbdisk1`. Остановите процессы, которые держат файлы. `umount -l` применяйте только после остановки служб и проверки, что активных держателей не осталось.

**После перезагрузки storage не смонтировался.**

Сравните UUID в `blkid`, `/etc/fstab` и `m_Storage`. Затем повторите подключение:

```sh
printf "%s\n" "$(basename "$NEW_DISK")" | /etc/rc/connect_storage
```

**Службы долго находятся в `Initializing` или `Waiting`.**

Сразу после boot это нормально для части проверок `monit`. Подождите 1-2 минуты и повторите:

```sh
monit summary
asterisk -rx "core show uptime"
```

[^1]: Укажите путь к Вашему диску, найденный на прошлом шаге
