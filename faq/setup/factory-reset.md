# Сброс к заводским настройкам

## Способ 1

1. Перейдите в раздел «**Общие настройки**» -> "**Удаление настроек системы**"

<figure><img src="../../.gitbook/assets/1 (43).png" alt=""><figcaption></figcaption></figure>

2. В поле ввода вставьте текст "**удалить всё**", нажмите "**Сохранить**"

<figure><img src="../../.gitbook/assets/2 (8).png" alt=""><figcaption></figcaption></figure>

## Способ 2

1. Откройте консольное меню MikoPBX. С клавиатуры введите **9** для перехода в **консоль АТС**.

<figure><img src="../../.gitbook/assets/3 (7).png" alt=""><figcaption></figcaption></figure>

2. Введите последовательно две команды:

```
cp /conf.default/mikopbx.db /cf/conf/mikopbx.db
reboot
```

3. После выполнения данных команд MikoPBX перезагрузится. Вход в web-интерфейс происходит с логином (admin) и паролем (admin) по умолчанию.
