# Имитация внешних входящих и исходящих звонков

Полезным инструментом для настройки АТС MikoPBX будет имитация входящих и исходящих внешних звонков, чтобы не подключать реального провайдера, тем самым сэкономив.

## Создание нового SIP-провайдера на MikoPBX

1. Переходим в раздел **Маршрутизация** → **Провайдеры телефонии**, нажимаем на кнопку «**Подключить SIP"**

<figure><img src="../../.gitbook/assets/11 (5).png" alt=""><figcaption></figcaption></figure>

2. Установите тип учетной записи - **Входящая регистрация**

<figure><img src="../../.gitbook/assets/13 (8).png" alt=""><figcaption></figcaption></figure>

3. Перейдите в **Расширенные настройки**

<figure><img src="../../.gitbook/assets/5 (10).png" alt=""><figcaption></figcaption></figure>

4. Включите переключатель «**Отключить использование поля fromuser**»;

<figure><img src="../../.gitbook/assets/3 (26).png" alt=""><figcaption></figcaption></figure>

5. В поле "**Дополнительные параметры**" введите:

```
[endpoint]
callerid = 79257184275 <79257184275>
```

Где номер «**79257184275**» может быть другим. Он будет отображаться в качестве номера абонента при внешнем входящем звонке. Все остальные настройки оставьте без изменений

<figure><img src="../../.gitbook/assets/4 (11).png" alt=""><figcaption></figcaption></figure>

6. Опишите входящие и исходящие маршруты. О том как это сделать написано в следующих документациях:

* [Исходящие маршруты](../../manual/routing/outbound-routes.md)
* [Входящие маршруты](../../manual/routing/incoming-routes.md)

## Настройте sip-учетку на софтфоне <a href="#nastrojte_sip-uchetku_na_softfone" id="nastrojte_sip-uchetku_na_softfone"></a>

Установите софтфон. В данном примере используется Zoiper. Но вы можете использовать и другой, например MicroSIP. ([Документация](../softphones/))

Создайте аккаунт где параметры:

* «**Domain**» - адрес вашей АТС;
* «**Username**» - сохраненный из АТС идентификатор **SIP-##########**.
* «**Password**» - пароль провайдера

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

После успешного подключения софтфона, лампочка возле созданного провайдера загорится зеленым

<figure><img src="../../.gitbook/assets/10 (9).png" alt=""><figcaption></figcaption></figure>

На этом настройка имитации провайдера закончена. Теперь, любой внешний исходящий звонок, звонок пойдет на ваш софтфон. Позвонив с софтфона на любой номер, вызов пойдет на вашу АТС, как внешний входящий звонок через провайдера, которого вы настроили. При этом номер звонящего абонента будет отображаться тот, который вы указали в поле «Дополнительные параметры» при настройке провайдера. Вы можете изменить этот номер, чтобы изменился номер «звонящего абонента».