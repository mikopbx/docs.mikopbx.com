# Отключить переадресацию на мобильный для внутренних звонков

В настройках [Сотрудника ](../../manual/telefoniya/extensions.md)есть возможность настроить маршрутизацию. Если сотрудник не доступен, то вызов можно направить на его мобильный номер телефона. Не для всех случаев эта функция необходима. К примеру часто для внутренних звонков клиенты просят ее отключить.

1. Перейдите в раздел  «**Система**» -> «**Кастомизация системных файлов**»

<figure><img src="../../.gitbook/assets/1 (41).png" alt=""><figcaption></figcaption></figure>

2. Выберите для редактирования файл «**/etc/asterisk/extensions.conf**»

<figure><img src="../../.gitbook/assets/4 (23).png" alt=""><figcaption></figcaption></figure>

3. Выберите режим «**Добавлять в конец файла**»

<figure><img src="../../.gitbook/assets/5 (9).png" alt=""><figcaption></figcaption></figure>

4. Добавьте во второе поле текст

```
[internal-users-custom]
; Только для внутренних отключаем переадресацию. 
exten => _X!,1,ExecIf($[ "${FROM_DID}x" == "x" ]?Set(__QUEUE_SRC_CHAN=${CHANNEL}))
	same => n,return
```

<figure><img src="../../.gitbook/assets/9 (3).png" alt=""><figcaption></figcaption></figure>

5. Сохраните изменения
