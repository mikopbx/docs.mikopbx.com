---
description: Обновление MikoPBX, установленной на отдельный физический компьютер
---

# Обновление физического сервера

{% hint style="warning" %}
Рекомендуется производить обновления последовательно, «не перепрыгивая» через релизы и версии.
{% endhint %}

Эта инструкция подходит для отдельного компьютера или сервера, на котором установлена MikoPBX.

Основной способ - обновление через web-интерфейс. Если станция не загружается штатно или web-интерфейс недоступен, используйте ISO-образ, записанный на отдельный USB-носитель.

{% hint style="danger" %}
Перед обновлением обязательно создайте резервную копию настроек и сохраните её на другом устройстве. Так же сохраните копию записей разговоров.
{% endhint %}

## Подготовка

1. Убедитесь, что у вас есть физический доступ к серверу либо рабочая консоль. Создайте резервную копию настроек MikoPBX и записей разговоров.
2. Убедитесь, что в Storage свободно не менее **400 МБ**.
3. Завершите активные вызовы и запланируйте перерыв в работе телефонии.

## Обновление через web-интерфейс

### Онлайн-обновление

1. В web-интерфейсе откройте "**Обслуживание"** → "**Обновление PBX"**.

<figure><img src="../../../.gitbook/assets/PBXUpdate_Section.png" alt=""><figcaption><p>Раздел "Обновление PBX"</p></figcaption></figure>

2. Выберите желаемую версию в списке. Ознакомьтесь со списком изменений и запустите процесс обновления.

<figure><img src="../../../.gitbook/assets/updatePBXFromWeb.png" alt=""><figcaption><p>Кнопка для обновления станции из веб интерфейса</p></figcaption></figure>

3. После загрузки введите фразу **Да, у меня есть резервная копия**. Нажмите **Обновить**.

<figure><img src="../../../.gitbook/assets/MikoPBX_UpdateConfirmation.png" alt=""><figcaption><p>Подтверждение наличия резервной копии</p></figcaption></figure>

### Обновление локальным IMG-файлом

1. Скачайте `.img` нужной версии со страницы [релизов MikoPBX](https://github.com/mikopbx/Core/releases).

{% hint style="info" %}
Обратите внимание на архитектуру Вашей MikoPBX: в случае если у Вас ARM MikoPBX, то используйте файл обновления с "arm64" в названии. Если у Вас X86 MikoPBX, то используйте файл обновления с "x86\_64" в названии.
{% endhint %}

<figure><img src="../../../.gitbook/assets/MikoPBXGithubReleases-IMG.png" alt=""><figcaption><p>Загрузка .img для обновления</p></figcaption></figure>

2. Откройте **Обслуживание** → **Обновление PBX**.

<figure><img src="../../../.gitbook/assets/PBXUpdate_Section.png" alt=""><figcaption><p>Раздел "Обновление PBX"</p></figcaption></figure>

3. Выберите скачанный файл `.img`.

<figure><img src="../../../.gitbook/assets/MikoPBXUpdate_ChooseIMG.png" alt=""><figcaption><p>Выбор файла обновления</p></figcaption></figure>

4. Нажмите **Применить обновление**.

<figure><img src="../../../.gitbook/assets/MikoPBX_ApplyUdate.png" alt=""><figcaption><p>Кнопка для применения обновления</p></figcaption></figure>

4. Введите фразу **Да, у меня есть резервная копия** и подтвердите операцию.

<figure><img src="../../../.gitbook/assets/MikoPBX_UpdateConfirmation.png" alt=""><figcaption><p>Подтверждение наличии резервной копии станции</p></figcaption></figure>

Дождитесь автоматической перезагрузки станции после обновления.

<figure><img src="../../../.gitbook/assets/MikoPBXStation2026.3.40.png" alt=""><figcaption><p>Станция на новой версии</p></figcaption></figure>

## Обновление с ISO на USB-носителе

Этот способ подходит для восстановления и обновления станции с локальной консоли.

### Создание загрузочного носителя

1. Скачайте ISO нужной версии со страницы [релизов MikoPBX](https://github.com/mikopbx/Core/releases).

{% hint style="info" %}
Обратите внимание на архитектуру Вашей MikoPBX: в случае если у Вас ARM MikoPBX, то используйте файл обновления с "arm64" в названии. Если у Вас X86 MikoPBX, то используйте файл обновления с "x86\_64" в названии.
{% endhint %}

<figure><img src="../../../.gitbook/assets/MikoPBXGithubReleases-ISO.png" alt=""><figcaption></figcaption></figure>

2. Подключите отдельную USB-флешку объёмом не менее 1 ГБ.
3. Запишите ISO в режиме образа диска с помощью Rufus, balenaEtcher или `dd`. Инструкцию по записи ISO образа Вы можете найти [здесь](../../../setup/bare-metal/live-usb.md#zapis-obraza-na-usb-nositel).

{% hint style="danger" %}
При записи ISO все данные на выбранной USB-флешке будут удалены. Внимательно проверьте выбранное устройство.
{% endhint %}

### Запуск обновления

1. Подключите подготовленную флешку к серверу. В BIOS/UEFI или Boot Menu выберите загрузку с USB.
2. Дождитесь запуска MikoPBX в Recovery mode.

<figure><img src="../../../.gitbook/assets/MikoPBX-RecoveryMode3.40.png" alt=""><figcaption><p>MikoPBX в Recovery Mode</p></figcaption></figure>

3. Нажмите на любую клавишу, чтобы перейти к консольному меню. Далее откройте **"\[4] Install or recover"**.

<figure><img src="../../../.gitbook/assets/MikoPBX-3.40-InstallOrRecover.png" alt=""><figcaption><p>Раздел "Install or recover" в консольном меню MikoPBX</p></figcaption></figure>

4. Выберите **"2) Update to version...".**

{% hint style="info" %}
Пункт **Install** предназначен для новой установки и удаляет данные на выбранном устройстве. Для обновления с сохранением настроек выбирайте только "**Update to version..."**
{% endhint %}

<figure><img src="../../../.gitbook/assets/MikoPBXConsole-UpdateTo3.40.png" alt=""><figcaption><p>Обновление версии на 2026.3.40</p></figcaption></figure>

5. Дождитесь окончания записи и перезагрузки. Извлеките USB-флешку либо верните системный диск на первое место в порядке загрузки.

<figure><img src="../../../.gitbook/assets/UpdatedMikoPBX2026.3.40.png" alt=""><figcaption><p>Успешное обновление станции на версию 2026.3.40</p></figcaption></figure>
