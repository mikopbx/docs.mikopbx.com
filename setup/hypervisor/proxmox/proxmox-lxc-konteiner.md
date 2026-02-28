---
description: Установка MikoPBX в LXC контейнер
---

# Proxmox LXC контейнер

**Proxmox LXC** — это легковесные контейнеры в составе платформы виртуализации Proxmox VE, работающие на базе технологии LXC (Linux Containers). Они позволяют запускать изолированные Linux-системы с минимальным потреблением ресурсов по сравнению с полноценными виртуальными машинами.

### Загрузка шаблона контейнера

1. Перейдите в "**local**" хранилище, далее "**CT Templates".** Нажмите "**Download from URL**" для перехода к диалогу загрузки шаблона из URL.

<figure><img src="../../../.gitbook/assets/proxmox-CTtemplates.png" alt=""><figcaption><p>Загрузка шаблона из ссылки</p></figcaption></figure>

2. Перейдите на [Github MikoPBX](https://github.com/mikopbx/core/releases) с релизами и скопируйте ссылку на скачивание файла-шаблона с расширением "**lxc.tar.gz**".

<figure><img src="../../../.gitbook/assets/copyingLinkToLxctargz.png" alt=""><figcaption><p>Копирование ссылки на шаблон</p></figcaption></figure>

