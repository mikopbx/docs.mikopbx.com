# Установка на MDADM RAID1

## Подготовка <a href="#podgotovka" id="podgotovka"></a>

{% hint style="danger" %}
ВНИМАНИЕ: Все данные на дисках будут уничтожены.
{% endhint %}

1. Подготовьте ПК с двумя дисками одинакового объема
2. Загрузите машину в режиме [LiveCD или LiveUSB](../../setup/bare-metal.md)
3. Подключитесь к машине через [SSH](../troubleshooting/connecting-to-a-pbx-using-an-ssh-client.md)

## Сборка RAID 1 <a href="#sborka_raid_1" id="sborka_raid_1"></a>

1. Выполните в консоли команду.

```
fdisk -l 
```

2. Будут отображены имена дисков. В моем случае это.

```
/dev/sda 
/dev/sdb
```

3. Затираем суперблоки на дисках.

```
mdadm --zero-superblock --force /dev/sd{b,a} 
```

4. Чистим старые метаданные.

```
wipefs --all --force /dev/sd{b,a}
```

5. Создаем RAID1.

```
mdadm --create --metadata=0.90 --verbose /dev/md0 -l 1 -n 2 /dev/sd{b,a}
```

6. На вопрос «**Continue creating array?**» отвечаем утвердительно "**y".**
7. Далее можно начать установку [по инструкции ](../../setup/bare-metal.md). При выборе диска следует указать **md0**

### Полезные статьи <a href="#poleznye_stati" id="poleznye_stati"></a>

* [Работа с mdadm в Linux для организации RAID](https://www.dmosk.ru/miniinstruktions.php?mini=mdadm)

### Grub <a href="#grub" id="grub"></a>

{% hint style="warning" %}
**TODO**: Необходимо править **grub.cfg** файл. Иначе, не факт, что при сбое одного из дисков, система загрузится.
{% endhint %}
