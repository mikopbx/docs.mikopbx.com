# Настройка индивидуального нерабочего времени для нескольких провайдеров, на одном хосте

## Постановка задачи <a href="#postanovka_zadachi" id="postanovka_zadachi"></a>

Имеем две учетные записи от провайдера Zadarma, настроенных в MikoPBX по [инструкции](registering-multiple-accounts-from-one-provider.md).

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption><p>Две учетные записи</p></figcaption></figure>

Необходимо для каждого номера Zadarma настроить свое нерабочее время. Например, для номера +7(495)-229-30-42 рабочее время с 9.00 до 18.00 по МСК; для номера +7 (499) 638-25-84 рабочее время с 8.00 до 20.00 по МСК.

## Решение <a href="#reshenie" id="reshenie"></a>

1. Перейдите в раздел **Система** → **Кастомизация системных файлов**.

<figure><img src="../../.gitbook/assets/systemFileCustomization.png" alt=""><figcaption><p>Раздел "Кастомизация системных файлов"</p></figcaption></figure>

2. &#x20;Откройте для редактирования конфигурационный файл **extensions.conf**. Установите режим «**Добавлять в конец файла**». В черное окно добавьте следующий фрагмент кода:

```php
[public-direct-dial-custom]
exten => _.!,1,NoOp(check time)
    same => n,Gosub(check-out-work-time-custom,${FROM_DID},1)
    same => n,return
	
[check-out-work-time-custom]
exten => 584611,1,NoOp(check time)
    same => n,ExecIfTime(00:00-09:00,mon-fri,*,*?Macro(playback-exit,/offload/asterisk/sounds/other/out_work_times))
    same => n,ExecIfTime(18:00-23:59,mon-fri,*,*?Macro(playback-exit,/offload/asterisk/sounds/other/out_work_times))
    same => n,return
exten => 420296,1,NoOp(check time)
    same => n,ExecIfTime(00:00-08:00,mon-fri,*,*?Macro(playback-exit,/offload/asterisk/sounds/other/out_work_times))
    same => n,ExecIfTime(20:00-23:59,mon-fri,*,*?Macro(playback-exit,/offload/asterisk/sounds/other/out_work_times))
    same => n,return
```

