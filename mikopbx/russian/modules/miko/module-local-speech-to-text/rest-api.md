# REST API

Базовый путь:

```
/pbxcore/api/v3/module-local-speech-to-text
```

Запросы авторизуются Bearer-токеном.

#### Расшифровки для интеграций

* `GET /transcripts?limit=50&offset=0&date_from=YYYY-MM-DD&date_to=YYYY-MM-DD&search={текст}`
* `GET /transcripts/{result_id}`
* `GET /call-transcripts/{call_transcript_id}?revision={revision}`
* `GET /call-transcripts/events?cursor=created_at:event_id&limit=100&include_deleted=false`

`transcripts` возвращает результаты распознавания отдельных записей. Параметр `limit` принимает значения от 1 до 200, `search` — поисковый запрос по тексту расшифровки. Детальная расшифровка содержит стабильные `segment_id`, исходные сегменты, объединенные реплики `turns` и простой текст.

`call-transcripts` объединяет несколько записей одного логического звонка в версионную расшифровку. Манифест частей сохраняет `cdr_start_ms` и `cdr_end_ms`, а сегменты содержат относительные и абсолютные таймкоды. Соседние сегменты одного участника и канала объединяются в одну реплику без потери исходных `segment_id`. Ответ содержит `source_instance_id` — постоянный идентификатор набора данных модуля — и `contract_version`.

`call-transcripts/events` публикует идемпотентные события `call-transcript.completed` и `call-transcript.updated`. Для чтения с начала используйте курсор `0:0`; сохраняйте `next_cursor` только после успешной обработки всех полученных событий. С параметром `include_deleted=true` дополнительно возвращаются события `call-transcript.deleted` с полями `deleted_at` и `deletion_reason`: `manual` — удалено вручную, `retention` — истек срок хранения.

#### Администрирование

| Операция                            | Endpoint                            |
| ----------------------------------- | ----------------------------------- |
| Список заданий                      | `GET /jobs`                         |
| Задание                             | `GET /jobs/{job_id}`                |
| Создание задания для файла записи   | `POST /jobs`                        |
| Повтор или сброс ошибочного задания | `PATCH /jobs/{job_id}`              |
| Повтор или сброс нескольких заданий | `PATCH /jobs`                       |
| Удаление задания                    | `DELETE /jobs/{job_id}`             |
| Массовое удаление по статусу        | `DELETE /jobs`                      |
| Список обработчиков                 | `GET /workers`                      |
| Обработчик                          | `GET /workers/{worker_id}`          |
| Изменение обработчика               | `PUT`, `PATCH /workers/{worker_id}` |
| Удаление обработчика                | `DELETE /workers/{worker_id}`       |

Действие `retry` сохраняет счетчик попыток, `reset` обнуляет его; изменять можно только ошибочные задания. Удалить можно только ожидающее или ошибочное задание без сохраненных результатов. Обработчик удаляется только при отсутствии активного lease; история заданий и результатов сохраняется. Учетные данные обработчика не дают доступа к управлению заданиями.

#### Worker API v2

Worker сначала вызывает `GET /worker-api-contract`, а затем передает заголовок `X-MikoPBX-Worker-API-Version: 2` во всех запросах Worker API. Запрос без подходящей версии отклоняется с кодом `426` и ошибкой `worker_upgrade_required`.

Ответ `GET /worker-processing-settings` содержит централизованный профиль обработки и объект `selected_model` с идентификатором, репозиторием, движком, типом Core ML-артефакта и отображаемым названием модели. Задание также содержит `model_engine` и `model_artifact_type`, по которым Worker выбирает Parakeet или WhisperKit. Произвольные сочетания движка, модели и артефакта отклоняются.

| Операция                       | Endpoint                          |
| ------------------------------ | --------------------------------- |
| Регистрация                    | `POST /workers`                   |
| Профиль обработки              | `GET /worker-processing-settings` |
| Лицензия для обновления Worker | `GET /worker-update-license`      |
| Получение lease                | `POST /job-leases`                |
| Скачивание записи              | `GET /job-recordings/{job_id}`    |
| Продление lease                | `PATCH /job-leases/{job_id}`      |
| Освобождение lease             | `DELETE /job-leases/{job_id}`     |
| Отправка результата            | `PUT /job-results/{job_id}`       |
| Отправка ошибки                | `PUT /job-failures/{job_id}`      |
