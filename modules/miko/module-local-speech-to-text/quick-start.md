---
description: Инструкция по быстрому началу работы с модулем
---

# Быстрый старт

### Установка модуля

1. Откройте веб-интерфейс MikoPBX.
2. Перейдите в раздел **"Модули"** → **"Маркетплейс модулей"**.

<figure><img src="../../../.gitbook/assets/MikoPBXModuleMarketplace.png" alt=""><figcaption><p>Раздел "Маркетплейс модулей"</p></figcaption></figure>

3. Найдите **Модуль локальной транскрибации** и установите его.
4. Перейдите в список установленных модулей и включите модуль.

<figure><img src="../../../.gitbook/assets/STTModuleInstalledModulesSection.png" alt=""><figcaption><p>Включение модуля "Локальная транскрибация"</p></figcaption></figure>

5. Откройте страницу модуля, нажав на иконку "Настройки", справа от версии модуля.

<figure><img src="../../../.gitbook/assets/STTModuleOpen.png" alt=""><figcaption><p>Переход на страницу модуля</p></figcaption></figure>

### Настройка модуля

1. Во вкладке "**Настройки"** оставьте язык "**Определять автоматически"** или выберите основной язык звонков.

<figure><img src="../../../.gitbook/assets/STTModuleSettingsPageFastStart.png" alt=""><figcaption><p>Выбор языка распознавания</p></figcaption></figure>

2. Во вкладке "**Маркетплейс моделей"** выберите модель "**Large V3 Turbo"** или **"Podlodka Turbo"** (для русского языка). Это рекомендуемые варианты для рабочей расшифровки звонков.

Нажмите "**Сохранить модель**".

<figure><img src="../../../.gitbook/assets/STTModuleChoosingModel.png" alt=""><figcaption><p>Выбор модели для распознавания</p></figcaption></figure>

3. Перейдите во вкладку "**Воркеры**" нажмите "**Создать API-ключ"** и сразу скопируйте ключ.

<figure><img src="../../../.gitbook/assets/STTModuleCreatingANewWorkerKey.png" alt=""><figcaption><p>Создание API ключа для внешней работы с модулем</p></figcaption></figure>

4. Откройте приложение-обработчик "**MIKO AI Worker**" на Mac. На первом шаге онбоардинга выберите язык интерфейса. Нажмите "**Далее**".

<figure><img src="../../../.gitbook/assets/MIKOAIWorkerChooseLang.png" alt=""><figcaption><p>Выбор языка онбоардинга и приложения</p></figcaption></figure>

5. Перейдите на Шаг 3 (нажмите "**Далее**" на Шаге 2). Введите адрес Вашей MikoPBX в формате `https://...` и задайте понятное имя обработчика.

Нажмите "**Проверить соединение**", потом "**Далее**".

<figure><img src="../../../.gitbook/assets/MIKOAIWorkerPBXAddress.png" alt=""><figcaption><p>Шаг "Подключение к MikoPBX"</p></figcaption></figure>

6. На шаге "**TLS и доверие сертификату"** выберите, проверять ли сертификат PBX. Если у Вас есть сертификат - включите проверку и укажите PEM-файл CA.

<figure><img src="../../../.gitbook/assets/MIKOAIWorkerTLSCert.png" alt=""><figcaption><p>Шаг "TLS и доверие сертификату"</p></figcaption></figure>

7. На шаге **"Распознавание речи",** вставьте созданный STT воркер token, нажмите **"Проверить токен"**, затем **"Зарегистрировать распознавание речи"**.

Нажмите "**Далее**".

<figure><img src="../../../.gitbook/assets/MIKOAISttRegister.png" alt=""><figcaption><p>Шаг "Распознавание речи"</p></figcaption></figure>

8. Если ИИ Аналитика на этом Mac не нужна, пропустите шаг **AI Analysis (**&#x43D;ажмите **"Далее".**

{% hint style="info" %}
Подробнее про модуль "ИИ Супервайзер" можно прочитать <mark style="color:red;">здесь</mark>.
{% endhint %}

9. На финальном шаге включите **"Запускать при входе"**, если обработчик должен запускаться вместе с macOS.

Нажмите "**Открыть MIKO AI Worker**".

<figure><img src="../../../.gitbook/assets/MIKOAIWorkerOnboardSummary.png" alt=""><figcaption><p>Итог настройки, выбор опции запуска при входе в MacOS</p></figcaption></figure>

Будет загружена модель с ресурса [Hugging Face](https://huggingface.co/), после этого начнется процесс расшифровки записей, переданных со станции. Можно отключить доступ к внешним ресурсам, локальной сети со станцией будет достаточно для дальнейшей работы.

Результат расшифровки Вы сможете найти в разделе "**Транскрипты**".

<figure><img src="../../../.gitbook/assets/STTWorkerProcess.png" alt=""><figcaption><p>Процесс расшифровки записи</p></figcaption></figure>

Более подробную инструкцию по настройке обработчика можно посмотреть [здесь](miko-ai-worker.md).
