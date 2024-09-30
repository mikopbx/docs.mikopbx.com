# Подключение к АТС с помощью WinSCP

1. Скачайте установщик программы WinSCP с официального сайта по [ссылке](https://winscp.net/eng/download.php)
2. Установите программу&#x20;
3. В случае, если ранее вы подключались к MikoPBX по SSH соединению с помощью PuTTYб программа сразу предложит импортировать соединение

<figure><img src="../../.gitbook/assets/1 (53).png" alt=""><figcaption></figcaption></figure>

4. Если же подключения не было - нужно заполнять данные вручную&#x20;

<figure><img src="../../.gitbook/assets/2 (33).png" alt=""><figcaption></figcaption></figure>

5. **Имя хоста** - IP адрес MikoPBX\
   **Порт -** порт для подключения по SSH по умолчанию это 22\
   **Имя пользователя** - всегда нужно указывать root\
   **Пароль -** SSH пароль от  Вашей **MikoPBX**

<figure><img src="../../.gitbook/assets/3 (40).png" alt=""><figcaption></figcaption></figure>

6. Перед подключением вам необходимо разрешить авторизацию по паролю в веб-интерфейсе MikoPBX, а также задать пароль для подключения: для этого перейдите "**Общие настройки**" -> "**SSH**" &#x20;

<figure><img src="../../.gitbook/assets/13 (11).png" alt=""><figcaption></figcaption></figure>

7. Для подключения нажмите "**Войти**"

<figure><img src="../../.gitbook/assets/4 (12).png" alt=""><figcaption></figcaption></figure>

8. После успешного подключения у вас откроется корневая папка MikoPBX

<figure><img src="../../.gitbook/assets/15 (10).png" alt=""><figcaption></figcaption></figure>
