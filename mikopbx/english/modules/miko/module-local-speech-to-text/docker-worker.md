---
description: >-
  Docker worker for local transcription: installing it on a Linux server from
  scratch, connecting it to MikoPBX, checking that it works, updating and
  removing it.
---

# Docker worker (Linux)

The **Docker worker** is a container that recognizes call recordings locally on your Linux server. It takes jobs from the queue of the **Local Transcription** module, downloads the recording, recognizes it with the model selected in MikoPBX, and sends timestamped segments and technical diagnostics back to the PBX. Recordings never leave your network.

The Docker worker is an alternative to [Local STT Worker for macOS](miko-ai-worker.md). Only one platform works on a PBX at a time: the model selected on the **Model marketplace** tab decides which one.

The image is published on Docker Hub: [`excla1m/mikopbx-stt-worker`](https://hub.docker.com/r/excla1m/mikopbx-stt-worker). The `latest` tag always points to the latest stable version.

### Requirements

| Component | Minimum | Recommended |
| --- | --- | --- |
| OS | 64-bit Linux (x86_64 or arm64) | Ubuntu Server 24.04 LTS or Debian 12 |
| CPU | 4 cores | 8 cores with AVX2 support |
| Memory | 4 GB | 8 GB (required for T-one) |
| Disk | 30 GB | 50 GB |
| Docker | Docker Engine with the Compose plugin 2.24 or newer | latest stable version |
| Network | outbound HTTPS to MikoPBX, Docker Hub and huggingface.co | the same; no inbound ports are needed |
| MikoPBX | 2025.1.1 and a module version with the **Linux (Docker)** platform on the **Model marketplace** tab | latest module version |

{% hint style="info" %}
On a VMware virtual machine, make sure the CPU compatibility mode (EVC) does not hide AVX2 instructions: without them recognition is several times slower. Step 1 shows how to check.
{% endhint %}

### Models

The model is selected in MikoPBX. The worker downloads it on first start and keeps it in the `models` volume. Every model is pinned to a specific version, and the worker checks the SHA-256 of each file and refuses to load the model on any mismatch.

| Model | Languages | When to choose | Size |
| --- | --- | --- | --- |
| **GigaAM v3** (default) | Russian | Russian calls: best accuracy, punctuation, numerals | 0.9 GB |
| **T-one** | Russian | Russian telephony, no punctuation | 5.6 GB |
| **Whisper Large V3 Turbo** | about 99 languages | Any language and mixed-language calls; the slowest on a CPU | 1.6 GB |
| **Parakeet TDT 0.6B v3** | 25 European | European languages | 2.5 GB |
| **GigaAM Multilingual** | Kazakh, Kyrgyz, Uzbek, Russian, English | Central Asian languages, no punctuation | 0.9 GB |

### Step 1. Prepare the server

Install Docker Engine and the Compose plugin:

```bash
curl -fsSL https://get.docker.com | sudo sh
sudo usermod -aG docker "$USER"      # to run docker without sudo
newgrp docker                        # or log in again
```

{% hint style="info" %}
If `get.docker.com` is not reachable from the server, on Ubuntu and Debian install Docker from the distribution packages: `sudo apt-get install -y docker.io docker-compose-v2`.
{% endhint %}

Check the installation and the CPU:

```bash
docker version
docker compose version
uname -m                                       # x86_64 or aarch64
grep -o -w avx2 /proc/cpuinfo | head -1        # on x86_64 it must print avx2
```

Check access to MikoPBX, Docker Hub and the model storage:

```bash
curl -skI https://<mikopbx-address> | head -1
curl -sI https://registry-1.docker.io/v2/ | head -1   # a 401 response is fine: the registry is reachable
curl -sI https://huggingface.co | head -1
```

### Step 2. Prepare MikoPBX

1. Open **Modules** → **Local Transcription** → the **Model marketplace** tab.
2. Switch the platform to **Linux (Docker)** and select a model, for example **GigaAM v3**.
3. Go to the **Workers** tab and click **Create API key**.
4. Copy the key: it is shown only once and is no longer available after the page is reloaded.

{% hint style="info" %}
If a macOS model is selected on the PBX, the Docker worker connects but does not take jobs, and `status` shows `waiting: MikoPBX has a macOS model selected (choose a Linux model)`.
{% endhint %}

### Step 3. Get the image

```bash
docker pull excla1m/mikopbx-stt-worker:latest
```

Docker picks the server architecture (x86_64 or arm64) automatically. The image takes about 2 GB.

{% hint style="info" %}
The download block on the **Workers** tab has a **Linux/macOS (Docker)** switch: it shows a ready `docker run` command with this PBX address and the key you have just created. It is handy for a quick check; for permanent use, take `compose.yaml` from step 4, which adds container restrictions and a clean shutdown.
{% endhint %}

### Step 4. The compose.yaml file

Create the worker directory and the `compose.yaml` file:

```bash
sudo mkdir -p /opt/mikopbx-stt-worker
sudo chown "$USER" /opt/mikopbx-stt-worker
cd /opt/mikopbx-stt-worker

cat > compose.yaml <<'EOF'
name: local-stt-worker

services:
  worker:
    image: excla1m/mikopbx-stt-worker:latest
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

| Parameter | Purpose |
| --- | --- |
| `name: local-stt-worker` | Fixes the volume names (`local-stt-worker_models`, `local-stt-worker_data`) whatever the directory is called. |
| `restart: unless-stopped` | The worker starts together with Docker after a server reboot. |
| `volumes` | `models` holds the downloaded models, `data` holds the worker settings, UID and status. |
| `read_only`, `cap_drop`, `no-new-privileges` | The container cannot write anywhere except the volumes and `/tmp`, and runs without privileges as uid 10001. |
| `stop_grace_period: 90s` | Time for the current job to finish or return to the queue when the worker stops. |

### Step 5. Connect to MikoPBX (setup)

Run the setup wizard:

```bash
docker compose run --rm worker setup
```

The wizard asks for:

1. **MikoPBX address**: `https://<mikopbx-address>` without a path.
2. **Whether to verify the MikoPBX certificate**: no by default (see the TLS section).
3. **Worker key**: the key from step 2; the input is hidden.
4. **Worker name**: how it is shown on the **Workers** tab.

The wizard then checks the connection, registers the worker and saves the settings in the `data` volume (`/data/config.json`, mode 600). Example of a successful run:

```
Connecting to https://pbx.example.com as docker-stt-031c9681 ...
  OK  ModuleLocalSpeechToText 1.118
  OK  registered as worker #6
  OK  selected model: GigaAM v3
  No NVIDIA GPU: models run on the CPU.

Saved /data/config.json.
```

<details>

<summary>Non-interactive setup (for scripts)</summary>

Save the key to a file readable by the container user (uid 10001) and pass the answers as flags:

```bash
install -m 600 /dev/null worker.key
nano worker.key                                   # paste the key and save
sudo chown 10001 worker.key

docker compose run --rm -v "$PWD/worker.key:/run/worker.key:ro" worker setup \
  --pbx-url https://<mikopbx-address> \
  --token-file /run/worker.key \
  --name "Docker STT" \
  --yes

sudo rm -f worker.key                             # the key is already saved in the data volume
```

</details>

{% hint style="info" %}
To change the address, key or name, run `setup` again. The worker UID is kept in the `data` volume and does not change.
{% endhint %}

### Step 6. Start

```bash
docker compose up -d
docker compose exec worker local-stt-worker status
```

On first start the worker downloads the model (GigaAM v3 is about 0.9 GB, about 2-3 minutes at 50 Mbit/s) and checks its checksums, then starts taking jobs. While the model is loading, `status` shows `downloading / loading the model`.

### Checking that it works

1. On the **Workers** tab in MikoPBX, the worker appears with the **Online** status, its IP address and model.
2. `status` shows `processing a job` or `waiting for jobs` and a growing job counter:

```
Worker            Docker STT (docker-stt-031c9681), version <version>
PBX               https://pbx.example.com
State             processing a job
Model             GigaAM v3
Engine            GigaAM (onnx-asr) on cpu
Jobs              50 done (2:00:32 of audio), 0 failed, 0 handed back
Last job          #399 16s ago: 70.388s audio in 6.791s (RTF 0.0965), 25 segments
```

3. The container log shows every job:

```bash
docker compose logs -f
```

```
Claimed job #3087 (call mikopbx-..., model istupakov/gigaam-v3-onnx/v3_e2e_rnnt)
Job #3087 done: 215 segments, language ru, 70.9s for 769.0s of audio (RTF 0.0923)
```

4. The call card shows the transcript, and its technical information shows the engine, device and processing time.

**RTF** is the ratio of processing time to recording length. For example, RTF 0.1 means a 10-minute call is recognized in one minute.

### Management

| Task | Command |
| --- | --- |
| Status | `docker compose exec worker local-stt-worker status` |
| Log | `docker compose logs -f --tail 100` |
| Check the connection and key | `docker compose run --rm worker check` |
| Stop | `docker compose stop` |
| Start | `docker compose start` |
| Restart | `docker compose restart` |
| Change settings | `docker compose run --rm worker setup`, then `docker compose up -d` |

Run all commands in the `/opt/mikopbx-stt-worker` directory. When the worker stops, the current job finishes or returns to the MikoPBX queue, and new calls wait in the queue.

### Updating

Pull the new image version and re-create the container. Models and settings are kept in the volumes.

```bash
cd /opt/mikopbx-stt-worker
docker compose pull
docker compose up -d
docker compose exec worker local-stt-worker status
```

When the container is re-created, the current job finishes or returns to the queue. To pin a specific version, set a tag such as `excla1m/mikopbx-stt-worker:<version>-cpu` in the `image` line; the list of versions is on the **Tags** tab on [Docker Hub](https://hub.docker.com/r/excla1m/mikopbx-stt-worker/tags).

### Removing

```bash
cd /opt/mikopbx-stt-worker
docker compose down -v         # the container and volumes: models, settings, UID
docker image rm excla1m/mikopbx-stt-worker:latest
```

Then delete this worker's API key on the **Workers** tab in MikoPBX. Without `-v` the volumes are kept, and a reinstall does not have to download the model again.

### Configuration with .env

Instead of `setup`, you can set the parameters in a `.env` file next to `compose.yaml`. A non-empty environment variable always takes precedence over the value saved by the wizard.

```bash
cat > .env <<'EOF'
PBX_URL=https://<mikopbx-address>
WORKER_TOKEN=<worker-key>
WORKER_NAME=Docker STT
EOF
chmod 600 .env
docker compose up -d
```

| Variable | Default | Purpose |
| --- | --- | --- |
| `PBX_URL` | - | MikoPBX address: `https://host[:port]`, no path |
| `WORKER_TOKEN` / `WORKER_TOKEN_FILE` | - | Worker key as a string or a path to a file inside the container |
| `WORKER_NAME` | `Docker STT worker` | Name on the **Workers** tab |
| `WORKER_UID` | generated automatically | Permanent identifier; a key is bound to one UID |
| `PBX_TLS_VERIFY` | `false` | `true` rejects an untrusted MikoPBX certificate |
| `PBX_CA_FILE` | - | Additional CA (PEM or DER) when TLS verification is on |
| `POLL_INTERVAL` | `5` | Pause between queue polls when idle, seconds |
| `HEARTBEAT_INTERVAL` | `60` | Job lease renewal interval, seconds |
| `LOG_LEVEL` | `INFO` | `DEBUG`, `INFO`, `WARNING`, `ERROR` |

### TLS

Certificate verification is off by default: the worker accepts any MikoPBX certificate (self-signed, issued for another name, or when connecting by IP address), and the traffic is still encrypted. This matches Local STT Worker for macOS.

To reject untrusted certificates, turn verification on in `setup` or set `PBX_TLS_VERIFY=true`. The system CA store is used; for your own certificate authority, pass its file with `PBX_CA_FILE` and mount it into the container as a volume.

### Troubleshooting

| Symptom | Cause and solution |
| --- | --- |
| `status`: `waiting: MikoPBX has a macOS model selected (choose a Linux model)` | A macOS model is selected on the PBX. Switch **Model marketplace** to **Linux (Docker)** and select a model. |
| `status`: `waiting: update ModuleLocalSpeechToText` | The module version does not support the Linux platform. Update **Local Transcription**. |
| `status`: `waiting: this image cannot run the selected model` | This image version does not support the selected model. Update the image or select another model. |
| `MikoPBX rejected the worker key` | The key was deleted, mistyped, or is already bound to another worker. Create a new key on the **Workers** tab and run `setup` again. |
| TLS error when connecting | Certificate verification is on, and the MikoPBX certificate is untrusted. Turn verification off or pass your CA. |
| `docker pull` fails with `connection reset` or a timeout | Docker Hub is not reachable. Allow outbound HTTPS to `registry-1.docker.io` and `production.cloudfront.docker.com`, or configure a registry mirror. |
| The model does not download | huggingface.co is not reachable. Allow outbound HTTPS for the server or set up a proxy. |
| Recognition is slow (RTF above 0.3 for GigaAM) | Too few cores or a CPU without AVX2. Add cores to the virtual machine and check the EVC mode. |
| The container is `unhealthy` | The worker loop has not reported for 10 minutes. Check `docker compose logs --tail 200`. |

{% hint style="info" %}
For good phrase splitting, turn on **Voice activity detection (VAD)** in the module settings: without it, the recording is cut into fixed-length windows instead of at pauses in speech.
{% endhint %}

### Data and security

| Data | Where it is stored |
| --- | --- |
| Models | volume `local-stt-worker_models` (`/models`) |
| Settings and key | volume `local-stt-worker_data`, file `/data/config.json` with mode 600 |
| Worker UID | volume `local-stt-worker_data`, file `/data/worker_uid` |
| Temporary audio | `/tmp` inside the container, in memory, deleted after the job |
| Logs | Docker log, up to 5 files of 20 MB |

The worker connects to MikoPBX only with outbound HTTPS requests, receives only the recordings of jobs assigned to it, and does not keep them after processing. The worker key grants access only to the transcription module API.
