---
description: Описание эндпоинтов
---

# REST API

Модуль предоставляет REST API для интеграций. Базовый путь:

```
/pbxcore/api/v3/module-ai-supervisor
```

| Назначение                       | Endpoint                                                             |
| -------------------------------- | -------------------------------------------------------------------- |
| Состояние модуля                 | `GET /health`                                                        |
| Данные вкладки «Обзор»           | `GET /dashboard`                                                     |
| Список и карточка звонка, разбор | `GET /calls`, `GET /calls/{id}`, `PATCH /calls`, `PATCH /calls/{id}` |
| Настройки                        | `GET/PATCH /settings`                                                |
| Мастер первичной настройки       | `GET/PATCH /onboarding`                                              |
| Источники расшифровок            | `GET/POST /sources`                                                  |
| Скрипты и их назначения          | `/quality-scripts`, `/quality-script-assignments`                    |
| Очередь заданий                  | `GET/POST/PATCH/DELETE /jobs`                                        |
| Импорт расшифровок               | `POST /imports`, `GET /imports/{id}`                                 |
| Ключи доступа обработчиков       | `GET/POST /worker-api-keys`, `DELETE /worker-api-keys/{id}`          |
| Зарегистрированные обработчики   | `/workers`                                                           |
| Журнал и диагностика             | `GET /logs`, `GET /diagnostic-reports`                               |

Ресурсы `/worker-api-contract`, `/job-leases`, `/job-recordings`, `/job-results`, `/job-partials`, `/job-failures` и `/job-voice-analytics` использует AI Supervisor Worker для получения и выполнения заданий. Для внешних интеграций они не предназначены.
