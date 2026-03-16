---
description: Подключение провайдера Mango Office
---

# Mango

### Настройка в личном кабинете Mango Office <a href="#nastrojka_v_lichnom_kabinete_mango_office" id="nastrojka_v_lichnom_kabinete_mango_office"></a>

1. Авторизуйтесь в [личном кабинете Mango](https://lk.mango-office.ru/). Подключите номер в личном кабинете.
2. Добавьте в личном кабинете учетную запись сотрудника.

<figure><img src="../../.gitbook/assets/newEmployeeMango.png" alt=""><figcaption><p>Добавление нового сотрудника в ЛК Mango Office</p></figcaption></figure>

3. Укажите параметры сотрудника, укажите в качестве роли - администратор.

<figure><img src="../../.gitbook/assets/newEmployeeMangoParameers.png" alt=""><figcaption><p>Параметры нового сотрудника</p></figcaption></figure>

4. Для созданного сотрудника должна быть автоматически создана SIP-учетная запись. Сохраните эти данные, они понадобятся нам для подключения провайдера в MikoPBX.

<figure><img src="../../.gitbook/assets/SIPCredetionals.png" alt=""><figcaption><p>Данные для подключения провайдера</p></figcaption></figure>

5. Перейдите на вкладку "**Обработка звонков"** → "**Голосовое меню и распределение звонков"**. Для подключенного номера направьте все входящие звонки на созданного вами сотрудника «**Админ 76**».

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption><p>Настройки обработки звонков</p></figcaption></figure>

6. Переходим в настройки SIP-учетной записи для сотрудника «Админ 76». **Общие настройки** → **Настройки SIP**, вкладка «**Учетные записи и домены SIP**»

<figure><img src="../../.gitbook/assets/image (1) (3).png" alt=""><figcaption><p>Вкладка «Учетные записи и домены SIP»</p></figcaption></figure>

Следующие данные будут использованы для SIP-подключения:

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption><p>Параметры для SIP авторизации</p></figcaption></figure>

### Подключение провайдера в веб-интерфейсе MikoPBX <a href="#podkljuchenie_provajdera_mango_v_mikopbx" id="podkljuchenie_provajdera_mango_v_mikopbx"></a>

1. Перейдите в раздел "**Провайдеры телефонии**".

<figure><img src="../../.gitbook/assets/providersMikoPBX.jpg" alt=""><figcaption><p>Раздел "Провайдеры телефонии"</p></figcaption></figure>

2. Нажмите "**Подключить SIP**" для подключение провайдера.

<figure><img src="../../.gitbook/assets/providers (1).jpg" alt=""><figcaption><p>Элемент "Подключить SIP"</p></figcaption></figure>

3. Укажите параметры подключения:

* **Название** - произвольное.
* **Хост или IP адрес** - IP-адрес из личного кабинета Mango.
* **Логин** - логин из личного кабинета Mango.

<figure><img src="../../.gitbook/assets/mangoProviderParameters (1).png" alt=""><figcaption><p>Параметры для SIP-подключения</p></figcaption></figure>

4. Перейдите в "**Расширенные настройки**". В поле "**Дополнительные адреса провайдера"** - добавьте две подсети «**81.88.86.0/24**» и «**81.88.82.245/32**». Для этого введите первый адрес в поле ввода, нажмите «Enter», он отобразится ниже. Таким же образом введите второй адрес. Это пул адресов Mango, с них могут поступать входящие вызовы. Сохраните настройки провайдера.

<figure><img src="../../.gitbook/assets/mangoProviderParameters2.png" alt=""><figcaption><p>Параметры для SIP-подключения</p></figcaption></figure>

Результатом успешного подключения является зеленый индикатор.

<figure><img src="../../.gitbook/assets/image (5).png" alt=""><figcaption><p>Успешное подключение провайдера</p></figcaption></figure>
