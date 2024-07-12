# Обновление из консоли MikoPBX

Ниже приведен пример с АТС, установленной на виртуальную машину VirtualBOX. Обновление производится с версии 2022.2.102 до версии 2022.3.15.\
Скачайте из [репозитория](https://github.com/mikopbx/Core/releases) iso-образ нужной вам версии АТС.

<figure><img src="../../../.gitbook/assets/obnov_kons_0 (1).png" alt=""><figcaption></figcaption></figure>

В программе VirtualBOX откройте настройки виртуальной машины на которой установлена АТС.\
Перейдите в раздел **Носители.**\
Выделите виртуальный оптический привод. \
Нажмите на значок ![](../../../.gitbook/assets/obnov\_kons\_1.png) в группе **Атрибуты,** нажмите **Выбрать файл диска**.\
Выберите скачанный iso-образ АТС.\
Запустите машину.

<figure><img src="../../../.gitbook/assets/obn_kons_0.gif" alt=""><figcaption></figcaption></figure>

В консоли отобразится соответствующая строка "<mark style="color:red;">**The system loaded in Recovery mode**</mark>" («АТС загружена в режиме восстановления» на русском).

<figure><img src="../../../.gitbook/assets/obnov_kons_2.png" alt=""><figcaption></figcaption></figure>

Выберите **Install / Repair** (или нажмите на клавиатуре цифру **8)** и нажмите «**Enter**».

Вам нужна команда "**Update to version \*\*\*\*.\*.\*\*".** Нажмите на клавиатуре цифру **2,** затем нажмите «**Enter**».\
Начнется установка обновления. Когда она завершится, АТС перезагрузится.

<figure><img src="../../../.gitbook/assets/obnov_kons_1.gif" alt=""><figcaption></figcaption></figure>

После перезагрузки АТС сообщения "<mark style="color:red;">**The system loaded in Recovery mode**</mark>" уже не будет, что означает, что АТС загрузилась с жесткого диска, а не с виртуального оптического привода.\
Вверху зеленым шрифтом будет обозначена установленная версия обновления.

<figure><img src="../../../.gitbook/assets/obnov_kons_3.png" alt=""><figcaption></figcaption></figure>
