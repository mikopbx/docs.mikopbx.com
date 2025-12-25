---
description: >-
  Модуль удобно использовать как бесплатный генератор речи из текста. Модуль
  предоставляет REST API и web интерфейс для генерации медиа файлов.
---

# Генерация речи RHVoice

### Системные требования

* Должен быть установлен модуль Docker - он бесплатный

### Настройка модуля

<figure><img src="../../.gitbook/assets/Снимок экрана 2025-12-25 в 13.26.41.png" alt=""><figcaption></figcaption></figure>

* **Порт для публикации сервиса** - позволяет настроит порт, который займет HTTP сервис, будет доступен только локально
* **Диктор** / **Темп** / **Текст** - указав эти параметры и нажав "**Создать и скачать**" - будет загружен сгенерированный из текста **wav** файл

### REST и сценарии использования

Пример запроса генерации речи:

```
curl -X GET \
"http://127.0.0.1:8081/say?text=%D0%9F%D1%80%D0%B8%D0%B2%D0%B5%D1%82&voice=vitaliy-ng&format=wav&rate=60"\
--output output.wav
```

* **`8081`** - порт, задается в настройках модуля.&#x20;
* **`text`** - текст для генерации речи
*   **`voice`** - диктор

    ```
             alan|bdl|clb|evgeniy-eng|lyubov|slt| # American English
             aleksandr|aleksandr-hq|anna|arina|artemiy|elena|evgeniy-rus|irina|mikhail| # Russian
             pavel|tatiana|timofey|umka|victoria|vitaliy|vitaliy-ng|vsevolod|yuriy| # Russian
             alicja|cezary|magda|michal|natan| # Polish
             azamat|nazgul| # Kyrgyz
             hana| # Albanian
             kiko|suze| # Macedonian
             letícia-f123| # Brazilian Portuguese
             natia| # Georgian
             ondro| # Slovak
             sevinch| # Uzbek
             spomenka| # Esperanto
             talgat| # Tatar
             zdenek # Czech
    ```
* **`format`** - расширение итогового медиа файла, допустимые варианты: `wav` / `mp3`
* **`rate`** - темп речи, комфортное значение 60

Мы используем этот API в случаях:

* Реализация голосового меню Smart IVR
* Модуль "Автоинформатор" - для автоматических обзвонов и обработки входящих вызовов

### Документация

* [https://github.com/RHVoice/RHVoice/tree/master](https://github.com/RHVoice/RHVoice/tree/master)
* [https://rhvoice.ru/](https://rhvoice.ru/)
* [https://github.com/Aculeasis/rhvoice-rest](https://github.com/Aculeasis/rhvoice-rest)

