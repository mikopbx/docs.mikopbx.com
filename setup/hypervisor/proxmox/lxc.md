---
description: Установка MikoPBX в LXC контейнер
---

# Proxmox LXC контейнер

**Proxmox LXC** — это легковесные контейнеры в составе платформы виртуализации Proxmox VE, работающие на базе технологии LXC (Linux Containers). Они позволяют запускать изолированные Linux-системы с минимальным потреблением ресурсов по сравнению с полноценными виртуальными машинами.

{% embed url="https://vkvideo.ru/video-100268702_456239066" %}

### Загрузка шаблона контейнера

1. Перейдите в "**local**" хранилище, далее "**CT Templates".** Нажмите "**Download from URL**" для перехода к диалогу загрузки шаблона из URL.

<figure><img src="../../../.gitbook/assets/proxmox-CTtemplates.png" alt=""><figcaption><p>Загрузка шаблона из ссылки</p></figcaption></figure>

2. Перейдите на [Github MikoPBX](https://github.com/mikopbx/core/releases) с релизами и скопируйте ссылку на скачивание файла-шаблона с расширением "**lxc.tar.gz**".

<figure><img src="../../../.gitbook/assets/copyingLinkToLxctargz.png" alt=""><figcaption><p>Копирование ссылки на шаблон</p></figcaption></figure>

3. Вставьте ссылку в поле "**URL**", нажмите "**Query URL**". Если Вы скопировали правильную ссылку, то в поле "**File name**" будет подставлено название файла с расширением "**lxc.tar.gz**".

Нажмите "**Download**" для начала загрузки.

<figure><img src="../../../.gitbook/assets/proxmox-downloadFromURL.png" alt=""><figcaption><p>Загрузка шаблона из URL</p></figcaption></figure>

После окончания загрузки, Вы увидите надпись "**TASK OK**".

<figure><img src="../../../.gitbook/assets/CTimg-taskOK.png" alt=""><figcaption><p>Успешная загрузка шаблона</p></figcaption></figure>

### Создание LXC контейнера

1. Нажмите "Create CT" в правой верхней части интерфейса для создания нового контейнера.

<figure><img src="../../../.gitbook/assets/createCTbutton.png" alt=""><figcaption><p>Кнопка "Create CT" для создания нового контейнера</p></figcaption></figure>

2. Заполните все базовые параметры контейнера:

* **Hostname** - укажите название для сервиса.
* **Password** - укажите пароль для входа в Web-интерфейс MikoPBX.
* **SSH public keys** - сгенерируйте и вставьте Ваш ssh-ключ. Далее Вы сможете использовать его для подключения к станции по SSH. Подробнее про генерацию ключей и SSH подключение можно прочитать [здесь](../../../faq/troubleshooting/connecting-to-a-pbx-using-ssh/).

Нажмите "**Next**".

<figure><img src="../../../.gitbook/assets/CreateLXCGeneral.png" alt=""><figcaption><p>Базовые параметры контейнера</p></figcaption></figure>

3. Выберите ранее загруженный шаблон в разделе "**Template**".

Нажмите "**Next**".

<figure><img src="../../../.gitbook/assets/CreateLXCTemplate.png" alt=""><figcaption><p>Выбор шаблона для создаваемого контейнера</p></figcaption></figure>

4. Далее укажите размер системного диска. Рекомендуемое значение - 1 ГБ.

Нажмите "**Add**" для добавления нового диска.

<figure><img src="../../../.gitbook/assets/CreateLXCDisksP1.png" alt=""><figcaption><p>Параметры системного диска</p></figcaption></figure>

5. Укажите размер второго диска: на нем будут храниться записи разговоров. Рекомендуемый размер - не менее 50 ГБ. Так же укажите путь к диску - "**/storage**".

Нажмите "**Add**" для добавления нового диска.

<figure><img src="../../../.gitbook/assets/CreateLXCDisksP2.png" alt=""><figcaption><p>Указание параметров для второго диска</p></figcaption></figure>

6. Укажите размер третьего диска для хранения конфигурации. Рекомендуемый размер - 0.5 ГБ. Так же укажите путь к диску - "**/cf**".

Нажмите "**Next**".

<figure><img src="../../../.gitbook/assets/CreateLXCDisksP3.png" alt=""><figcaption><p>Указание парметров для третьего диска</p></figcaption></figure>

7. На следующей вкладке укажите количество ядер, которые будут использованы. Для небольшой компании можно указать 1-2 ядра (подробнее в [этой статье](../../../readme/system-requirements.md)).

Нажмите "**Next**".

<figure><img src="../../../.gitbook/assets/CreateLXCcpu.png" alt=""><figcaption><p>Параметры создаваемого контейнера (CPU)</p></figcaption></figure>

8. Далее укажите количество оперативной и Swap памяти для контейнера.

{% hint style="info" %}
Swap — это область на диске, которую система использует как дополнительную память, когда заканчивается оперативная память (RAM). Она работает значительно медленнее RAM и служит резервом, чтобы система не завершала процессы при нехватке памяти.
{% endhint %}

Нажмите "**Next**".

<figure><img src="../../../.gitbook/assets/CreateLXCmemory.png" alt=""><figcaption><p>Параметры создаваемого контейнера (Memory)</p></figcaption></figure>

9. В следующем разделе, выберите Ваши параметры сети. В нашем случае используется DHCP для получения IPv4. Firewall здесь можно не включать, но необходимо настроить его позже в MikoPBX (подробнее можно прочитать в [этой статье](../../../manual/connectivity/)).

Нажмите "Next".

<figure><img src="../../../.gitbook/assets/CreateLXCnetwork.png" alt=""><figcaption><p>Параметры создаваемого контейнера (Network)</p></figcaption></figure>

10. В разделе с настройкой параметров DNS нажмите "Next".

<figure><img src="../../../.gitbook/assets/CreateLXCdns.png" alt=""><figcaption><p>Параметры создаваемого контейнера (DNS)</p></figcaption></figure>

Вы увидите итоговую конфигурацию контейнера. Нажмите "**Finish**".

<figure><img src="../../../.gitbook/assets/CreateLXCconfirm.png" alt=""><figcaption><p>Итоговая конфигурация контейнера</p></figcaption></figure>

### Первый запуск

1. Перейдите в окно управления созданного контейнера, нажав на его название. Нажмите на кнопку "**Start**" для запуска.

<figure><img src="../../../.gitbook/assets/LXCStart.png" alt=""><figcaption><p>Процесс запуска контейнера</p></figcaption></figure>

2. Далее перейдите во вкладку "**Console**". Дождитесь загрузки системы и найдите IP-адрес Web-интерфейса.

<figure><img src="../../../.gitbook/assets/LXCConsole.png" alt=""><figcaption><p>IP-адрес web-интерфейса</p></figcaption></figure>

Введите его в адресную строку Вашего браузера. Далее произведите первую авторизацию в MikoPBX.

{% hint style="info" %}
Данные для входа:

**Login:** admin

**Password:** пароль, который Вы задали на начальном этапе создания контейнера.
{% endhint %}

<figure><img src="../../../.gitbook/assets/LXC-MikoPBXweb.png" alt=""><figcaption><p>Web-интерфейс MikoPBX</p></figcaption></figure>
