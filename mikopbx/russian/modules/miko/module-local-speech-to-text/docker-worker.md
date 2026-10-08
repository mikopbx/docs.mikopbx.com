---
description: >-
  Docker-обработчик для локальной транскрибации: установка на Linux-сервер с
  нуля, подключение к MikoPBX, проверка работы, обновление и удаление.
---

# Docker-обработчик (Linux)

**Docker-обработчик** - контейнер для локального распознавания записей звонков на вашем Linux-сервере. Он берет задания из очереди модуля **Локальная транскрибация**, скачивает запись, распознает ее выбранной в MikoPBX моделью и отправляет на станцию сегменты с таймкодами и техническую диагностику. Записи не покидают вашу сеть.

Docker-обработчик - альтернатива [Local STT Worker для macOS](miko-ai-worker.md). На одной станции в каждый момент работает одна платформа: какая именно, определяется моделью, выбранной на вкладке **Каталог моделей**.

{% hint style="warning" %}
Образ пока не опубликован. Значения в угловых скобках, например `ghcr.io/mikopbx/<имя-образа>:<версия>-cpu`, - заглушки: их нужно заменить точным именем и версией образа после публикации в GitHub Container Registry.
{% endhint %}

### Требования

| Компонент | Минимум | Рекомендуется |
| --- | --- | --- |
| ОС | 64-битный Linux (x86_64 или arm64) | Ubuntu Server 24.04 LTS или Debian 12 |
| Процессор | 4 ядра | 8 ядер с поддержкой AVX2 |
| Память | 4 ГБ | 8 ГБ (нужно для T-one) |
| Диск | 30 ГБ | 50 ГБ |
| Docker | Docker Engine с плагином Compose 2.24 или новее | последняя стабильная версия |
| Сеть | исходящий HTTPS до MikoPBX и до huggingface.co | то же; входящие порты не нужны |
| MikoPBX | 2025.1.1 и модуль с платформой **Linux (Docker)** в **Каталоге моделей** | последняя версия модуля |

{% hint style="info" %}
На виртуальной машине VMware проверьте, что режим совместимости процессора (EVC) не скрывает инструкции AVX2: без них распознавание работает в несколько раз медленнее. Проверка показана в шаге 1.
{% endhint %}

### Модели

Модель выбирается в MikoPBX, обработчик скачивает ее при первом запуске и хранит в томе `models`. Каждая модель закреплена на конкретной версии, а обработчик сверяет SHA-256 каждого файла и не загружает модель при расхождении.

| Модель | Языки | Когда выбирать | Размер |
| --- | --- | --- | --- |
| **GigaAM v3** (по умолчанию) | русский | Русские звонки: лучшая точность, пунктуация, цифры | 0,9 ГБ |
| **T-one** | русский | Русская телефония, без пунктуации | 5,6 ГБ |
| **Whisper Large V3 Turbo** | около 99 языков | Любые языки и смешанные звонки; самая медленная на CPU | 1,6 ГБ |
| **Parakeet TDT 0.6B v3** | 25 европейских | Европейские языки | 2,5 ГБ |
| **GigaAM Multilingual** | казахский, киргизский, узбекский, русский, английский | Языки Средней Азии, без пунктуации | 0,9 ГБ |

### Шаг 1. Подготовка сервера

Установите Docker Engine и плагин Compose:

```bash
curl -fsSL https://get.docker.com | sudo sh
sudo usermod -aG docker "$USER"      # чтобы запускать docker без sudo
newgrp docker                        # или перезайдите в сессию
```

{% hint style="info" %}
Если `get.docker.com` недоступен с сервера, в Ubuntu и Debian установите Docker из пакетов дистрибутива: `sudo apt-get install -y docker.io docker-compose-v2`.
{% endhint %}

Проверьте установку и процессор:

```bash
docker version
docker compose version
uname -m                                       # x86_64 или aarch64
grep -o -w avx2 /proc/cpuinfo | head -1        # на x86_64 должно вывести avx2
```

Проверьте доступ к MikoPBX и к хранилищу моделей:

```bash
curl -skI https://<адрес-mikopbx> | head -1
curl -sI https://huggingface.co | head -1
```

### Шаг 2. Подготовка MikoPBX

1. Откройте **Модули** → **Локальная транскрибация** → вкладка **Каталог моделей**.
2. Переключите платформу на **Linux (Docker)** и выберите модель, например **GigaAM v3**.
3. Перейдите на вкладку **Обработчики** и нажмите **Создать ключ доступа**.
4. Скопируйте ключ: он показывается один раз и после обновления страницы больше не будет доступен.

{% hint style="info" %}
Если на станции выбрана модель платформы macOS, Docker-обработчик подключится, но не будет брать задания, а `status` покажет `waiting: MikoPBX has a macOS model selected (choose a Linux model)`.
{% endhint %}

### Шаг 3. Получение образа

Выберите один из вариантов.

{% tabs %}
{% tab title="Из реестра GitHub" %}
```bash
docker pull ghcr.io/mikopbx/<имя-образа>:<версия>-cpu
```

Docker сам выберет архитектуру сервера (x86_64 или arm64).
{% endtab %}

{% tab title="Из архива релиза" %}
Для серверов без доступа к реестру образ выкладывается архивом вместе с контрольными суммами. Скачивайте архив под архитектуру сервера: `x86_64` или `arm64`.

```bash
mkdir -p ~/stt-image && cd ~/stt-image
curl -fLO <ссылка-на-релиз>/docker-stt-worker-<версия>-x86_64-cpu-docker.tar
curl -fLO <ссылка-на-релиз>/checksum.sha256

sha256sum -c checksum.sha256 --ignore-missing   # должно вывести: OK
docker load -i docker-stt-worker-<версия>-x86_64-cpu-docker.tar
```

Последняя строка `docker load` покажет имя загруженного образа: используйте его в шаге 4.

{% hint style="warning" %}
Если контрольная сумма не совпала, архив поврежден при скачивании. Скачайте его заново: браузер может испортить большой файл при прерванной загрузке.
{% endhint %}
{% endtab %}
{% endtabs %}

### Шаг 4. Файл compose.yaml

Создайте каталог обработчика и файл `compose.yaml`. В строке `image` укажите образ из шага 3.

```bash
sudo mkdir -p /opt/mikopbx-stt-worker
sudo chown "$USER" /opt/mikopbx-stt-worker
cd /opt/mikopbx-stt-worker

cat > compose.yaml <<'EOF'
name: local-stt-worker

services:
  worker:
    image: ghcr.io/mikopbx/<имя-образа>:<версия>-cpu
    container_name: local-stt-worker
    restart: unless-stopped
    env_file:
      - path: .env
        required: false
    volumes:
      - models:/models
      - data:/data
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

volumes:
  models:
  data:
EOF
```

| Параметр | Назначение |
| --- | --- |
| `name: local-stt-worker` | Фиксирует имена томов (`local-stt-worker_models`, `local-stt-worker_data`) независимо от имени каталога. |
| `restart: unless-stopped` | Обработчик запускается вместе с Docker после перезагрузки сервера. |
| `volumes` | `models` - скачанные модели, `data` - настройки, UID и статус обработчика. |
| `read_only`, `cap_drop`, `no-new-privileges` | Контейнер не может писать никуда, кроме томов и `/tmp`, и работает без привилегий от пользователя с uid 10001. |
| `stop_grace_period: 90s` | Время, за которое текущее задание успевает завершиться или вернуться в очередь при остановке. |

### Шаг 5. Подключение к MikoPBX (setup)

Запустите мастер настройки:

```bash
docker compose run --rm worker setup
```

Мастер спросит:

1. **Адрес MikoPBX** - `https://<адрес-mikopbx>` без пути.
2. **Проверять ли сертификат MikoPBX** - по умолчанию нет (см. раздел TLS).
3. **Ключ обработчика** - ключ из шага 2, ввод скрыт.
4. **Имя обработчика** - как он будет показан на вкладке **Обработчики**.

Затем мастер проверит подключение, зарегистрирует обработчик и сохранит настройки в томе `data` (`/data/config.json`, права 600). Пример успешного результата:

```
Connecting to https://pbx.example.com as docker-stt-031c9681 ...
  OK  ModuleLocalSpeechToText 1.118
  OK  registered as worker #6
  OK  selected model: GigaAM v3
  No NVIDIA GPU: models run on the CPU.

Saved /data/config.json.
```

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
  --name "Docker STT" \
  --yes

sudo rm -f worker.key                             # ключ уже сохранен в томе data
```

</details>

{% hint style="info" %}
Чтобы изменить адрес, ключ или имя, запустите `setup` еще раз. UID обработчика сохраняется в томе `data` и не меняется.
{% endhint %}

### Шаг 6. Запуск

```bash
docker compose up -d
docker compose exec worker local-stt-worker status
```

При первом запуске обработчик скачивает модель (GigaAM v3 - около 0,9 ГБ, около 2-3 минут при скорости 50 Мбит/с) и проверяет ее контрольные суммы, после чего начинает брать задания. Пока модель загружается, `status` показывает `downloading / loading the model`.

### Проверка работы

1. В MikoPBX на вкладке **Обработчики** появился обработчик со статусом **Онлайн**, его IP и модель.
2. `status` показывает `processing a job` или `waiting for jobs` и растущий счетчик заданий:

```
Worker            Docker STT (docker-stt-031c9681), version <версия>
PBX               https://pbx.example.com
State             processing a job
Model             GigaAM v3
Engine            GigaAM (onnx-asr) on cpu
Jobs              50 done (2:00:32 of audio), 0 failed, 0 handed back
Last job          #399 16s ago: 70.388s audio in 6.791s (RTF 0.0965), 25 segments
```

3. В журнале контейнера видно каждое задание:

```bash
docker compose logs -f
```

```
Claimed job #3087 (call mikopbx-..., model istupakov/gigaam-v3-onnx/v3_e2e_rnnt)
Job #3087 done: 215 segments, language ru, 70.9s for 769.0s of audio (RTF 0.0923)
```

4. В карточке звонка появилась расшифровка, а в ее технической информации - движок, устройство и время обработки.

**RTF** - отношение времени обработки к длительности записи. Например, RTF 0,1 означает, что 10-минутный звонок распознается за минуту.

### Управление

| Задача | Команда |
| --- | --- |
| Состояние | `docker compose exec worker local-stt-worker status` |
| Журнал | `docker compose logs -f --tail 100` |
| Проверить подключение и ключ | `docker compose run --rm worker check` |
| Остановить | `docker compose stop` |
| Запустить | `docker compose start` |
| Перезапустить | `docker compose restart` |
| Изменить настройки | `docker compose run --rm worker setup`, затем `docker compose up -d` |

Все команды выполняются в каталоге `/opt/mikopbx-stt-worker`. При остановке текущее задание завершается или возвращается в очередь MikoPBX, новые звонки ждут в очереди.

### Обновление

Получите новый образ (шаг 3), укажите его версию в строке `image` файла `compose.yaml` и пересоздайте контейнер. Модели и настройки хранятся в томах и сохраняются.

```bash
cd /opt/mikopbx-stt-worker
nano compose.yaml              # image: ghcr.io/mikopbx/<имя-образа>:<новая-версия>-cpu
docker compose pull            # для архива вместо этого: docker load -i <новый-архив>.tar
docker compose up -d
docker compose exec worker local-stt-worker status
```

### Удаление

```bash
cd /opt/mikopbx-stt-worker
docker compose down -v         # контейнер и тома: модели, настройки, UID
docker image rm ghcr.io/mikopbx/<имя-образа>:<версия>-cpu
```

Затем в MikoPBX на вкладке **Обработчики** удалите ключ доступа этого обработчика. Без `-v` тома сохраняются, и при повторной установке модель не придется скачивать заново.

### Настройка через .env

Вместо `setup` параметры можно задать в файле `.env` рядом с `compose.yaml`. Непустая переменная окружения всегда важнее значения, сохраненного мастером.

```bash
cat > .env <<'EOF'
PBX_URL=https://<адрес-mikopbx>
WORKER_TOKEN=<ключ-обработчика>
WORKER_NAME=Docker STT
EOF
chmod 600 .env
docker compose up -d
```

| Переменная | По умолчанию | Назначение |
| --- | --- | --- |
| `PBX_URL` | - | Адрес MikoPBX: `https://host[:port]`, без пути |
| `WORKER_TOKEN` / `WORKER_TOKEN_FILE` | - | Ключ обработчика строкой или путь к файлу внутри контейнера |
| `WORKER_NAME` | `Docker STT worker` | Имя на вкладке **Обработчики** |
| `WORKER_UID` | создается автоматически | Постоянный идентификатор; ключ привязывается к одному UID |
| `PBX_TLS_VERIFY` | `false` | `true` - отклонять недоверенный сертификат MikoPBX |
| `PBX_CA_FILE` | - | Дополнительный CA (PEM или DER) при включенной проверке TLS |
| `POLL_INTERVAL` | `5` | Пауза между опросами очереди в простое, секунд |
| `HEARTBEAT_INTERVAL` | `60` | Интервал продления аренды задания, секунд |
| `LOG_LEVEL` | `INFO` | `DEBUG`, `INFO`, `WARNING`, `ERROR` |

### TLS

Проверка сертификата по умолчанию выключена: обработчик принимает любой сертификат MikoPBX (самоподписанный, выпущенный на другое имя, при обращении по IP), трафик при этом все равно шифруется. Это поведение совпадает с Local STT Worker для macOS.

Чтобы отклонять недоверенные сертификаты, включите проверку в `setup` или задайте `PBX_TLS_VERIFY=true`. Используется системное хранилище CA; для собственного центра сертификации передайте файл через `PBX_CA_FILE` и подключите его в контейнер томом.

### Диагностика проблем

| Симптом | Причина и решение |
| --- | --- |
| `status`: `waiting: MikoPBX has a macOS model selected (choose a Linux model)` | На станции выбрана модель платформы macOS. Переключите **Каталог моделей** на **Linux (Docker)** и выберите модель. |
| `status`: `waiting: update ModuleLocalSpeechToText` | Версия модуля не поддерживает платформу Linux. Обновите **Локальную транскрибацию**. |
| `status`: `waiting: this image cannot run the selected model` | Выбранная модель не поддерживается этой версией образа. Обновите образ или выберите другую модель. |
| `MikoPBX rejected the worker key` | Ключ удален, введен с ошибкой или уже привязан к другому обработчику. Создайте новый ключ на вкладке **Обработчики** и запустите `setup` еще раз. |
| Ошибка TLS при подключении | Включена проверка сертификата, а сертификат MikoPBX недоверенный. Выключите проверку или передайте свой CA. |
| Модель не скачивается | Нет доступа к huggingface.co. Откройте исходящий HTTPS для сервера или прокси. |
| Распознавание медленное (RTF больше 0,3 для GigaAM) | Мало ядер или процессор без AVX2. Добавьте ядра виртуальной машине и проверьте режим EVC. |
| Контейнер `unhealthy` | Цикл обработчика не отчитывался 10 минут. Посмотрите `docker compose logs --tail 200`. |

{% hint style="info" %}
Для качественной разбивки на фразы включите **Определение голосовой активности (VAD)** в настройках модуля: без него запись режется на окна фиксированной длины, а не по паузам в речи.
{% endhint %}

### Данные и безопасность

| Данные | Где хранятся |
| --- | --- |
| Модели | том `local-stt-worker_models` (`/models`) |
| Настройки и ключ | том `local-stt-worker_data`, файл `/data/config.json` с правами 600 |
| UID обработчика | том `local-stt-worker_data`, файл `/data/worker_uid` |
| Временное аудио | `/tmp` внутри контейнера, в памяти, удаляется после задания |
| Журналы | журнал Docker, до 5 файлов по 20 МБ |

Обработчик подключается к MikoPBX только исходящими HTTPS-запросами, получает записи только назначенных ему заданий и не хранит их после обработки. Ключ обработчика дает доступ только к API модуля транскрибации.
