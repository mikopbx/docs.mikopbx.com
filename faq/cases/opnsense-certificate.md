# SSL сертификат для web-интерфейса MIKOPBX от OPNSense

**OPNsense** — это операционная система на базе FreeBSD, предназначенная для создания межсетевых экранов (firewall) и маршрутизаторов. Она предоставляет мощные инструменты для управления сетью, включая VPN, фильтрацию трафика, мониторинг и балансировку нагрузки.

OPNSense в своем составе имеет центр сертификации и может выпускать SSL сертификаты для  web-интерфейса MIKOPBX.

Для начала необходимо убедится, что центр сертификации OPNSense настроен и корневой сертификат установлен на рабочий ПК пользователя.&#x20;

## Создание сертификата в OPNSense

1\. Сгенерируйте сертификат. Перейдите в ваш OPNSense сервер, откройте менеджер сертификатов:

<figure><img src="https://lh7-rt.googleusercontent.com/docsz/AD_4nXeFATohG5cZ-WZF1HftWTuhvEZCQwd1VD7SNn6zksje1t-pA7g6tQQ7KNvqy0AAAkVsxF4nqwYf1-XEQ3PVSGQ5-4SNoMyzKVMPLpZoR8HtW44x4imUVu5VfPaIiJpYTKUvNtfc0rgbJe3gGKV7jZgfvJJLUm_LgIcRsO7jgA?key=mKA6FU2fXdcsY4hVdKBAGA" alt=""><figcaption><p>Менеджер сертификатов</p></figcaption></figure>

2. Выпустите внутренний сертификат с понятным для вас **названием**, **тип** сертификата - сервер, укажите **эмитентом** ваш внутренний цент сертификации, так же укажите срок действия сертификата в днях.

<figure><img src="https://lh7-rt.googleusercontent.com/docsz/AD_4nXeASTmI1dcGVK5iOvG4NheM-w3GNxAs9m3q8vf8KcK_b_4HkBD4LWI1NHak7j6ez4CbJnBORynXPHTjR0B8ex-mBL2dbmw_KmoXvOb5ZuHxJA38LyXgk0mlGZW6h5bDOMsdBkgre_joy1YUn9AdfobQQeixqSpeM0upCiEc?key=mKA6FU2fXdcsY4hVdKBAGA" alt=""><figcaption><p>Параметры сертификата</p></figcaption></figure>

3. Далее укажите DNS, настроенный на MIKOPBX.

<figure><img src="https://lh7-rt.googleusercontent.com/docsz/AD_4nXfAMLuS4DfTGqtTiLTJWZmDHU7iK0Qu4B8l_SUZaaF0O35aYHu5R7GzMxYMdkk003mgPAyKqNre4ZGMBhtsx26XbDgJ-aRR5x9UlYKXev5saVQhvSSr0MYhO4LqkHz-0LpXl_dvpyDrMZnmEZcgdaYohwKiEt5jnHQgyHAfaA?key=mKA6FU2fXdcsY4hVdKBAGA" alt=""><figcaption><p>DNS domain names</p></figcaption></figure>

Сохраните сертификат в OPNSense.

4.  &#x20;Сохраните публичный ключ и приватный ключ  сертификата на компьютер.

    Найдите в списке сертификатов, ваш сертификат PBX, нажмите скачать.



    <figure><img src="https://lh7-rt.googleusercontent.com/docsz/AD_4nXf6_IguGhBqXvi4gvQlSYg8xPtMnBnXMFkvLChzkblpckXz7G9EoAnhc9hueKFCFH-GM6aRdTsrRkBPRqxh7Q9cQ_LJAaAsbTqOn4ObE0x-BnqOTUT32nFUfGqzY3She3B9HCebCu_X4tRr3aGZl01t_iYu_GN7TXZTFOOMuw?key=mKA6FU2fXdcsY4hVdKBAGA" alt=""><figcaption><p>Кнопка "Скачать"</p></figcaption></figure>
5.  В списке скачайте публичный ключ (1) и приватный ключ (2).

    \


    <figure><img src="https://lh7-rt.googleusercontent.com/docsz/AD_4nXcDJvB_9dBFPhtigBKpgkrsXTjvwShi9cO4YOVThlyD8XObXHFkuq-SrOC9VZ0QGoStSNKHMwRHbevBp4MBBQbmqaJBqCNmbiEEvjA6ozM0V3Nu2zulggQZGgUF-DT00HsfG_ZIgiKJrFJWuHGFZA_6d5EJsLnxRATZ8Jr_?key=mKA6FU2fXdcsY4hVdKBAGA" alt=""><figcaption><p>Загрузка публичного и приватного ключа</p></figcaption></figure>

## Установка сертификата в MikoPBX

\
Перейдите в раздел **Общие настройки -> WEB-интерфейс**.

<figure><img src="../../.gitbook/assets/webInterface.png" alt=""><figcaption><p>Раздел WEB-интерфейс</p></figcaption></figure>

Откройте ранее скачанные файлы в блокноте. Вставьте содержимое файлов в Web-интерфейс.

Публичный ключ **HTTPs** содержимое с  - BEGIN CERTIFICATE

Приватный ключ **HTTPs** содержимое с  - BEGIN PRIVATE KEY

**Сохраните**.

Откройте Web-интерфейс MIKOPBX в анонимное режиме браузера используя SSL соединение HTTPS. Ваше подключение защищено.
