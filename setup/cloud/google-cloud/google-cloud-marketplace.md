# Google Cloud Маркетплейс

Авторизуйтесь на платформе [https://console.cloud.google.com/](https://console.cloud.google.com/)

MikoPBX в Google Cloud Маркетплейс: [https://console.cloud.google.com/marketplace/product/mikopbx-public/mikopbx](https://console.cloud.google.com/marketplace/product/mikopbx-public/mikopbx)

Приступим к настройке

{% hint style="info" %}
Для быстрого и удобного поиска на платформе Google Cloud используйте панель поиска
{% endhint %}

### **Добавление** ролей с**ервисной** учетной записи

Если у вас есть сервисная учетная запись, проверьте наличие нужных ролей, в случае необходимости добавьте их

Если сервисной учетной записи нет, создайте и добавьте ей нужные роли

1. Откройте Navigation menu / Products & solutions / Management / **IAM & Admin**
2. Перейдите к вкладке Service accounts и нажмите на **CREATE SERVICE ACCOUNT**
3. Введите имя сервисной учетной записи, например _mikopbx-service-account_
4. Нажмите кнопку **CREATE AND CONTINUE**

<figure><img src="../../../.gitbook/assets/MikoPBXGoogleCloudInstallation_1 (1).png" alt=""><figcaption></figcaption></figure>

5. Добавьте роли **Cloud Infrastructure Manager Agent, Compute Admin, Compute Network Admin, Service Account User**

<figure><img src="../../../.gitbook/assets/MikoPBXGoogleCloudInstallation_2 (1).png" alt=""><figcaption></figcaption></figure>

6. Нажмите кнопку **DONE**

<figure><img src="../../../.gitbook/assets/MikoPBXGoogleCloudInstallation_4 (1).png" alt=""><figcaption></figcaption></figure>

### **Создание виртуальной машины**

1. Откройте Marketplace и введите в поисковой строке **MikoPBX**
2. Выберите образ [MikoPBX](https://console.cloud.google.com/marketplace/product/mikopbx-public/mikopbx)
3. На открытой вкладке выберите **LAUNCH**

<figure><img src="../../../.gitbook/assets/MikoPBXGoogleCloudInstallation_5 (1).png" alt=""><figcaption></figcaption></figure>

4. В поле Deployment name введите имя, например _mikopbx-vm_
5. В разделе Deployment Service Account установите флажок Existing account и выберите созданный ранее сервисный аккаунт

<figure><img src="../../../.gitbook/assets/MikoPBXGoogleCloudInstallation_7 (2).png" alt=""><figcaption></figcaption></figure>

{% hint style="danger" %}
Для развертывания АТС используйте **два** диска:

* диск объемом **1 Гб** для основной системы
* диск объемом **50+ Гб** для хранения записей разговоров
{% endhint %}

6. При необходимости измените размер диска для хранения данных в разделе Data Storage, по умолчанию его размер - 50Гб

<figure><img src="../../../.gitbook/assets/MikoPBXGoogleCloudInstallation_8 (1).png" alt=""><figcaption></figcaption></figure>

7. В разделе Networking все необходимые правила Firewall настраиваются автоматически

<figure><img src="../../../.gitbook/assets/MikoPBXGoogleCloudInstallation_15 (1).png" alt=""><figcaption></figcaption></figure>

8. Для других полей используйте значения по умолчанию
9. Завершив ввод значений, нажмите кнопку **DEPLOY**

<figure><img src="../../../.gitbook/assets/MikoPBXGoogleCloudInstallation_9 (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/MikoPBXGoogleCloudInstallation_10 (2).png" alt=""><figcaption></figcaption></figure>

### **Запуск АТС MikoPBX**

1. Откройте вкладку Compute Engine и перейдите в раздел Virtual machines / VM Instance
2. Перейдите к созданной виртуальной машине _mikopbx-vm-mikopbx-vm_
3. На открытой вкладке перейдите к Logs / Serial port 1 (console)

<figure><img src="../../../.gitbook/assets/MikoPBXGoogleCloudInstallation_12 (2).png" alt=""><figcaption></figcaption></figure>

4. Скопируйте внешний адрес созданной виртуальной машины и введите его в строке браузера
5. Для входа используйте указанные в Serial port 1 (console) логин и пароль

<figure><img src="../../../.gitbook/assets/MikoPBXGoogleCloudInstallation_13 (2).png" alt=""><figcaption></figcaption></figure>
