# Закончилось место на доп. диске, увеличение размера диска

{% hint style="success" %}
Увеличить размер диска позволяют некоторый виртуальные машины
{% endhint %}

{% hint style="danger" %}
Перед работами обязательно выполните резервное копирование данных!
{% endhint %}

Для выполнения предложенных далее команд потребуется [Подключение к АТС с помощью SSH-клиента](../troubleshooting/podklyuchenie-k-ats-s-pomoshyu-ssh-klienta.md).

## Контроль свободного места на диске <a href="#kontrol_svobodnogo_mesta_na_diske" id="kontrol_svobodnogo_mesta_na_diske"></a>

```php
~ # df -h
Filesystem                Size      Used Available Use% Mounted on
none                    281.3M    324.0K    281.0M   0% /dev
/dev/sda2               392.3M    384.6M      3.8M  99% /offload
/dev/sda3                14.1M    915.0K     12.9M   6% /cf
/dev/sdb1                 4.9G     71.0M      4.5G   2% /storage/usbdisk1
```

{% hint style="success" %}
Диск для хранения данных обычно монтируется в каталог «/storage/usbdisk1». Из примера выше видно, что на текущий момент доступно 4.5G из 4.9G.
{% endhint %}

## Отключение диска <a href="#otkljuchenie_diska" id="otkljuchenie_diska"></a>

Перед началом работ следует отмонтировать диск. Для этого запустите скрипт:

```php
~ # /etc/rc/freestorage
```

Убедитесь, что диск для хранения данных более не смонтирован:

```php
~ # df -h
Filesystem                Size      Used Available Use% Mounted on
none                    281.3M    324.0K    281.0M   0% /dev
/dev/sda2               392.3M    388.3M         0 100% /offload
/dev/sda3                14.1M    915.0K     12.9M   6% /cf
```

## Редактирование таблицы разделов <a href="#redaktirovanie_tablicy_razdelov" id="redaktirovanie_tablicy_razdelov"></a>

### Удаление раздела <a href="#udalenie_razdela" id="udalenie_razdela"></a>

Для начала удалим существующий раздел. **Эта операция НЕ удаляет данные на диске**, просто правит таблицу разделов.

Запускаем редактор разделов:

```
# fdisk /dev/sdb
```

Система запросит ввести команду, вводим «d» и жмем Enter:

```
Command (m for help): d
```

Система запросит выбрать раздел к удалению, он один, вводим номер раздела «1» и жмем Enter:

```
Selected partition 1
```

Сохраняем таблицу разделов, вводим команду «w» и жмем Enter:

```
Command (m for help): w
```

### Добавление большего раздела <a href="#dobavlenie_bolshego_razdela" id="dobavlenie_bolshego_razdela"></a>

Запускаем редактор разделов:

```
# fdisk /dev/sdb
```

Система запросит ввести команду, вводим «n» и жмем Enter:

```
Command (m for help): n
```

Далее указываем команду «p», раздел будет первичным, жмем Enter:

```
Command action p
```

Вводим номер создаваемого раздела «1», жмем Enter:

```
Partition number (1-4): 1
```

Далее система запросит ввести номера первого и последнего сектора «**First sector**» / «**Last sector**», ждем Enter, ничего не вводим и соглашаемся со значениями «**по умолчанию**».

### Проверка нового раздела <a href="#proverka_novogo_razdela" id="proverka_novogo_razdela"></a>

{% hint style="warning" %}
Размер раздела должен соответствовать размеру диска.
{% endhint %}

```php
~ # fdisk -l 
Disk /dev/sdb: 10 GB, 10737418240 bytes, 20971520 sectors
1305 cylinders, 255 heads, 63 sectors/track
Units: cylinders of 16065 * 512 = 8225280 bytes

Device  Boot StartCHS    EndCHS        StartLBA     EndLBA    Sectors  Size Id Type
/dev/sdb1    0,1,1       1023,254,63         63   20964824   20964762  9.9G 83 Linux
```

## Проверка раздела на ошибки <a href="#proverka_razdela_na_oshibki" id="proverka_razdela_na_oshibki"></a>

Запускаем команду проверки:

```
e2fsck -f /dev/sdb1
```

Пример результата работы команды:

```php
e2fsck 1.43.4 (31-Jan-2017)
Pass 1: Checking inodes, blocks, and sizes
Pass 2: Checking directory structure
Pass 3: Checking directory connectivity
Pass 4: Checking reference counts
Pass 5: Checking group summary information
/dev/sdb1: 35/655360 files (11.4% non-contiguous), 63423/2620595 blocks
```

## Размер файловой системы раздела <a href="#razmer_fajlovoj_sistemy_razdela" id="razmer_fajlovoj_sistemy_razdela"></a>

Запускаем команду:

```
resize2fs /dev/sdb1
```

Пример вывода команды:

```
resize2fs 1.43.4 (31-Jan-2017)
The filesystem is already 2620595 (4k) blocks long.  Nothing to do!
```

## Перезагрузка и монтирование <a href="#perezagruzka_i_montirovanie" id="perezagruzka_i_montirovanie"></a>

При загрузке система автоматически смонтирует диск для хранения данных:

```php
~ # df -h
Filesystem                Size      Used Available Use% Mounted on
none                    281.3M    324.0K    281.0M   0% /dev
/dev/sda2               392.3M    384.6M      3.8M  99% /offload
/dev/sda3                14.1M    915.0K     12.9M   6% /cf
/dev/sdb1                 9.8G     73.3M      9.2G   1% /tmp/123
```
