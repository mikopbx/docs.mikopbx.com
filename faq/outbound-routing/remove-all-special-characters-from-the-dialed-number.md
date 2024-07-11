# Убрать все спецсимволы из набираемого номера

Некоторые софтфоны / CTI решения при наборе передают номер телефона с спецсимволами, к примеру **+7(495) 229-3042**.

Такой вызов с большой вероятностью не пройдет, будет завершен по ошибке.

1. Для решения задачи «фильтрации» символов следует описать дополнительный контекст через меню [Кастомизация системных файлов](https://wiki.mikopbx.ru/custom-files)

<figure><img src="../../.gitbook/assets/1 (1) (1) (1).png" alt=""><figcaption><p>Окно "Кастомизация системных файлов"</p></figcaption></figure>

2. Править будем файл **extensions.conf**.&#x20;

<figure><img src="../../.gitbook/assets/2 (1) (1) (1).png" alt=""><figcaption><p>Файл 'Extensions.conf"</p></figcaption></figure>

3. Добавьте в конец файла следующий контекст:

```php
[all_peers-custom]

exten => s,1,NoOp(Cleaning dst number)
	same => n,Set(cleanNumber=${FILTER(\*\#1234567890,${EXTEN})})
	same => n,ExecIf($["${EXTEN}" != "${cleanNumber}"]?Goto(all_peers,${cleanNumber},1))
	same => n,return
```

<figure><img src="../../.gitbook/assets/newForm1 (1).png" alt=""><figcaption><p>Добавление кода в конец файла</p></figcaption></figure>

{% hint style="success" %}
Согласно описанному правилу, в набираемом номере останутся только символы \*#1234567890
{% endhint %}
