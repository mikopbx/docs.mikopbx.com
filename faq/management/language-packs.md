---
description: >-
  Инструкция по установке языковых пакетов для добавления новых звуков системных
  сообщений
---

# Установка языковых пакетов

По умолчанию MikoPBX поставляется с ограниченным набором языков для системных сообщений. Языковые пакеты позволяют расширить этот набор - добавить голосовые файлы на нужном языке и сделать их доступными для использования в IVR, очередях и других элементах телефонии.

В этой статье рассматривается установка языкового пакета на примере японского языка.

1. Для начала необходимо пройти процедуру регистрации в маркетплейсе MikoPBX. Руководство доступно [по ссылке](../../manual/modules/licensing.md).
2. Перейдите в раздел "**Модули**" -> "**Маркетплейс модулей**".

<figure><img src="../../.gitbook/assets/2026.1.223ModuleMarketplaceSection.png" alt=""><figcaption><p>Раздел "Маркетплейс модулей"</p></figcaption></figure>

3. В разделе "**Маркетплейс**" найдите интересующий Вас языковой пакет (в текущей документации - Японский). Установите его.

<figure><img src="../../.gitbook/assets/JapanesePackInMarketplace.png" alt=""><figcaption><p>Установка языкового пакета в маркетплейсе MikoPBX</p></figcaption></figure>

После установки языкового пакета, на диск будет загружен соответствующий набор звуков по пути `/storage/usbdisk1/mikopbx/media/sounds/ja-jp` (для японского языка).

```bash
~# cd /storage/usbdisk1/mikopbx/media/sounds/ja-jp
/storage/usbdisk1/mikopbx/media/sounds/ja-jp# ls -la
total 84432
drwxr-xr-x  8 root root  135168 Apr 28 14:20 .
drwxr-xr-x 28 root root    4096 Apr 28 14:19 ..
-rw-r--r--  1 root root    9676 Apr 28 14:20 activated.alaw
-rw-r--r--  1 root root    9676 Apr 28 14:20 activated.g722
-rw-r--r--  1 root root    1980 Apr 28 14:20 activated.gsm
-rw-r--r--  1 root root    7000 Apr 28 14:20 activated.opus
-rw-r--r--  1 root root   19246 Apr 28 14:20 activated.sln
-rw-r--r--  1 root root     493 Apr 28 14:20 .activated.sound-meta
-rw-r--r--  1 root root    9676 Apr 28 14:20 activated.ulaw
-rw-r--r--  1 root root   19212 Apr 28 14:20 activated.wav
-rw-r--r--  1 root root    7928 Apr 28 14:20 added.alaw
-rw-r--r--  1 root root    7928 Apr 28 14:20 added.g722
-rw-r--r--  1 root root    1617 Apr 28 14:20 added.gsm
-rw-r--r--  1 root root    5797 Apr 28 14:20 added.opus
-rw-r--r--  1 root root   15748 Apr 28 14:20 added.sln
-rw-r--r--  1 root root     489 Apr 28 14:20 .added.sound-meta
-rw-r--r--  1 root root    7928 Apr 28 14:20 added.ulaw
-rw-r--r--  1 root root   15714 Apr 28 14:20 added.wav
-rw-r--r--  1 root root   52772 Apr 28 14:20 agent-alreadyon.alaw
-rw-r--r--  1 root root   52772 Apr 28 14:20 agent-alreadyon.g722
-rw-r--r--  1 root root   10890 Apr 28 14:20 agent-alreadyon.gsm
-rw-r--r--  1 root root   36010 Apr 28 14:20 agent-alreadyon.opus
-rw-r--r--  1 root root  105436 Apr 28 14:20 agent-alreadyon.sln
-rw-r--r--  1 root root     499 Apr 28 14:20 .agent-alreadyon.sound-meta
-rw-r--r--  1 root root   52772 Apr 28 14:20 agent-alreadyon.ulaw
-rw-r--r--  1 root root  105402 Apr 28 14:20 agent-alreadyon.wav
-rw-r--r--  1 root root   52748 Apr 28 14:20 agent-incorrect.alaw
...
```

### Подключение звукового пакета

1. Перейдите на вкладку "**Установленные модули**". Включите установленный модуль с языковым пакетом.

<figure><img src="../../.gitbook/assets/InstalledModules-JapaneseSOundpack.png" alt=""><figcaption><p>Включение модуля с языковым пакетом</p></figcaption></figure>

2. Перейдите в раздел "**Система**" -> "**Общие настройки**".

<figure><img src="../../.gitbook/assets/GeneralSettingsTabMikoPBX.png" alt=""><figcaption><p>Раздел "Общие настройки"</p></figcaption></figure>

3. Далее на вкладке "**Основные**", в поле "**Язык звуковых сообщений системы**" выберите Ваш язык.

Нажмите "**Сохранить**".

<figure><img src="../../.gitbook/assets/chooseLanguageForTheSystem.png" alt=""><figcaption><p>Выбор языка для звуков</p></figcaption></figure>

Служба Asterisk будет перезапущена и язык звуковых сообщений будет заменен.
