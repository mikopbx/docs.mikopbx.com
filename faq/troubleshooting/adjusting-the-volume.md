# Настройка громкости

Бывают случаи, когда при телефонном звонке очень тихо слышно клиента и на телефонном аппарате / софтфоне нет возможности повысить уровень громкости. Опишем способ, который может помочь в решении проблемы:

1. Перейдите в раздел "**Система**" -> "**Кастомизация системных файлов**"&#x20;

<figure><img src="../../.gitbook/assets/1 (15).png" alt=""><figcaption></figcaption></figure>

2. Откройте на редактирование **modules.conf**

<figure><img src="../../.gitbook/assets/2 (7).png" alt=""><figcaption></figcaption></figure>

3. Добавьте в конец файла

```
load => func_volume.so
```

Сохраните изменения

<figure><img src="../../.gitbook/assets/3 (12).png" alt=""><figcaption></figcaption></figure>

4. Откройте на редактирование файл **extensions.conf**

<figure><img src="../../.gitbook/assets/4 (20).png" alt=""><figcaption></figcaption></figure>

5. Добавьте следующий код в конец файла:

```
[all_peers-custom]
exten => _X!,1,Set(__DYNAMIC_FEATURES=vUp#vDown)
    same => n,Set(VOLUME(TX)=5)
    same => n,Set(VOLUME(RX)=5)
    same => n,return   
    
[add-trim-prefix-clid-custom]
exten => _.X!,1,Set(__DYNAMIC_FEATURES=vUp#vDown)
    same => n,Set(VOLUME(TX)=5)
    same => n,Set(VOLUME(RX)=5)
    same => n,return

[volume-level-control]
exten => up,1,NoOp()
    same => n,ExecIf($[ "${Vol}x" == "x" ]?Set(Vol=0)
    same => n,Set(Vol=$[${Vol}+5])
    same => n,Set(VOLUME(TX)=${Vol})  
    same => n,return
exten => down,1,NoOp()
    same => n,ExecIf($[ "${Vol}x" == "x" ]?Set(Vol=0)
    same => n,Set(Vol=$[${Vol}-5])
    same => n,Set(VOLUME(TX)=${Vol})  
    same => n,return
```

Сохраните изменения.

<figure><img src="../../.gitbook/assets/23 (2).png" alt=""><figcaption></figcaption></figure>

6. Откройте на редактирование файл **features.conf**

<figure><img src="../../.gitbook/assets/6 (11).png" alt=""><figcaption></figcaption></figure>

7. Добавьте в конец файла следующий код:

```
[applicationmap](+)
vUp => #1,self,Gosub,"volume-level-control,up,1"
vDown => #0,self,Gosub,"volume-level-control,down,1"
```

Сохраните изменения.

<figure><img src="../../.gitbook/assets/7 (6).png" alt=""><figcaption></figcaption></figure>

{% hint style="success" %}
Громкость по умолчанию станет выше, значение 5 вместо 0. При желании сотрудник может **набрать #1 для увеличения** громкости или **#0 для уменьшения**.
{% endhint %}
