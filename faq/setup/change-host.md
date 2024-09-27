# Перенос MikoPBX на другой сервер

## Вариант №1 <a href="#variant_1" id="variant_1"></a>

1. Выполните **Резервное копирование** Вашей текущей MikoPBX по[ инструкции](../../modules/miko/module-backup.md).
2. Реализуйте сброс всех активных **настроек лицензирования** с текущей АТС по [инструкции](https://wiki.mikopbx.ru/licensing#sbros\_i\_ustanovka\_aktivnyx\_privjazok\_licenzii).
3. Установите MikoPBX на **новый сервер**. Выполните действия, указанные в [инструкции](https://wiki.mikopbx.ru/quickstart\_lancher): [Шаг 1](https://wiki.mikopbx.ru/quickstart\_lancher#shag\_1\_ustanovka\_askozia\_pbx) и[ Шаг 2](https://wiki.mikopbx.ru/quickstart\_lancher#shag\_2\_podkljuchenie\_diska\_dlja\_xranenija\_zapisej\_razgovorov).
4. **Загрузите** ранее выполненный бекап конфигурации MikoPBX на новом сервере по [инструкции](https://wiki.mikopbx.ru/module-backup#vosstanovlenie\_iz\_arxiva).
5. Выполните настройки лицензирования по [инструкции](https://wiki.mikopbx.ru/licensing#aktivacija\_registracionnogo\_nomera).

### Вариант №2 <a href="#variant_2" id="variant_2"></a>

**Если размер данных большой**, то имеет смысл сохранить резервную копию сразу на целевую машину. Порядок действий:

1. Устанавливаем MikoPBX на новый ПК
2. На основной MikoPBX настраиваем «[Резервное копирование по расписанию](https://wiki.mikopbx.ru/module-backup#rezervnoe\_kopirovanie\_po\_raspisaniju)»
3. Подключение должно производиться по **SFTP**
4. **Имя пользователя** и **пароль** те, что используются для [подключения по ssh](https://wiki.mikopbx.ru/general-settings#ssh)
5. «**Путь на сервере**» укажите «**/storage/usbdisk1/mikopbx/backup/**»
6. Дождитесь завершения резервного копирования
7. **Отключите основную машину**
8. На целевой машине выполните [восстановление из резервной копии](https://wiki.mikopbx.ru/module-backup#vosstanovlenie\_iz\_arxiva)

### Вариант №3 <a href="#variant_3" id="variant_3"></a>

Скрипт для переноса данных вручную.

```
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
# Копирование SSH ключей, чтобы не вводить многократно пароль.
#rsaPubKey="$HOME/.ssh/id_rsa.pub";
#if [ ! -f "$rsaPubKey" ];then
#  dropbearkey -y -f /etc/dropbear/dropbear_rsa_host_key | grep "^ssh-rsa" > "$rsaPubKey";
#fi;
#ssh "$PBX_HOST" -p "$SSH_PORT" "echo >> /root/.ssh/authorized_keys && echo '$(cat "$rsaPubKey")' >> /root/.ssh/authorized_keys && nohup sh -c 'killall dropbear && /usr/sbin/dropbear -p $SSH_PORT' 2>&1 &";
#ssh "$PBX_HOST" -p "$SSH_PORT" 'sqlite3 /cf/conf/mikopbx.db .tables';

# Дамп.
ssh "$PBX_HOST" -p "$SSH_PORT" "sqlite3 $CONF_DB_FILE .dump > $STORAGE_PBX_DIR/tmp/mikopbx.db.dmp";
# Копируем дамп на локальную машину.
"$SYNC_APP" "$PBX_HOST":"$STORAGE_PBX_DIR/tmp/mikopbx.db.dmp" "$STORAGE_PBX_DIR/tmp/mikopbx.db.dmp"
# Восстанавливаем базу данных из дампа.
sqlite3 "$STORAGE_PBX_DIR/tmp/mikopbx.db" < "$STORAGE_PBX_DIR/tmp/mikopbx.db.dmp";
# Перемещаем восстановленную базу данных.
mv "$STORAGE_PBX_DIR/tmp/mikopbx.db" "$CONF_DB_FILE";
# Чистим временные файлы.
rm -rf "$STORAGE_PBX_DIR/tmp/mikopbx.db";
ssh "$PBX_HOST" -p "$SSH_PORT" "rm -rf $STORAGE_PBX_DIR/tmp/mikopbx.db";

# Отключаем провайдеров на локальной машине.
sqlite3 "$CONF_DB_FILE" "UPDATE m_Sip SET disabled='1' WHERE type='friend'"

# Копируем медиа файлы с удаленной машины.
"$SYNC_APP" -r "$PBX_HOST":"$STORAGE_PBX_DIR"/media/* "$STORAGE_PBX_DIR"/media
# Копируем дополнительные модули с удаленной машины.
"$SYNC_APP" -r "$PBX_HOST":"$STORAGE_PBX_DIR"/custom_modules/* "$STORAGE_PBX_DIR"/custom_modules

# Копируем историю звонков.
# Дамп.
ssh "$PBX_HOST" -p "$SSH_PORT" "sqlite3 $STORAGE_PBX_DIR/astlogs/asterisk/cdr.db .dump > $STORAGE_PBX_DIR/astlogs/asterisk/cdr.db.dmp";
# Копируем дамп на локальную машину.
"$SYNC_APP" -r "$PBX_HOST":"$STORAGE_PBX_DIR"/astlogs/asterisk/cdr.db.dmp "$STORAGE_PBX_DIR"/astlogs/asterisk/cdr.db.dmp;
# Восстанавливаем базу данных из дампа.
sqlite3 "$STORAGE_PBX_DIR"/astlogs/asterisk/cdrdb.tmp < "$STORAGE_PBX_DIR"/astlogs/asterisk/cdr.db.dmp;
# Удаляем текущую базу данных истории звонков.
rm -rf "$STORAGE_PBX_DIR"/astlogs/asterisk/cdr.db*;
# Перемещаем восстановленную базу данных.
mv "$STORAGE_PBX_DIR"/astlogs/asterisk/cdrdb.tmp "$STORAGE_PBX_DIR"/astlogs/asterisk/cdr.db;

# Чистим временные файлы.
rm -rf "$STORAGE_PBX_DIR"/astlogs/asterisk/cdr.db.dmp;
ssh "$PBX_HOST" -p "$SSH_PORT" "rm -rf $STORAGE_PBX_DIR/astlogs/asterisk/cdr.db.dmp";

# Обновление структуры баз данных sqlite3.
php -r 'require_once "Globals.php"; use MikoPBX\Core\System\Upgrade\UpdateDatabase; $dbUpdater = new UpdateDatabase(); $dbUpdater->updateDatabaseStructure();'

# Копирование записей разговоров.
"$SYNC_APP" -r "$PBX_HOST":"$STORAGE_PBX_DIR"/astspool/monitor "$STORAGE_PBX_DIR"/astspool/monitor
```
