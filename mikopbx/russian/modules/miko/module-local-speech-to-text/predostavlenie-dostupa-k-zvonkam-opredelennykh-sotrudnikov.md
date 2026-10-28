# Предоставление доступа к звонкам определенных сотрудников

Иногда руководителю отдела нужно открыть доступ к звонкам не всех, а лишь некоторых подчиненных. Из этой статьи вы узнаете, как пошагово выполнить такую настройку.

1. Установите модуль "[Управление доступом в систему](../module-users-u-i.md)".

<figure><img src="../../../.gitbook/assets/MikoPBXUIModule.png" alt=""><figcaption><p>Установка модуля управления доступом в систему</p></figcaption></figure>

2. Включите модуль и перейдите в его настройки.

<figure><img src="../../../.gitbook/assets/MikoPBXUIModuleGoToSettings.png" alt=""><figcaption><p>Переход в настройки модуля Управления доступом в систему</p></figcaption></figure>

3. Создайте новую группу доступа.

<figure><img src="../../../.gitbook/assets/UIModuleNewGroup.png" alt=""><figcaption><p>Создание новой группы доступа</p></figcaption></figure>

4. Задайте название новой группе и нажмите "Сохранить".

<figure><img src="../../../.gitbook/assets/UIModuleNewGroupParameters.png" alt=""><figcaption><p>Параметры создаваемой группы</p></figcaption></figure>

5. После сохранения настроек у Вас откроется раздел с управлением правами доступа. Перейдите во вкладку "Модуль локальной транскрибации" и поставьте галочку напротив "просмотр расшифровок и записей звонков". Сохраните настройку.

<figure><img src="../../../.gitbook/assets/MikoPBXUIModuleAccessRulesForSTT.png" alt=""><figcaption><p>Настройка доступа</p></figcaption></figure>

6. Перейдите в раздел "Административный интрфейс". Поставьте там галочку напротив "Доступ к разделу" в "История вызовов". Сохраните настройки.

<figure><img src="../../../.gitbook/assets/MikoPBXUIModuleAccessRulesForCDR.png" alt=""><figcaption><p>Выдача прав на доступ к разделу История вызовов</p></figcaption></figure>

7. После сохранения настроек, у Вас появится раздел "Ограничения для истории разговоров". В нем выберите к звонками каких сотрудников у менеджера будет доступ.&#x20;

<figure><img src="../../../.gitbook/assets/MikoPBXUIModuleChooseExtensions.png" alt=""><figcaption><p>Настройка к записям каких сотрудников будет доступ</p></figcaption></figure>

8. Перейдите в раздел "Пользователи группы доступа" и добавьте учетку менеджера к текущей группе.

<figure><img src="../../../.gitbook/assets/MikoPBXUIModuleGiveAccess.png" alt=""><figcaption><p>Добавление пользователя в группу доступа</p></figcaption></figure>

9. Вернитесь на главную страницу модуля. Задайте данные для входа Вашему менеджеру (логин и пароль).

<figure><img src="../../../.gitbook/assets/MikoPBXUIModuleUserCredentials.png" alt=""><figcaption><p>Добавление данных для входа менеджеру</p></figcaption></figure>

Далее менеджер должен авторизоваться в веб-интерфейсе MikoPBX с его данными. Он получит доступ только к разделу со звонками и к локальной транскрибации, к записям сотрудников которых Вы разрешили.
