# Hyper-V

### **Создание виртуальной машины**

1. Выберите **Действие / Создать / Виртуальная машина**
2. На вкладке Укажите имя и местонахождение введите имя виртуальной машины, например _mikopbx-vm_

<figure><img src="../../.gitbook/assets/MikoPBXHyperVInstallation_ru_1.png" alt=""><figcaption></figcaption></figure>

3. Перейдите к следующей вкладке Укажите поколение, выберите поколение - **Поколение 1**

<figure><img src="../../.gitbook/assets/MikoPBXHyperVInstallation_ru_2.png" alt=""><figcaption></figcaption></figure>

4. На вкладке Выделить память выделите необходимый размер оперативной памяти, исходя из ожидаемой нагрузки на АТС. Для тестовой машины можно указать 2 Гб

<figure><img src="../../.gitbook/assets/MikoPBXHyperVInstallation_ru_3.png" alt=""><figcaption></figcaption></figure>

5. Перейдите к вкладке Настройка сети, выберите заранее настроенное сетевое соединение

<figure><img src="../../.gitbook/assets/MikoPBXHyperVInstallation_ru_4.png" alt=""><figcaption></figcaption></figure>

6. На вкладке Подключить виртуальный жесткий диск скорректируйте размер диска под систему до **1 Гб**

<figure><img src="../../.gitbook/assets/MikoPBXHyperVInstallation_ru_5.png" alt=""><figcaption></figcaption></figure>

7. На вкладке Параметры установки установите флажок **Установить операционную систему с загрузочного компакт- или DVD-диска**
8. Выберите **Файл образа (.iso)** и укажите ссылку на файл из дистрибутива MikoPBX с расширением **.iso**

<figure><img src="../../.gitbook/assets/MikoPBXHyperVInstallation_ru_6.png" alt=""><figcaption></figcaption></figure>

9. Завершив ввод значений, нажмите кнопку **Готово**

<figure><img src="../../.gitbook/assets/MikoPBXHyperVInstallation_ru_7.png" alt=""><figcaption></figcaption></figure>

### **Диск для хранения данных**

{% hint style="danger" %}
Для развертывания АТС используйте **два** диска:

* диск объемом **1 Гб** для основной системы
* диск объемом **50+ Гб** для хранения записей разговоров
{% endhint %}

1. Перейдите к Параметрам созданной виртуальной машины
2. Выберите IDE контроллер, к которому подключен диск под систему
3. На открывшейся вкладке выберите Жесткий диск, нажмите кнопку **Добавить**
4. Нажмите кнопку **Создать**
5. На вкладке Выбор формата диска выберите формат - **VHD**

<figure><img src="../../.gitbook/assets/MikoPBXHyperVInstallation_ru_8.png" alt=""><figcaption></figcaption></figure>

6. На вкладке Выберите тип диска укажите тип диска - **Фиксированного размера**

<figure><img src="../../.gitbook/assets/MikoPBXHyperVInstallation_ru_9.png" alt=""><figcaption></figcaption></figure>

7. На вкладке Укажите имя и местонахождение укажите имя, например _storage.vhd_, и расположение диска

<figure><img src="../../.gitbook/assets/MikoPBXHyperVInstallation_ru_10.png" alt=""><figcaption></figcaption></figure>

8. На вкладке Настройка диска задайте размер диска для хранения данных не менее 50Гб

<figure><img src="../../.gitbook/assets/MikoPBXHyperVInstallation_ru_11.png" alt=""><figcaption></figcaption></figure>

9. Для других полей используйте значения по умолчанию
10. Завершите настройку, нажав кнопку **Готово**

<figure><img src="../../.gitbook/assets/MikoPBXHyperVInstallation_ru_12.png" alt=""><figcaption></figcaption></figure>

### **Установка АТС MikoPBX**

1. Для запуска виртуальной машины нажмите **Пуск**

<figure><img src="../../.gitbook/assets/MikoPBXHyperVInstallation_ru_13.png" alt=""><figcaption></figcaption></figure>

2. Перейдите к вкладке Подключить созданной виртуальной машины _mikopbx-vm_
3. Если загрузка прошла успешно, появится консольное меню. Введите с клавиатуры **8** для начала установки

<figure><img src="../../.gitbook/assets/MikoPBXHyperVInstallation_ru_14.png" alt=""><figcaption></figcaption></figure>

4. Выберите диск под систему и введите с клавиатуры имя диска, например _**sdb**_. Подтвердите выбор, введите с клавиатуры _**y**_

<figure><img src="../../.gitbook/assets/MikoPBXHyperVInstallation_ru_15.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/MikoPBXHyperVInstallation_ru_16.png" alt=""><figcaption></figcaption></figure>

5. Подключите диск для хранения записей разговоров, ведите с клавиатуры наименование диска для подключения, например _**sdc**_

<figure><img src="../../.gitbook/assets/MikoPBXHyperVInstallation_ru_17.png" alt=""><figcaption></figcaption></figure>

{% hint style="danger" %}
После появления сообщения “Press any key within 30 seconds to boot from LiveCD…” не нажимайте никаких кнопок. В этом случае система загрузится с жесткого диска.
{% endhint %}

### **Запуск АТС MikoPBX**

1. На открытой вкладке Подключить скопируйте внешний адрес созданной виртуальной машины и введите его в строке браузера

<figure><img src="../../.gitbook/assets/MikoPBXHyperVInstallation_ru_18.png" alt=""><figcaption></figcaption></figure>

2. Для входа используйте логин - admin и пароль - admin

<figure><img src="../../.gitbook/assets/MikoPBXHyperVInstallation_19.png" alt=""><figcaption></figcaption></figure>
