# Диагностика проблем с лицензированием

{% hint style="warning" %}
**Обязательное требование**: необходим доступ в интернет!\
ПК где запускается MikoPBX должен имеет доступ к следующим адресам:\
**lic.miko.ru:443**\
**lm.miko.ru:443**

Убедитесь, что **наши ресурсы** включены в список **доверенных** в антивирусе / брандмауэре или прочем софте.
{% endhint %}

Если Вы испытываете проблемы при подключении **MikoPBX**, и в логах отображаются ошибки, связанные с лицензированием SaaS, то Вам необходимо выполнить следующие проверки:

1. Если Вы используете **Прокси сервер**, то отключите проксирование запросов на **lic.miko.ru:443** и **lm.miko.ru:443** по https протоколу.
2. Проверьте, открывается ли у Вас сайт [**https://lm.miko.ru**](https://lm.miko.ru/) или [**https://lic.miko.ru/**](https://lic.miko.ru/).&#x20;
3. Подключитесь к АТС с помощью **SSH-клиента** по [инструкции](http://wiki.askozia.ru/handbook:putty).
4. Выполните команду: `aprotect license.getinfo MIKO-XXXXX-XXXXX-XXXXX-XXXXX`, где **MIKO-XXXXX-XXXXX-XXXXX-XXXXX** - Ваш [регистрационный номер](https://wiki.miko.ru/astpanel:miko\_saas\_license\_overview#registracionnyj\_nomer).
5.  Выполните команды:

    ```
    ping lm.miko.ru 
    ping lic.miko.ru
    ```
6.  Выполните команды:

    ```
    traceroute lic.miko.ru
    traceroute lm.miko.ru
    ```
7.  Выполните команды:

    ```
    openssl s_client -connect lm.miko.ru:443
    openssl s_client -connect lic.miko.ru:443
    ```
8.  Выполните команду:

    ```
    cat /etc/resolv.conf
    ```

{% hint style="success" %}
**Результат** выполнения выше описанных проверок пришлите нам через [форму обратной связи](https://telefon.miko.ru/contacts/).
{% endhint %}
