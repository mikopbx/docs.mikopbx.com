---
description: >-
  Docker-обработчик для ИИ Супервайзера: установка на Linux-сервер с нуля,
  видеокарта NVIDIA, подключение к MikoPBX, проверка работы, обновление и
  удаление.
---

# Docker-обработчик (Linux)

**Docker-обработчик** - контейнер для локального ИИ-анализа звонков на вашем Linux-сервере. Он берет задания из очереди модуля **Локальный ИИ Супервайзер**, анализирует расшифровку локальной языковой моделью, определяет эмоции по записи разговора и отправляет результат на станцию. Расшифровки и записи не покидают вашу сеть.

Docker-обработчик - альтернатива приложению [AI Supervisor Worker для macOS](miko-ai-worker.md). Логика анализа у них общая: промпты, схема результата, привязка выводов к репликам расшифровки и критерии качества одинаковые, поэтому результат не зависит от того, какой обработчик взял задание.

Образ опубликован в Docker Hub: [`excla1m/mikopbx-ai-supervisor-worker`](https://hub.docker.com/r/excla1m/mikopbx-ai-supervisor-worker). Тег `latest` всегда указывает на последнюю стабильную версию.

### Как устроен

Обработчик запускается двумя контейнерами из одного файла `compose.yaml`:

| Контейнер              | Образ                                   | Что делает                                                                                                                       |
| ---------------------- | --------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| `ai-supervisor-worker` | `excla1m/mikopbx-ai-supervisor-worker`  | Связь с MikoPBX, очередь заданий, подготовка промптов, проверка ответа модели, анализ эмоций по голосу (SenseVoice, на процессоре) |
| `ai-supervisor-ollama` | `ollama/ollama`                         | Локальная языковая модель. Если на сервере есть видеокарта NVIDIA, модель работает на ней                                        |

Обработчик не выбирает звонки и не хранит серверную очередь: какие звонки анализировать, какой моделью и с какими компонентами, решает модуль в MikoPBX.

### Требования

| Компонент  | Минимум                                                                 | Рекомендуется                                                  |
| ---------- | ----------------------------------------------------------------------- | -------------------------------------------------------------- |
| ОС         | 64-битный Linux (x86\_64 или arm64)                                     | Ubuntu Server 24.04 LTS или Debian 12                          |
| Процессор  | 4 ядра с поддержкой AVX2                                                | 6-8 ядер                                                       |
| Память     | 8 ГБ                                                                    | 16 ГБ                                                          |
| Видеокарта | не обязательна                                                          | NVIDIA с 8 ГБ видеопамяти и больше, архитектура Turing или новее |
| Диск       | 40 ГБ                                                                   | 60 ГБ                                                          |
| Docker     | Docker Engine с плагином Compose 2.24 или новее                         | последняя стабильная версия                                    |
| MikoPBX    | модуль **Локальный ИИ Супервайзер** с поддержкой worker API v2          | последняя версия модуля                                        |
| Сеть       | исходящий HTTPS до MikoPBX, Docker Hub, registry.ollama.ai и huggingface.co | то же                                                          |

{% hint style="warning" %}
Без видеокарты модель работает на процессоре. Это подходит для небольшого потока звонков на сервере с современным процессором (AVX2, 8 ядер и больше). На старых процессорах без AVX2 анализ одного звонка может не уложиться в отведенное модулем время. Для постоянной работы рекомендуется видеокарта NVIDIA: анализ звонка на ней занимает десятки секунд.
{% endhint %}

### Модели

Модель выбирается в MikoPBX: **Настройки** → **ИИ-анализ**. Каждое задание приходит со своим профилем модели, и обработчик сам скачивает модель в Ollama, если ее еще нет.

| Профиль в MikoPBX                       | Модель Ollama | Размер   | Память (видеокарта или ОЗУ) | Когда выбирать                                   |
| --------------------------------------- | ------------- | -------- | --------------------------- | ------------------------------------------------ |
| **Qwen3.5 4B Q4** (рекомендуется)       | `qwen3.5:4b`  | 3,3 ГБ   | 4-6 ГБ                      | Повседневный анализ звонков, быстрая работа      |
| **Qwen3.5 9B Q4** (повышенное качество) | `qwen3.5:9b`  | 6,6 ГБ   | 8-12 ГБ                     | Сложные и длинные разговоры, нужна видеокарта от 12 ГБ |

Для анализа эмоций по голосу обработчик при первом таком задании скачивает модель **SenseVoice Small** (ONNX, около 240 МБ) в том `models` и проверяет контрольную сумму каждого файла.

{% hint style="info" %}
Если модель не помещается в видеопамять целиком, Ollama переносит часть слоев на процессор и анализ замедляется. Проверить размещение можно командой `docker compose exec ollama ollama ps` (см. раздел **Проверка работы**).
{% endhint %}

### Шаг 1. Подготовка сервера

Установите Docker Engine и плагин Compose:

```bash
curl -fsSL https://get.docker.com | sudo sh
sudo usermod -aG docker "$USER"      # чтобы запускать docker без sudo
newgrp docker                        # или перезайдите в сессию
```

{% hint style="info" %}
Если `get.docker.com` недоступен с сервера, установите Docker из [официального репозитория Docker](https://docs.docker.com/engine/install/ubuntu/) или из пакетов дистрибутива: `sudo apt-get install -y docker.io docker-compose-v2`.
{% endhint %}

Проверьте установку и процессор:

```bash
docker version
docker compose version
uname -m                                       # x86_64 или aarch64
grep -o -w avx2 /proc/cpuinfo | head -1        # на x86_64 должно вывести avx2
```

Проверьте доступ к MikoPBX и Docker Hub:

```bash
curl -skI https://<адрес-mikopbx> | head -1
curl -sI https://registry-1.docker.io/v2/ | head -1   # ответ 401 - это нормально, реестр доступен
```

### Шаг 2. Видеокарта NVIDIA (необязательно)

Пропустите этот шаг, если видеокарты нет: модель будет работать на процессоре.

1. Установите драйвер NVIDIA 550 или новее. В Ubuntu:

   ```bash
   sudo ubuntu-drivers install
   sudo reboot
   ```

2. Проверьте, что драйвер видит карту:

   ```bash
   nvidia-smi
   ```

3. Установите NVIDIA Container Toolkit, чтобы Docker мог передавать видеокарту контейнерам:

   ```bash
   curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey \
     | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg
   curl -fsSL https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list \
     | sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' \
     | sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
   sudo apt-get update
   sudo apt-get install -y nvidia-container-toolkit
   sudo nvidia-ctk runtime configure --runtime=docker
   sudo systemctl restart docker
   ```

4. Проверьте видеокарту внутри контейнера:

   ```bash
   docker run --rm --gpus all ubuntu nvidia-smi
   ```

   Команда должна показать вашу видеокарту, например:

   ```
   NVIDIA GeForce GTX 1080, 8192 MiB
   ```

### Шаг 3. Подготовка MikoPBX

1. Откройте **Модули** → **Локальный ИИ Супервайзер** → **Настройки** → **Обработчики**.
2. Нажмите **Создать ключ доступа**.
3. Скопируйте ключ: он показывается один раз и после обновления страницы больше не будет доступен.
4. В разделе **Настройки** → **ИИ-анализ** проверьте выбранную модель (см. раздел **Модели**).

Если модуль устанавливается впервые, ключ можно создать в мастере настройки на шаге **Локальный воркер**: нажмите **Создать ключ доступа**, а приложение для macOS скачивать не нужно.

{% hint style="info" %}
Для каждого обработчика создавайте отдельный ключ: ключ привязывается к одному обработчику.
{% endhint %}

### Шаг 4. Файл compose.yaml

Создайте каталог обработчика и файл `compose.yaml`:

```bash
sudo mkdir -p /opt/mikopbx-ai-supervisor-worker
sudo chown "$USER" /opt/mikopbx-ai-supervisor-worker
cd /opt/mikopbx-ai-supervisor-worker

cat > compose.yaml <<'EOF'
name: ai-supervisor-worker

services:
  worker:
    image: excla1m/mikopbx-ai-supervisor-worker:latest
    container_name: ai-supervisor-worker
    restart: unless-stopped
    depends_on:
      - ollama
    env_file:
      - path: .env
        required: false
    volumes:
      - data:/data
      - models:/models
    read_only: true
    tmpfs:
      - /tmp:size=512m
    cap_drop:
      - ALL
    security_opt:
      - no-new-privileges:true
    stop_grace_period: 90s
    logging:
      driver: json-file
      options:
        max-size: 20m
        max-file: "5"

  ollama:
    image: ollama/ollama:0.34.2
    container_name: ai-supervisor-ollama
    restart: unless-stopped
    environment:
      OLLAMA_NUM_PARALLEL: "1"
      OLLAMA_KEEP_ALIVE: "30m"
    volumes:
      - ollama:/root/.ollama
    # Видеокарта NVIDIA (шаг 2). Без видеокарты удалите блок deploy целиком.
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all
              capabilities: [gpu]
    logging:
      driver: json-file
      options:
        max-size: 20m
        max-file: "3"

volumes:
  data:
  models:
  ollama:
EOF
```

| Параметр                                     | Назначение                                                                                                      |
| -------------------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| `name: ai-supervisor-worker`                 | Фиксирует имена томов (`ai-supervisor-worker_data` и другие) независимо от имени каталога.                      |
| `restart: unless-stopped`                    | Обработчик и Ollama запускаются вместе с Docker после перезагрузки сервера.                                     |
| `volumes`                                    | `data` - настройки, UID и статус обработчика; `models` - модель SenseVoice; `ollama` - языковые модели.         |
| `read_only`, `cap_drop`, `no-new-privileges` | Обработчик не может писать никуда, кроме томов и `/tmp`, и работает без привилегий от пользователя с uid 10001. |
| `stop_grace_period: 90s`                     | Время, за которое текущее задание успевает завершиться или вернуться в очередь при остановке.                   |
| `OLLAMA_NUM_PARALLEL: "1"`                   | Обработчик анализирует по одному звонку, Ollama не делит память между запросами.                                |
| `OLLAMA_KEEP_ALIVE: "30m"`                   | Модель остается загруженной между заданиями и не перечитывается с диска для каждого звонка.                     |
| `deploy.resources...` (сервис `ollama`)      | Передает видеокарту NVIDIA контейнеру Ollama. Обработчику видеокарта не нужна.                                  |

Порт Ollama наружу не публикуется: к ней обращается только обработчик по внутренней сети Docker.

### Шаг 5. Получение образов

```bash
docker compose pull
```

Docker сам выберет архитектуру сервера (x86\_64 или arm64). Образ обработчика занимает около 1 ГБ на диске, образ Ollama - около 9 ГБ.

### Шаг 6. Подключение к MikoPBX (setup)

Запустите мастер настройки:

```bash
docker compose run --rm worker setup
```

Мастер спросит:

1. **Адрес MikoPBX** - `https://<адрес-mikopbx>` без пути.
2. **Проверять ли сертификат MikoPBX** - по умолчанию нет (см. раздел TLS).
3. **Ключ обработчика** - ключ из шага 3, ввод скрыт.
4. **Имя обработчика** - как он будет показан в разделе **Обработчики**.
5. **Адрес локальной модели** - оставьте `http://ollama:11434/v1`.
6. **Модель для подготовки** - модель, которую обработчик скачает до первого задания. Укажите модель профиля, выбранного в MikoPBX: `qwen3.5:4b` или `qwen3.5:9b`.

Затем мастер проверит подключение и сохранит настройки в томе `data` (`/data/config.json`, права 600). Пример:

```
AI Supervisor Worker setup. Settings are saved to /data/config.json.
In MikoPBX open AI Supervisor → Workers and generate a worker API key first.

PBX address (https://host[:port]): https://pbx.example.com
Verify the PBX certificate? (yes/no; no accepts a self-signed one) [no]:
Worker API key:
Worker name shown in MikoPBX [Docker AI worker]: GPU AI worker
Local AI runtime (OpenAI-compatible or Ollama URL) [http://ollama:11434/v1]:
Model to prepare before the first job [qwen3.5:4b]:

Checking the PBX…
  OK: ModuleAISupervisor 1.162.1, worker API v2; key accepted
Checking the local AI runtime…
  OK: Local AI endpoint is reachable and model "qwen3.5:4b" is installed.

Saved. Worker UID ai-docker-e16a7a84fc7a4919. Start the worker with `docker compose up -d`.
```

При первой установке модели в Ollama еще нет, и вместо последней проверки будет строка `Reachable: Local runtime is reachable, but model "qwen3.5:4b" is not installed yet.` - это нормально: модель скачается после запуска.

<details>

<summary>Настройка без вопросов (для скриптов)</summary>

Сохраните ключ в файл, доступный пользователю контейнера (uid 10001), и передайте ответы флагами:

```bash
install -m 600 /dev/null worker.key
nano worker.key                                   # вставьте ключ и сохраните
sudo chown 10001 worker.key

docker compose run --rm -v "$PWD/worker.key:/run/worker.key:ro" worker setup \
  --pbx-url https://<адрес-mikopbx> \
  --token-file /run/worker.key \
  --name "GPU AI worker" \
  --ai-model qwen3.5:4b \
  --yes

sudo rm -f worker.key                             # ключ уже сохранен в томе data
```

</details>

{% hint style="info" %}
Чтобы изменить адрес, ключ, имя или модель, запустите `setup` еще раз. UID обработчика сохраняется в томе `data` и не меняется.
{% endhint %}

### Шаг 7. Запуск

```bash
docker compose up -d
docker compose logs -f worker
```

При первом запуске обработчик регистрируется в MikoPBX и скачивает модель в Ollama (`qwen3.5:4b` - 3,3 ГБ, несколько минут), после чего начинает брать задания:

```
INFO [worker] PBX https://pbx.example.com, TLS verification disabled (any PBX certificate is accepted)
INFO [worker] Local AI http://ollama:11434/v1, model qwen3.5:4b until the PBX assigns one
INFO [worker] AI Supervisor Worker 0.1.7 (linux) starting as GPU AI worker [ai-docker-e16a7a84fc7a4919]
INFO [ai-worker] Registered AI worker (server id 5), server poll interval 15.0s
INFO [ai-model] Local AI endpoint is reachable and model "qwen3.5:4b" is installed.
INFO [ai-worker] Claimed AI job #1566 call=mikopbx-1792739827.22 model=qwen3.5:4b
INFO [ai-worker] Submitted AI result for job #1566
```

### Проверка работы

1. В MikoPBX в разделе **Настройки** → **Обработчики** появился обработчик со статусом **Онлайн** и его IP-адресом.
2. Команда `status` показывает состояние и счетчик заданий:

   ```bash
   docker compose exec worker ai-supervisor-worker status
   ```

   ```
   State:        processing a job
   Stage:        Analyzing — Analyzing AI job #1522
   Job:          #1522 call mikopbx-1792394038.89 (25 %)
   PBX:          https://pbx.example.com (Connected)
   Worker:       GPU AI worker [ai-docker-e16a7a84fc7a4919]
   Model:        qwen3.5:4b — Local AI endpoint is reachable and model "qwen3.5:4b" is installed.
   Uptime:       21 min 21 s
   Jobs:         26 done, 0 failed, 0 handed back
   Last poll:    7 s ago
   Heartbeat:    6 s ago
   ```

3. Команда `check` проверяет ключ, модель и среду анализа голоса без запуска заданий:

   ```bash
   docker compose exec worker ai-supervisor-worker check
   ```

   ```
   Worker:      GPU AI worker [ai-docker-e16a7a84fc7a4919]
   TLS:         disabled (any PBX certificate is accepted)
   PBX:         OK — https://pbx.example.com: ModuleAISupervisor 1.162.1, worker API v2; key accepted
   Local AI:    OK — http://ollama:11434/v1: Local AI endpoint is reachable and model "qwen3.5:4b" is installed.
   ONNX:        OK — ONNX Runtime 1.29.1
   ```

4. Если используется видеокарта, проверьте, что модель загружена в нее целиком:

   ```bash
   docker compose exec ollama ollama ps
   ```

   ```
   NAME          ID              SIZE      PROCESSOR    CONTEXT    UNTIL
   qwen3.5:4b    d8b0f5e9760c    3.3 GB    100% GPU     8192       29 minutes from now
   ```

   Значение вида `13%/87% CPU/GPU` означает, что модель не поместилась в видеопамять и часть работы выполняет процессор. Выберите профиль модели меньше или уменьшите размер контекста в MikoPBX.

5. В журнале видны задания анализа звонка и анализа эмоций по голосу. При первом задании по голосу обработчик скачивает модель SenseVoice:

   ```
   INFO [ai-worker] Claimed AI job #1565 call=mikopbx-1792728833.20 model=qwen3.5:4b
   INFO [ai-worker] Submitted AI result for job #1565
   INFO [ai-worker] Claimed Voice Acoustic Emotion job #1567 call=mikopbx-1792739827.22 model=auto
   INFO [voice-analytics] Acoustic model SenseVoice Small (ONNX) is missing. Downloading it before acoustic patch.
   INFO [voice-model] SenseVoice Small (ONNX): ready
   INFO [voice-analytics] Submitted acoustic voice result for job #1567
   ```

6. В карточке звонка появились сводка, оценки и, если компонент включен, эмоции по голосу.

На видеокарте анализ звонка моделью `qwen3.5:4b` обычно занимает 30-60 секунд, анализ эмоций по голосу - 5-20 секунд.

### Управление

| Задача                               | Команда                                                              |
| ------------------------------------ | -------------------------------------------------------------------- |
| Состояние                            | `docker compose exec worker ai-supervisor-worker status`             |
| Проверить подключение, ключ и модель | `docker compose exec worker ai-supervisor-worker check`              |
| Журнал обработчика                   | `docker compose logs -f --tail 100 worker`                           |
| Загруженные модели                   | `docker compose exec ollama ollama ps`                               |
| Остановить                           | `docker compose stop`                                                |
| Запустить                            | `docker compose start`                                               |
| Перезапустить                        | `docker compose restart`                                             |
| Изменить настройки                   | `docker compose run --rm worker setup`, затем `docker compose up -d` |
| Версия                               | `docker compose exec worker ai-supervisor-worker version`            |

Все команды выполняются в каталоге `/opt/mikopbx-ai-supervisor-worker`. При остановке текущее задание завершается или возвращается в очередь MikoPBX, новые звонки ждут в очереди.

### Обновление

Скачайте новые версии образов и пересоздайте контейнеры. Модели и настройки хранятся в томах и сохраняются.

```bash
cd /opt/mikopbx-ai-supervisor-worker
docker compose pull
docker compose up -d
docker compose exec worker ai-supervisor-worker status
```

Чтобы закрепить конкретную версию, укажите в строке `image` тег вида `excla1m/mikopbx-ai-supervisor-worker:<версия>-cpu`, например `0.1.7-cpu`. Список версий есть на вкладке **Tags** в [Docker Hub](https://hub.docker.com/r/excla1m/mikopbx-ai-supervisor-worker/tags).

{% hint style="info" %}
Суффикс `cpu` относится к самому обработчику: ему видеокарта не нужна. Видеокарту использует контейнер Ollama, поэтому тот же образ работает и на серверах с NVIDIA.
{% endhint %}

### Удаление

```bash
cd /opt/mikopbx-ai-supervisor-worker
docker compose down -v         # контейнеры и тома: модели, настройки, UID
docker image rm excla1m/mikopbx-ai-supervisor-worker:latest ollama/ollama:0.34.2
```

Затем в MikoPBX в разделе **Обработчики** удалите ключ доступа этого обработчика. Без `-v` тома сохраняются, и при повторной установке модели не придется скачивать заново.

### Настройка через .env

Вместо `setup` параметры можно задать в файле `.env` рядом с `compose.yaml`. Непустая переменная окружения всегда важнее значения, сохраненного мастером.

```bash
cat > .env <<'EOF'
PBX_URL=https://<адрес-mikopbx>
WORKER_TOKEN=<ключ-обработчика>
WORKER_NAME=GPU AI worker
AI_MODEL=qwen3.5:4b
EOF
chmod 600 .env
docker compose up -d
```

| Переменная                           | По умолчанию                       | Назначение                                                                                   |
| ------------------------------------ | ---------------------------------- | -------------------------------------------------------------------------------------------- |
| `PBX_URL`                            | -                                  | Адрес MikoPBX: `https://host[:port]`, без пути                                               |
| `WORKER_TOKEN` / `WORKER_TOKEN_FILE` | -                                  | Ключ обработчика строкой или путь к файлу внутри контейнера                                  |
| `WORKER_NAME`                        | `Docker AI worker`                 | Имя в разделе **Обработчики**                                                                |
| `WORKER_UID`                         | создается автоматически            | Постоянный идентификатор; ключ привязывается к одному UID                                    |
| `PBX_TLS_VERIFY`                     | `false`                            | `true` - отклонять недоверенный сертификат MikoPBX                                           |
| `PBX_CA_FILE`                        | -                                  | Дополнительный CA (PEM или DER) при включенной проверке TLS                                  |
| `AI_BASE_URL`                        | `http://ollama:11434/v1`           | Адрес локальной модели: Ollama или OpenAI-совместимый сервер                                 |
| `AI_MODEL`                           | `qwen3.5:4b`                       | Модель, которую обработчик скачивает до первого задания. Модель для анализа выбирается в MikoPBX |
| `AI_TIMEOUT`                         | `300`                              | Ограничение одного запроса к модели, секунд (профиль в MikoPBX может его изменить)          |
| `POLL_INTERVAL`                      | `15`                               | Пауза между опросами очереди в простое, секунд                                               |
| `HEARTBEAT_INTERVAL`                 | `60`                               | Интервал продления аренды задания, секунд                                                    |
| `API_TIMEOUT`                        | `30`                               | Ограничение одного запроса к MikoPBX, секунд                                                 |
| `ACOUSTIC_TIMEOUT`                   | `60`                               | Ограничение анализа эмоций по голосу для одного звонка, секунд                               |
| `ONNX_THREADS`                       | `0`                                | Потоки процессора для анализа голоса, `0` - все ядра                                         |
| `LOG_LEVEL`                          | `INFO`                             | `DEBUG`, `INFO`, `WARNING`, `ERROR`                                                          |

{% hint style="info" %}
`AI_MODEL` необязательна: если в MikoPBX выбрана другая модель, обработчик скачает ее при первом задании. Переменная нужна, чтобы подготовить модель заранее и первое задание не ждало загрузки нескольких гигабайт.
{% endhint %}

### Другой сервер с моделью

Если языковая модель уже работает на другом сервере (Ollama, vLLM, LM Studio или другой OpenAI-совместимый сервер), контейнер Ollama можно не запускать:

1. Удалите из `compose.yaml` сервис `ollama`, строку `depends_on` и том `ollama`.
2. Укажите адрес сервера в `.env`: `AI_BASE_URL=http://<адрес-сервера>:11434/v1`.
3. Перезапустите обработчик: `docker compose up -d`.

Модель для каждого задания по-прежнему выбирается в MikoPBX, поэтому она должна быть доступна на этом сервере.

### TLS

Проверка сертификата по умолчанию выключена: обработчик принимает любой сертификат MikoPBX (самоподписанный, выпущенный на другое имя, при обращении по IP), трафик при этом все равно шифруется. Это поведение совпадает с AI Supervisor Worker для macOS.

Чтобы отклонять недоверенные сертификаты, включите проверку в `setup` или задайте `PBX_TLS_VERIFY=true`. Используется системное хранилище CA; для собственного центра сертификации передайте файл через `PBX_CA_FILE` и подключите его в контейнер томом.

### Диагностика проблем

| Симптом                                                                    | Причина и решение                                                                                                                                                 |
| -------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `status`: `update required (module or worker)`                             | Версия модуля и обработчика не совместимы. Обновите **Локальный ИИ Супервайзер** или образ обработчика. Обработчик повторяет проверку каждые 15 минут.            |
| `status`: `the PBX rejected the worker key`                                | Ключ удален, введен с ошибкой или уже привязан к другому обработчику. Создайте новый ключ в разделе **Обработчики** и запустите `setup` еще раз.                   |
| `status`: `PBX unreachable, waiting to retry`                              | Нет связи с MikoPBX. Проверьте `curl -skI https://<адрес-mikopbx>` с сервера. Обработчик сам повторяет подключение с растущим интервалом до 30 минут.              |
| `could not select device driver "nvidia"` при запуске                      | Не установлен NVIDIA Container Toolkit или Docker не перезапущен после его настройки. Повторите шаг 2.                                                            |
| `ollama ps` показывает `100% CPU` при наличии видеокарты                   | Контейнер Ollama не получил видеокарту: проверьте блок `deploy` в `compose.yaml` и команду `docker run --rm --gpus all ubuntu nvidia-smi`.                       |
| `ollama ps` показывает `CPU/GPU`, анализ медленный                         | Модель не помещается в видеопамять. Выберите профиль **Qwen3.5 4B Q4** или уменьшите размер контекста в MikoPBX.                                                 |
| Задания завершаются по тайм-ауту                                           | Модель работает слишком медленно, обычно на процессоре без видеокарты. Используйте видеокарту или сервер с более быстрым процессором.                             |
| Модель не скачивается                                                      | Нет доступа к registry.ollama.ai. Откройте исходящий HTTPS для сервера или прокси.                                                                                |
| Не скачивается модель SenseVoice                                           | Нет доступа к huggingface.co. Откройте исходящий HTTPS для сервера или прокси.                                                                                    |
| `docker pull` завершается ошибкой `connection reset` или по тайм-ауту      | Нет доступа к Docker Hub. Откройте исходящий HTTPS до `registry-1.docker.io` и `production.cloudfront.docker.com` или настройте зеркало реестра.                  |
| Ошибка TLS при подключении                                                 | Включена проверка сертификата, а сертификат MikoPBX недоверенный. Выключите проверку или передайте свой CA.                                                       |
| Контейнер `unhealthy`                                                      | Цикл обработчика не отчитывался 10 минут. Посмотрите `docker compose logs --tail 200 worker`.                                                                     |

### Данные и безопасность

| Данные             | Где хранятся                                                            |
| ------------------ | ----------------------------------------------------------------------- |
| Языковые модели    | том `ai-supervisor-worker_ollama`                                       |
| Модель SenseVoice  | том `ai-supervisor-worker_models` (`/models`)                           |
| Настройки и ключ   | том `ai-supervisor-worker_data`, файл `/data/config.json` с правами 600 |
| UID обработчика    | том `ai-supervisor-worker_data`, файл `/data/worker_uid`                |
| Журналы            | журнал Docker и файл в `/data/logs`                                     |
| Временные записи   | том `ai-supervisor-worker_data`, удаляются после задания                |

Обработчик подключается к MikoPBX только исходящими HTTPS-запросами, получает расшифровки и записи только назначенных ему заданий и не хранит их после обработки. Языковая модель работает внутри вашей сети и не отправляет данные во внешние сервисы. Ключ обработчика дает доступ только к API модуля ИИ Супервайзера.
