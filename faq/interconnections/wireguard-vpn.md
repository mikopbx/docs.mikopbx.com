---
description: Пример объединения двух АТС в приватную сеть WG.
---

# WireGuard - VPN

{% hint style="info" %}
Настроить WG возможно на АТС версии **2024.2.301-dev** и более новых сборках.&#x20;
{% endhint %}

Подключитесь к АТС по [SSH](../troubleshooting/connecting-to-a-pbx-using-ssh/). Скачайте скрипт настройки wg на сервер и на клиент:

```
cd /storage/usbdisk1/mikopbx/custom_modules
curl -o wg-configure.sh https://files.miko.ru/s/Rs9VKpzeXmmJcTC/download
```

На АТС "Клиенте" выполнить

```
sh /storage/usbdisk1/mikopbx/custom_modules/wg-configure.sh get-pubkey
```

Скопировать публичный ключ вида: "`bnJTY0HZwO6OzDrnmHKxQ`"

На "Сервере" выполнить назначение IP адрес для ключа:

```
sh /storage/usbdisk1/mikopbx/custom_modules/wg-configure.sh add-peer bnJTY0HZwO6OzDrnmHKxQ
```

Пример ответа

```
Create keys
Peer saved: IP=192.168.100.2 -> /storage/usbdisk1/mikopbx/custom_modules/wg/peers/192.168.100.2
```

Запустить сервер

```
sh /storage/usbdisk1/mikopbx/custom_modules/wg-configure.sh up-wg
```

Вызов этого скрипта можно добавить в cron через "[**Кастомизация системных файлов**](../../manual/system/custom-files.md)":

```
*/1 * * * * /bin/sh /storage/usbdisk1/mikopbx/custom_modules/wg-configure.sh up-wg > /dev/null 2>&1
```

На АТС "Сервере" выполнить

```
sh /storage/usbdisk1/mikopbx/custom_modules/wg-configure.sh get-pubkey
```

Скопировать публичный ключ вида: "`C82txdP8wh8pBztQ4Usyxw=`"

На клиенте подключиться к серверу:

```
sh /storage/usbdisk1/mikopbx/custom_modules/wg-configure.sh up-wg-client \
   192.168.100.2 \
   C82txdP8wh8pBztQ4Usyxw= \
   pbx.test.ru
```

* `192.168.100.2` - адрес клиента, назначенный на сервере командой `add-peer`
* "`C82txdP8wh8pBztQ4Usyxw=`" - публичный ключ сервера
* "`pbx.test.ru`" - публичный адрес сервера, порт всегда "`51820`"

Вызов этого скрипта можно добавить в cron через "[**Кастомизация системных файлов**](../../manual/system/custom-files.md)":

```
*/1 * * * * /bin/sh /storage/usbdisk1/mikopbx/custom_modules/wg-configure.sh up-wg-client 192.168.100.2 C82txdP8wh8pBztQ4Usyxw= pbx.test.ru > /dev/null 2>&1
```

В этом случае подключение будет подниматься автоматически при перезагрузке ПК. &#x20;

### Проверка

Выполните на клиенте и на сервере:

```
wg show
```

Пример вывода на клиенте:

```
interface: wg0-client
  public key: OCGp7zjfB1jQNLWOk1xIBk=
  private key: (hidden)
  listening port: 57731

peer: oIvFopfaQNhCDv1CAIM/F8=
  endpoint: *.*.*.*:51820
  allowed ips: 192.168.100.0/24
  latest handshake: 4 seconds ago
  transfer: 92 B received, 180 B sent
  persistent keepalive: every 25 seconds
```

Пример вывода на сервере

```
interface: wg0
  public key: oIvFopfaQNhCDv1CAIM/F8=
  private key: (hidden)
  listening port: 51820

peer: OCGp7zjfB1jQNLWOk1xIBk=
  endpoint: 158.160.179.211:57731
  allowed ips: 192.168.100.2/32
  latest handshake: 1 minute, 3 seconds ago
  transfer: 244 B received, 92 B sent
```

### Сетевой экран

Через кастомизацию системных файлов открыть порт на firewall:

{% include "../../.gitbook/includes/iptables-i-input-2-s-0.0.... (1).md" %}

* "`0.0.0.0/0`" - можно заменить на конкретную подсеть / адрес, для большей безопасности.&#x20;

В разделе "[Сетевой экран](../../manual/connectivity/firewall.md)" опишите подсеть `192.168.100.0/24` как локальную.&#x20;
