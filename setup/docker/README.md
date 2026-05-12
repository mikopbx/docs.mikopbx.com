---
description: Варианты установки MikoPBX в Docker контейнере
---

# В Docker контейнере

{% hint style="danger" %}
«**Host система**» должна быть запущена на базе Linux 5+. Тестировалось на Debian 11 и Ubuntu-21.04, Ubuntu Server 22.04 LTS
{% endhint %}

MikoPBX можно запустить в Docker, используя два основных способа. Первый способ включает в себя запуск контейнера напрямую через команду Docker с указанием необходимых параметров. Второй способ предусматривает использование Docker Compose, что упрощает управление многоконтейнерными приложениями и позволяет описать всю конфигурацию в yaml-файле, что делает развертывание и обслуживание системы более удобным.

<table data-view="cards"><thead><tr><th></th><th></th><th data-hidden data-card-cover data-type="files"></th><th data-hidden data-card-target data-type="content-ref"></th></tr></thead><tbody><tr><td><strong>Установка докера и cоздание пользователя и каталогов</strong></td><td>Команды для установки Docker и Docker Compose и настройка перед созданием контейнера</td><td><a href="../../.gitbook/assets/install.png">install.png</a></td><td><a href="docker-installation.md">docker-installation.md</a></td></tr><tr><td><strong>Запуск MikoPBX в контейнере</strong></td><td>Инструкции по запуску готового контейнера MikoPBX, созданию контейнера из произвольного образа и его запуску</td><td><a href="../../.gitbook/assets/docker.png">docker.png</a></td><td><a href="running-mikopbx-in-container.md">running-mikopbx-in-container.md</a></td></tr><tr><td><strong>Запуск MikoPBX с помощью docker compose</strong></td><td>Инструкции по запуску нескольких MikoPBX на одном хосте с помощью docker compose</td><td><a href="../../.gitbook/assets/compose.png">compose.png</a></td><td><a href="running-mikopbx-using-docker-compose.md">running-mikopbx-using-docker-compose.md</a></td></tr><tr><td><strong>Внешний файрвол для Docker</strong></td><td>Как защитить веб-интерфейс при `userland-proxy=true`: внешний bouncer или host-режим.</td><td></td><td><a href="external-firewall-enforcement.md">external-firewall-enforcement.md</a></td></tr></tbody></table>
