# Hyper-V

### **Создание виртуальной машины**

1. Выберите **Action / New / Virtual Machine**
2. На вкладке Specify Name and Location введите имя (Name) виртуальной машины, например _mikopbx-vm_

<figure><img src="../../.gitbook/assets/MikoPBXHyperVInstallation_1.png" alt=""><figcaption></figcaption></figure>

3. Перейдите к следующей вкладке Specify Generation, укажите поколение - **Generation 1**

<figure><img src="../../.gitbook/assets/MikoPBXHyperVInstallation_2.png" alt=""><figcaption></figcaption></figure>

4. На вкладке Assign Memory выделите необходимый размер оперативной памяти, исходя из ожидаемой нагрузки на АТС. Для тестовой машины можно указать 2 Гб

<figure><img src="../../.gitbook/assets/MikoPBXHyperVInstallation_3.png" alt=""><figcaption></figcaption></figure>

5. Перейдите к вкладке Configure Networking, выберите заранее настроенное сетевое соединение

<figure><img src="../../.gitbook/assets/MikoPBXHyperVInstallation_4.png" alt=""><figcaption></figcaption></figure>

6. На вкладке Connect Virtual Hard Disk скорректируйте размер диска под систему до **1 Гб**

<figure><img src="../../.gitbook/assets/MikoPBXHyperVInstallation_5.png" alt=""><figcaption></figcaption></figure>

7. На вкладке Installation Options установите флажок **Install an operating system from a bootable CD/DVD-ROM**
8. Выберите **Image file (.iso)** и укажите ссылку на файл из дистрибутива MikoPBX с расширением **.iso**

<figure><img src="../../.gitbook/assets/MikoPBXHyperVInstallation_6.png" alt=""><figcaption></figcaption></figure>

9. Завершив ввод значений, нажмите кнопку **Finish**

<figure><img src="../../.gitbook/assets/MikoPBXHyperVInstallation_7.png" alt=""><figcaption></figcaption></figure>

### **Диск для хранения данных**

1. Перейдите к настройкам (Settings) созданной виртуальной машины
2. Выберите IDE контроллер, к которому подключен диск под систему
3. На открывшейся вкладке выберите Hard Drive, нажмите кнопку **Add**
4. Нажмите кнопку **New**
5. На вкладке Choose disk format выберите формат - **VHD**

<figure><img src="../../.gitbook/assets/MikoPBXHyperVInstallation_8.png" alt=""><figcaption></figcaption></figure>

6. На вкладке Choose disk type выберите тип диска - **Fixed size**

<figure><img src="../../.gitbook/assets/MikoPBXHyperVInstallation_9.png" alt=""><figcaption></figcaption></figure>

7. На вкладке Specify name and location укажите имя (Name), например _storage.vhd_, и расположение диска

<figure><img src="../../.gitbook/assets/MikoPBXHyperVInstallation_10.png" alt=""><figcaption></figcaption></figure>

8. На вкладке Configure Disk задайте размер диска для хранения данных не менее 50Гб

<figure><img src="../../.gitbook/assets/MikoPBXHyperVInstallation_11.png" alt=""><figcaption></figcaption></figure>

9. Для других полей используйте значения по умолчанию
10. Завершите настройку, нажав кнопку **Finish**

<figure><img src="../../.gitbook/assets/MikoPBXHyperVInstallation_12.png" alt=""><figcaption></figcaption></figure>

11. Для запуска виртуальной машины нажмите **Start**

<figure><img src="../../.gitbook/assets/MikoPBXHyperVInstallation_13.png" alt=""><figcaption></figcaption></figure>

### **Установка АТС MikoPBX**

1. Перейдите к вкладке Connect созданной виртуальной машины _mikopbx-vm_
2. Если загрузка прошла успешно, появится консольное меню. Введите с клавиатуры **8** для начала установки

<figure><img src="../../.gitbook/assets/MikoPBXHyperVInstallation_14.png" alt=""><figcaption></figcaption></figure>

3. Выберите диск под систему и введите с клавиатуры имя диска, например _**sdс**_. Подтвердите выбор, введите с клавиатуры _**y**_

<figure><img src="../../.gitbook/assets/MikoPBXHyperVInstallation_15.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/MikoPBXHyperVInstallation_16.png" alt=""><figcaption></figcaption></figure>

4. Подключите диск для хранения записей разговоров, ведите с клавиатуры наименование диска для подключения, например _**sdb**_

<figure><img src="../../.gitbook/assets/MikoPBXHyperVInstallation_17.png" alt=""><figcaption></figcaption></figure>

### **Запуск АТС MikoPBX**

1. На открытой вкладке Connect скопируйте внешний адрес созданной виртуальной машины и введите его в строке браузера

<figure><img src="../../.gitbook/assets/MikoPBXHyperVInstallation_18.png" alt=""><figcaption></figcaption></figure>

2. Для входа используйте логин - admin и пароль - admin

<figure><img src="../../.gitbook/assets/MikoPBXHyperVInstallation_19.png" alt=""><figcaption></figcaption></figure>
