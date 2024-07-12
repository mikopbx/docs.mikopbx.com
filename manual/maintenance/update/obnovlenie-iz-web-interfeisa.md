# Обновление из web-интерфейса

В некоторых разделах интерфейса (например, **Сотрудники**) в правом нижнем углу указана текущая версия MikoPBX.

<figure><img src="../../../.gitbook/assets/obnov_ats_0 (1).png" alt=""><figcaption></figcaption></figure>

В web-интерфейсе АТС перейдите в **Обслуживание** → **Обновление PBX**.

<figure><img src="../../../.gitbook/assets/obnov_ats_1.png" alt=""><figcaption></figcaption></figure>

Если есть версии АТС новее вашей текущей, они будут отображены в таблице **Доступны онлайн обновления**, в которой в первом поле номер версии, а во втором - список изменений.

{% hint style="warning" %}
Рекомендуем проводить обновления последовательно, «не перепрыгивая» через релизы.
{% endhint %}

<figure><img src="../../../.gitbook/assets/obnov_ats_2.png" alt=""><figcaption></figcaption></figure>

Далее возможны два варианта обновления: **обновление онлайн, обновление скачанным img-файлом**.

### Обновление онлайн

{% hint style="danger" %}
**Будьте внимательны**! Если система установлена на тот же диск, где хранятся записи разговоров, то могут быть сложности с обновлением. [см. форум](https://qa.askozia.ru/5061/%D0%BF%D1%80%D0%BE%D0%BF%D0%B0%D0%B4%D0%B0%D0%B5%D1%82-%D1%80%D0%B0%D0%B7%D0%B4%D0%B5%D0%BB-%D0%BF%D0%BE%D1%81%D0%BB%D0%B5-%D0%BE%D0%B1%D0%BD%D0%BE%D0%B2%D0%BB%D0%B5%D0%BD%D0%B8%D1%8F-%D0%BD%D0%B0-6-7-7-31)
{% endhint %}

Обновления загружаются на АТС и сразу же применяются.\
Для обновления таким способом нажмите кнопку ![](../../../.gitbook/assets/obnov\_ats\_3.png) в нужной вам версии обновления.

<figure><img src="../../../.gitbook/assets/obnov_ats_4.png" alt=""><figcaption></figcaption></figure>

Появится окно предупреждения. Нажмите в нем **Обновить.**

<figure><img src="../../../.gitbook/assets/obnov_ats_5.png" alt=""><figcaption></figcaption></figure>

АТС загрузит и применит обновления, а затем перезагрузится.

### **Обновление скачанным img-файлом**

{% hint style="info" %}
Следует сразу заметить, что данным способом можно выполнить не только обновление, но и откат на более прежнюю версию.
{% endhint %}

Для обновления данным способом нажмите кнопку ![](../../../.gitbook/assets/obnov\_ats\_6.png) в нужной вам версии обновления.

<figure><img src="../../../.gitbook/assets/obnov_ats_7.png" alt=""><figcaption></figcaption></figure>

Начнется скачивание img-образа. Дождитесь завершения загрузки.

Затем нажмите кнопку ![](../../../.gitbook/assets/obnov\_ats\_8.png) и выберите данный img-файл.

<figure><img src="../../../.gitbook/assets/obnov_ats_0.gif" alt=""><figcaption></figcaption></figure>

Затем нажмите **Применить обновление,** а в появившемся окне предупреждения нажмите **Обновить.**&#x20;

<figure><img src="../../../.gitbook/assets/obnov_ats_1.gif" alt=""><figcaption></figcaption></figure>

Начнется применение обновлений. После окончания которого АТС перезагрузится.

<figure><img src="../../../.gitbook/assets/obnov_ats_2.gif" alt=""><figcaption></figcaption></figure>
