# Сетевой экран

В MikoPBX все локальные подсети возможно описать в разделе «**Сеть и Firewall**» → «**Сетевой экран**». Сетевой экран предназначен для ограничения доступа к станции по типу трафика и подсетям.

<figure><img src="../../.gitbook/assets/new1.png" alt=""><figcaption></figcaption></figure>

Чтобы добавить новое правило, необходимо нажать на кнопку:

<figure><img src="../../.gitbook/assets/new2.png" alt=""><figcaption></figcaption></figure>

## Общие настройки

<figure><img src="../../.gitbook/assets/13 (12).png" alt=""><figcaption></figcaption></figure>

Название правила можно задать произвольным.\
Справа от **адреса подсети** есть поле **Маска подсети** в формате CIDR.&#x20;

## Доступные сервисы <a href="#dostupnye_servisy" id="dostupnye_servisy"></a>

<figure><img src="../../.gitbook/assets/14 (5).png" alt=""><figcaption></figcaption></figure>

* SIP\&RTP-регистрация телефонов и голосовой трафик - Session Initiation Protocol используется для установки соединений между VoIP телефонами
* WEB-доступ к административному интерфейсу настройки АТС
* SSH-root доступ к системе. SSH(Secure Shell) позволяет получить доступ к консоли MikoPBX.
* AMI-доступ к asterisk manager api через telnet. Asterisk Manager Interface (AMI) предоставляет доступ к Asterisk по TCP/IP протоколу.
* AJAM - доступ к asterisk manager api, через http, https
* ICMP-проверка связи командой ping
* CTICLIENT-подключение панели телефонии для 1С

## Дополнительные параметры <a href="#dopolnitelnye_parametry" id="dopolnitelnye_parametry"></a>

<figure><img src="../../.gitbook/assets/15 (10).png" alt=""><figcaption></figcaption></figure>

* У каждой подсети есть флаг «**Это VPN или локальная сеть**». При установке этого флага MikoPBX будет представляться всем локальным подсетям локальным IP, а не внешним.
* Флаг «**Никогда не блокировать адреса из этой сети**» следует включать только для доверенных подсетей. Если включен данный флаг, то правила защиты от взлома на данную подсеть не распространяются.

