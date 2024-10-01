# Настройка Jitter

Обычно настройка Jitter необходима в том случае, если есть значительные проблемы с передачей RTP трафика. Голос собеседника запаздывает, пропадает, не слышно.

Лучше всего искать первопричину, анализировать дамп звонка на АТС / на роутере / на вышестоящем маршрутизаторе. Сравнивать и смотреть где jitter растет.

{% hint style="success" %}
Есть вот такая статья по [проблемам со звуком](https://blog.telefon1c.ru/analiz-probliem-so-zvukom/?\_ga=2.253624841.1000222345.1727446042-994790845.1727446042).
{% endhint %}

Но есть ряд случаев, когда нет возможности повлиять на качество канала связи. В этом случае, если задержка аудио потока будет слишком большой, то asterisk будет отбрасывать «опоздавшие» пакеты. Эту проблему может решить увеличение значения Jitter.

В MikoPBX настраивать jitter можно только через [кастомизацию системных файлов](../../../manual/system/custom-files.md):

**modules.conf** - добавить:

```
load => func_jitterbuffer.so
```

**extensions.conf** - для провайдера можно добавить **custom** контексты:

Для внутренних (исходящих):

```php
[all_peers-custom] 
exten => _.!,1,Set(JITTERBUFFER(fixed)=200,1500) 
    same => n,return
```

Для внутренних (входящих):

```php
[internal-users-custom] 
exten => _.!,1,Set(JITTERBUFFER(fixed)=200,1500) 
    same => n,return
```

Для входящих:

```php
[SIP-1611151795-incoming-custom] 
exten => _.!,1,Set(JITTERBUFFER(fixed)=200,1500) 
    same => n,return
```

Для исходящих :

```php
[SIP-1611151795-outgoing-custom] 
exten => _X!,1,Set(JITTERBUFFER(fixed)=200,1500) 
    same => n,return
```
