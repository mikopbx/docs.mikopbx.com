---
description: Установка MikoPBX с помощью Selectel
---

# Selectel

В данной инструкции мы пошагово произведем установку MikoPBX с помощью облачной платформы Selectel.

Перед началом вам необходимо скачать актуальный образ MikoPBX с расширением .raw. Сделать это можно по [ссылке](https://github.com/mikopbx/Core/releases).

{% embed url="https://vkvideo.ru/video-100268702_456239044" %}

## Загрузка образа в Selectel

1. Перейдите в раздел **Облачная платформа** -> **Образы**.

<figure><img src="../../.gitbook/assets/imagesSectionSelectel.jpg" alt=""><figcaption><p>Раздел "Образы"</p></figcaption></figure>

2. Нажмите "**Создать образ**".

<figure><img src="../../.gitbook/assets/createNewImageButton.jpg" alt=""><figcaption><p>Кнопка "Создать образ"</p></figcaption></figure>

3. Укажите:

* **Имя образа** - любое желаемое название для вашего образа.
* **ОС** - Linux
* **Источник** - Файл
* **Файл** - выберите раннее загруженный файл с расширением .raw

Все остальное - **по умолчанию**.

Нажмите создать и дождитесь окончания процесса.

<figure><img src="../../.gitbook/assets/menuOfCreatingImageSelectel.jpg" alt=""><figcaption><p>Параметры загружаемого образа диска</p></figcaption></figure>

## Создание сервера в Selectel

1. Перейдите в раздел **Облачная платформа** -> **Серверы**

<figure><img src="../../.gitbook/assets/serverSectionSelctel.jpg" alt=""><figcaption><p>Раздел "Серверы"</p></figcaption></figure>

2. Нажмите "**Создать сервер**":

<figure><img src="../../.gitbook/assets/newServer.jpg" alt=""><figcaption><p>"Создать сервер"</p></figcaption></figure>

3. В конфигурации вашей ВМ укажите:

* **Имя** - произвольное название.
* **Пул** - такой же, как у раннее созданного образа.
* **Источник** - выберите раннее загруженный образ.
* **Конфигурация** - желаемое "железо" исходя из ваших потребностей.

<figure><img src="../../.gitbook/assets/confSection1.jpg" alt=""><figcaption><p>Первая секция конфигурации</p></figcaption></figure>

* **Диски**: Здесь вам необходимо указать размер для первого диска (он же - системный диск) - **5Гб** (минимально возможный в Selectel). А так же создайте новый диск, используя кнопку "**Добавить**". Укажите размер. Для диска, который используется для записи разговоров - рекомендуемое значение >**50Гб**. Типы дисков - "**Базовый HDD**"
* **Сеть** - "Приватная + 1 публичный IP"

<figure><img src="../../.gitbook/assets/confSection2 (1).jpg" alt=""><figcaption><p>Вторая секция конфигурации</p></figcaption></figure>

3. Нажмите "**Создать сервер**".

<figure><img src="../../.gitbook/assets/createNewServer.jpg" alt=""><figcaption><p>"Создать сервер"</p></figcaption></figure>

После создания, сразу остановите запуск сервера.

## Включение DHCP

1. Перейдите в раздел **Облачная платформа** -> **Сеть**.

<figure><img src="../../.gitbook/assets/networkSection.jpg" alt=""><figcaption><p>Раздел "Сеть"</p></figcaption></figure>

2. Перейдите в конфигурацию сети "Nat":

<figure><img src="../../.gitbook/assets/natNetwork.jpg" alt=""><figcaption><p>Сеть "Nat"</p></figcaption></figure>

3. Перейдите в раздел **Подсети** -> **Автоматические сетевые настройки**.

<figure><img src="../../.gitbook/assets/autoSettings.jpg" alt=""><figcaption><p>Автоматические сетевые настройк</p></figcaption></figure>

4. Включите переключатель "DHCP-сервер".

<figure><img src="../../.gitbook/assets/dhcpServer.jpg" alt=""><figcaption><p>"DHCP-сервер"</p></figcaption></figure>

## Первый запуск MikoPBX

1. Вернитесь к разделу **Облачная платформа** -> **Серверы.** Далее - в созданный серве&#x440;**.**
2. Включите сервер:

<figure><img src="../../.gitbook/assets/turnOnTheServer.jpg" alt=""><figcaption><p>Элемент переключения состояния сервера</p></figcaption></figure>

3. Перейдите в раздел "Syslog":

<figure><img src="../../.gitbook/assets/syslogSection.jpg" alt=""><figcaption><p>Раздел "Syslog"</p></figcaption></figure>

Произведите подключение по:

External IP Address - внешний IP-адрес вашей MikoPBX. Скопируйте и вставьте его в адресную строку.

Web credentials - данные для входа в WEB-интерфейс. Введите логин и пароль.

<figure><img src="../../.gitbook/assets/mikopbxlogin.jpg" alt=""><figcaption></figcaption></figure>
