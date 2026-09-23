---
description: В данной инструкции будет описано подключение по SSH с помощью Putty
---

# Подключение к АТС с помощью SSH-клиента (Putty)

1. Скачайте программу для подключения по SSH. Это можно сделать на официальном сайте по [ссылке](https://putty.org.ru/download.html)
2. Запустите скаченную программу. У вас откроется главное меню.

<figure><img src="../../../.gitbook/assets/1 (40).png" alt=""><figcaption></figcaption></figure>

3. Перейдите в раздел «**Соединение**» - «**Данные**»

<figure><img src="../../../.gitbook/assets/2 (21).png" alt=""><figcaption></figcaption></figure>

4. «**Имя пользователя для автовхода**» укажите **root**

«**Строка типа терминала**» укажите **xterm-256color**

<figure><img src="../../../.gitbook/assets/3 (37).png" alt=""><figcaption></figcaption></figure>

5. Перейдите в раздел «**Кодировка**»

<figure><img src="../../../.gitbook/assets/4 (18).png" alt=""><figcaption></figcaption></figure>

6.  «**Кодировка**» - укажите **UTF-8**

    Установите флаг «**Включить рисование линий VT100 даже в режиме UTF-8**»

<figure><img src="../../../.gitbook/assets/5 (25).png" alt=""><figcaption></figcaption></figure>

7. Перейдите в раздел «**Сессия**» - «**Журнал**». Тут можно настроить вывод в файл:

<figure><img src="../../../.gitbook/assets/6 (14).png" alt=""><figcaption></figcaption></figure>

8. Перейдите в раздел "**Сеанс**"

<figure><img src="../../../.gitbook/assets/7 (14).png" alt=""><figcaption></figcaption></figure>

9\. Необходимые данные:

* **Имя хоста** (или IP-адрес)- IP адрес АТС
* **Порт -** порт для подключения по SSH по умолчанию **22**
* Введите имя сессии и сохраните ее настройки

<figure><img src="../../../.gitbook/assets/8 (5).png" alt=""><figcaption></figcaption></figure>

10. В дальнейшем используйте действие «**Загрузить**» для использования сохраненной ранее сессии

<figure><img src="../../../.gitbook/assets/9 (11).png" alt=""><figcaption></figcaption></figure>

11. Выполните действие «**Соединиться**» для подключения к АТС и введите пароль SSH

<figure><img src="../../../.gitbook/assets/10 (3).png" alt=""><figcaption></figcaption></figure>

12. Перед подключением вам необходимо разрешить авторизацию по паролю в веб-интерфейсе MikoPBX, а так же задать пароль для подключения: для этого перейдите "**Общие настройки**" -> "**SSH**"

<figure><img src="../../../.gitbook/assets/13 (12).png" alt=""><figcaption></figcaption></figure>

13. После ввода пароля SSH, у вас откроется меню АТС

<figure><img src="../../../.gitbook/assets/11 (3).png" alt=""><figcaption></figcaption></figure>

14. Для открытия консоли перейдите в "**\[9] Console(Shell)**"

<figure><img src="../../../.gitbook/assets/12 (11).png" alt=""><figcaption></figcaption></figure>
