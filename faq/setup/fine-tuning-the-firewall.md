# Тонкая настройка firewall

{% hint style="warning" %}
Эта страница применима к bare-metal и LXC. Для Docker-инсталляций см.
[Внешний файрвол для Docker](../../setup/docker/external-firewall-enforcement.md).
{% endhint %}

При публикации АТС на публичном IP адресе возникает задача по защите АТС от сканеров, вредителей, кто пытается подобрать пароли к SIP учетным записям АТС. Если установлен простой числовой пароль, то он будет подобран очень быстро, что повлечет убытки.

Для базовой защиты от сканеров обязательно следует включить fail2ban. Дополнительно, можно более тонко настроить правила iptables.

1. Перейдите в раздел "**Кастомизация системных файлов**"

<figure><img src="../../.gitbook/assets/CustomizationSystemFiles.png" alt=""><figcaption><p>Раздел "Кастомизация системных файлов"</p></figcaption></figure>

2. Перейдите к редактированию файла **/etc/firewall\_additional**

<figure><img src="../../.gitbook/assets/firewall_additionslFile (1).png" alt=""><figcaption><p>Файл "/etc/firewall_additional" </p></figcaption></figure>

3. Установите режим "**Добавлять в конец файла**", вставьте следующий код:

```php
iptables -I INPUT 2 -p udp -m udp --dport 5060 -m string --string 'friendly-scanner' --algo bm --to 65535 -j DROP
```

<figure><img src="../../.gitbook/assets/CodeForfirewall_additionslFile.png" alt=""><figcaption><p>Код для файла  "/etc/firewall_additional"</p></figcaption></figure>

{% hint style="warning" %}
Добавленное правило позволит блокировать все входящие запросы по UDP протоколу, которые содержат подстроку «**friendly-scanner**»
{% endhint %}

Более полный пример набора правил:

```
iptables -I INPUT 2 -p udp -m udp --dport 5060 -m string --string 'sipcli' --algo bm --to 65535 -j DROP
iptables -I INPUT 2 -p udp -m udp --dport 5060 -m string --string 'sip-scan' --algo bm --to 65535 -j DROP
iptables -I INPUT 2 -p udp -m udp --dport 5060 -m string --string 'iWar' --algo bm --to 65535 -j DROP
iptables -I INPUT 2 -p udp -m udp --dport 5060 -m string --string 'sipvicious' --algo bm --to 65535 -j DROP
iptables -I INPUT 2 -p udp -m udp --dport 5060 -m string --string 'sipsak' --algo bm --to 65535 -j DROP
iptables -I INPUT 2 -p udp -m udp --dport 5060 -m string --string 'sundayddr' --algo bm --to 65535 -j DROP
iptables -I INPUT 2 -p udp -m udp --dport 5060 -m string --string 'VaxSIPUserAgent' --algo bm --to 65535 -j DROP
iptables -I INPUT 2 -p udp -m udp --dport 5060 -m string --string 'friendly-scanner' --algo bm --to 65535 -j DROP

iptables -I INPUT 2 -p tcp -m tcp --dport 5060 -m string --string 'sipcli' --algo bm --to 65535 -j DROP
iptables -I INPUT 2 -p tcp -m tcp --dport 5060 -m string --string 'sip-scan' --algo bm --to 65535 -j DROP
iptables -I INPUT 2 -p tcp -m tcp --dport 5060 -m string --string 'iWar' --algo bm --to 65535 -j DROP
iptables -I INPUT 2 -p tcp -m tcp --dport 5060 -m string --string 'sipvicious' --algo bm --to 65535 -j DROP
iptables -I INPUT 2 -p tcp -m tcp --dport 5060 -m string --string 'sipsak' --algo bm --to 65535 -j DROP
iptables -I INPUT 2 -p tcp -m tcp --dport 5060 -m string --string 'sundayddr' --algo bm --to 65535 -j DROP
iptables -I INPUT 2 -p tcp -m tcp --dport 5060 -m string --string 'VaxSIPUserAgent' --algo bm --to 65535 -j DROP
iptables -I INPUT 2 -p tcp -m tcp --dport 5060 -m string --string 'friendly-scanner' --algo bm --to 65535 -j DROP
```

Это обезопасит от большинства сканеров, которые при запросе упоминаю User-Agent.

Накладываем ограничение на 30 запросов в 1 секунду.&#x20;

```
iptables -I INPUT 2 -p udp -m state --state NEW -m recent --set --name SipAttacks --mask 255.255.255.255 --rsource -m udp --dport 5060
iptables -I INPUT 2 -p udp -m state --state NEW -m recent --update --seconds 1 --hitcount 30 --name SipAttacks --mask 255.255.255.255 --rsource -m udp --dport 5060 -j DROP
iptables -I INPUT 2 -p tcp -m state --state NEW -m recent --set --name SipAttacks --mask 255.255.255.255 --rsource -m tcp --dport 5060
iptables -I INPUT 2 -p tcp -m state --state NEW -m recent --update --seconds 1 --hitcount 30 --name SipAttacks --mask 255.255.255.255 --rsource -m tcp --dport 5060 -j DROP
iptables -I INPUT 2 -p tcp -m state --state NEW -m recent --set --name SipAttacks --mask 255.255.255.255 --rsource -m tcp --dport 5061
iptables -I INPUT 2 -p tcp -m state --state NEW -m recent --update --seconds 1 --hitcount 30 --name SipAttacks --mask 255.255.255.255 --rsource -m tcp --dport 5061 -j DROP
```

