---
description: >-
  Docker worker for AI Supervisor: installing it on a Linux server from
  scratch, NVIDIA GPU, connecting it to MikoPBX, checking that it works,
  updating and removing it.
---

# Docker worker (Linux)

The **Docker worker** is a container that analyzes calls locally with AI on your Linux server. It takes jobs from the queue of the **Local AI Supervisor** module, analyzes the transcript with a local language model, detects emotions in the call recording and sends the result back to the PBX. Transcripts and recordings never leave your network.

The Docker worker is an alternative to the [AI Supervisor Worker application for macOS](miko-ai-worker.md). Both share the same analysis logic: prompts, result schema, linking of findings to transcript lines and quality criteria are identical, so the result does not depend on which worker took the job.

The image is published on Docker Hub: [`excla1m/mikopbx-ai-supervisor-worker`](https://hub.docker.com/r/excla1m/mikopbx-ai-supervisor-worker). The `latest` tag always points to the latest stable version.

### How it works

The worker runs as two containers from one `compose.yaml` file:

| Container              | Image                                  | What it does                                                                                                         |
| ---------------------- | -------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| `ai-supervisor-worker` | `excla1m/mikopbx-ai-supervisor-worker` | Connection to MikoPBX, job queue, prompts, validation of the model output, voice emotion analysis (SenseVoice, on the CPU) |
| `ai-supervisor-ollama` | `ollama/ollama`                        | The local language model. If the server has an NVIDIA GPU, the model runs on it                                       |

The worker does not pick calls and does not keep a server-side queue: the module in MikoPBX decides which calls are analyzed, with which model and which components.

### Requirements

| Component | Minimum                                                                     | Recommended                                                 |
| --------- | --------------------------------------------------------------------------- | ----------------------------------------------------------- |
| OS        | 64-bit Linux (x86\_64 or arm64)                                             | Ubuntu Server 24.04 LTS or Debian 12                        |
| CPU       | 4 cores with AVX2                                                           | 6-8 cores                                                   |
| Memory    | 8 GB                                                                        | 16 GB                                                       |
| GPU       | optional                                                                    | NVIDIA with 8 GB of video memory or more, Turing or newer   |
| Disk      | 40 GB                                                                       | 60 GB                                                       |
| Docker    | Docker Engine with the Compose plugin 2.24 or newer                         | latest stable version                                       |
| MikoPBX   | **Local AI Supervisor** module with worker API v2 support                   | latest module version                                       |
| Network   | outbound HTTPS to MikoPBX, Docker Hub, registry.ollama.ai and huggingface.co | same                                                        |

{% hint style="warning" %}
Without a GPU the model runs on the CPU. This is fine for a small call volume on a server with a modern CPU (AVX2, 8 cores or more). On old CPUs without AVX2 the analysis of one call may not fit into the time the module allows. For continuous work an NVIDIA GPU is recommended: a call is analyzed in tens of seconds.
{% endhint %}

### Models

The model is selected in MikoPBX: **Settings** → **AI analysis**. Every job comes with its own model profile, and the worker downloads the model into Ollama if it is not there yet.

| Profile in MikoPBX                  | Ollama model | Size   | Memory (GPU or RAM) | When to choose                                         |
| ----------------------------------- | ------------ | ------ | ------------------- | ------------------------------------------------------ |
| **Qwen3.5 4B Q4** (recommended)     | `qwen3.5:4b` | 3.3 GB | 4-6 GB              | Everyday call analysis, fast                           |
| **Qwen3.5 9B Q4** (higher quality)  | `qwen3.5:9b` | 6.6 GB | 8-12 GB             | Complex and long conversations; needs a 12 GB GPU      |

For voice emotion analysis the worker downloads the **SenseVoice Small** model (ONNX, about 240 MB) into the `models` volume on the first such job and verifies the checksum of every file.

{% hint style="info" %}
If the model does not fit into the video memory, Ollama moves some layers to the CPU and the analysis slows down. Check the placement with `docker compose exec ollama ollama ps` (see **Checking that it works**).
{% endhint %}

### Step 1. Preparing the server

Install Docker Engine and the Compose plugin:

```bash
curl -fsSL https://get.docker.com | sudo sh
sudo usermod -aG docker "$USER"      # to run docker without sudo
newgrp docker                        # or log in again
```

{% hint style="info" %}
If `get.docker.com` is not reachable from the server, install Docker from the [official Docker repository](https://docs.docker.com/engine/install/ubuntu/) or from the distribution packages: `sudo apt-get install -y docker.io docker-compose-v2`.
{% endhint %}

Check the installation and the CPU:

```bash
docker version
docker compose version
uname -m                                       # x86_64 or aarch64
grep -o -w avx2 /proc/cpuinfo | head -1        # must print avx2 on x86_64
```

Check access to MikoPBX and Docker Hub:

```bash
curl -skI https://<mikopbx-address> | head -1
curl -sI https://registry-1.docker.io/v2/ | head -1   # 401 is fine, the registry is reachable
```

### Step 2. NVIDIA GPU (optional)

Skip this step if there is no GPU: the model will run on the CPU.

1. Install the NVIDIA driver 550 or newer. On Ubuntu:

   ```bash
   sudo ubuntu-drivers install
   sudo reboot
   ```

2. Check that the driver sees the card:

   ```bash
   nvidia-smi
   ```

3. Install the NVIDIA Container Toolkit so that Docker can pass the GPU to containers:

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

4. Check the GPU inside a container:

   ```bash
   docker run --rm --gpus all ubuntu nvidia-smi
   ```

   The command must show your GPU, for example:

   ```
   NVIDIA GeForce GTX 1080, 8192 MiB
   ```

### Step 3. Preparing MikoPBX

1. Open **Modules** → **Local AI Supervisor** → **Settings** → **Workers**.
2. Click **Generate API key**.
3. Copy the key: it is shown only once and is no longer available after the page is reloaded.
4. In **Settings** → **AI analysis**, check the selected model (see **Models**).

If the module is being set up for the first time, the key can be created in the setup wizard on the **Local worker** step: click **Generate API key**; the macOS application does not need to be downloaded.

{% hint style="info" %}
Create a separate key for each worker: a key is bound to one worker.
{% endhint %}

### Step 4. The compose.yaml file

Create the worker directory and the `compose.yaml` file:

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
    # NVIDIA GPU (step 2). Without a GPU, remove the whole deploy block.
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

| Setting                                      | Purpose                                                                                                      |
| -------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| `name: ai-supervisor-worker`                 | Fixes the volume names (`ai-supervisor-worker_data` and others) regardless of the directory name.            |
| `restart: unless-stopped`                    | The worker and Ollama start together with Docker after a server reboot.                                      |
| `volumes`                                    | `data` - settings, UID and worker status; `models` - the SenseVoice model; `ollama` - language models.       |
| `read_only`, `cap_drop`, `no-new-privileges` | The worker cannot write anywhere except its volumes and `/tmp` and runs unprivileged as uid 10001.           |
| `stop_grace_period: 90s`                     | Time for the current job to finish or return to the queue when the worker stops.                             |
| `OLLAMA_NUM_PARALLEL: "1"`                   | The worker analyzes one call at a time, so Ollama does not split memory between requests.                    |
| `OLLAMA_KEEP_ALIVE: "30m"`                   | The model stays loaded between jobs instead of being read from disk for every call.                          |
| `deploy.resources...` (`ollama` service)     | Passes the NVIDIA GPU to the Ollama container. The worker itself does not need the GPU.                      |

The Ollama port is not published: only the worker talks to it over the internal Docker network.

### Step 5. Pulling the images

```bash
docker compose pull
```

Docker picks the server architecture (x86\_64 or arm64) itself. The worker image takes about 1 GB on disk, the Ollama image about 9 GB.

### Step 6. Connecting to MikoPBX (setup)

Run the setup wizard:

```bash
docker compose run --rm worker setup
```

The wizard asks for:

1. **MikoPBX address** - `https://<mikopbx-address>` without a path.
2. **Whether to verify the MikoPBX certificate** - no by default (see TLS).
3. **Worker key** - the key from step 3; input is hidden.
4. **Worker name** - how it is shown in the **Workers** section.
5. **Local model address** - keep `http://ollama:11434/v1`.
6. **Model to prepare** - the model the worker downloads before the first job. Use the model of the profile selected in MikoPBX: `qwen3.5:4b` or `qwen3.5:9b`.

The wizard then checks the connection and saves the settings to the `data` volume (`/data/config.json`, mode 600). Example:

```
PBX address (https://host[:port]): https://pbx.example.com
Verify the PBX certificate? (yes/no; no accepts a self-signed one) [no]:
Worker API key:
Worker name shown in MikoPBX [Docker AI worker]: GPU AI worker
Local AI runtime (OpenAI-compatible or Ollama URL) [http://ollama:11434/v1]:
Model to prepare before the first job [qwen3.5:4b]:

Checking the PBX…
  OK: https://pbx.example.com: ModuleAISupervisor 1.162.1, worker API v2; key accepted
Checking the local AI runtime…
  Reachable: Local runtime is reachable, but model "qwen3.5:4b" is not installed yet.

Saved. Worker UID ai-docker-e16a7a84fc7a4919. Start the worker with `docker compose up -d`.
```

The `Reachable ... not installed yet` line is expected on the first setup: the model is downloaded after the start.

<details>

<summary>Unattended setup (for scripts)</summary>

Save the key to a file readable by the container user (uid 10001) and pass the answers as flags:

```bash
install -m 600 /dev/null worker.key
nano worker.key                                   # paste the key and save
sudo chown 10001 worker.key

docker compose run --rm -v "$PWD/worker.key:/run/worker.key:ro" worker setup \
  --pbx-url https://<mikopbx-address> \
  --token-file /run/worker.key \
  --name "GPU AI worker" \
  --ai-model qwen3.5:4b \
  --yes

sudo rm -f worker.key                             # the key is already saved in the data volume
```

</details>

{% hint style="info" %}
To change the address, key, name or model, run `setup` again. The worker UID is kept in the `data` volume and does not change.
{% endhint %}

### Step 7. Starting

```bash
docker compose up -d
docker compose logs -f worker
```

On the first start the worker registers in MikoPBX and downloads the model into Ollama (`qwen3.5:4b` - 3.3 GB, a few minutes), then starts taking jobs:

```
INFO [worker] PBX https://pbx.example.com, TLS verification disabled (any PBX certificate is accepted)
INFO [worker] Local AI http://ollama:11434/v1, model qwen3.5:4b until the PBX assigns one
INFO [worker] AI Supervisor Worker 0.1.7 (linux) starting as GPU AI worker [ai-docker-e16a7a84fc7a4919]
INFO [ai-worker] Registered AI worker (server id 5), server poll interval 15.0s
INFO [ai-model] Local AI endpoint is reachable and model "qwen3.5:4b" is installed.
INFO [ai-worker] Claimed AI job #1566 call=mikopbx-1792739827.22 model=qwen3.5:4b
INFO [ai-worker] Submitted AI result for job #1566
```

### Checking that it works

1. In MikoPBX, the worker appears in **Settings** → **Workers** with the **Online** status and its IP address.
2. The `status` command shows the state and the job counter:

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

3. The `check` command checks the key, the model and the voice analysis runtime without running jobs:

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

4. With a GPU, check that the whole model is loaded into it:

   ```bash
   docker compose exec ollama ollama ps
   ```

   ```
   NAME          ID              SIZE      PROCESSOR    CONTEXT    UNTIL
   qwen3.5:4b    d8b0f5e9760c    3.3 GB    100% GPU     8192       29 minutes from now
   ```

   A value such as `13%/87% CPU/GPU` means the model did not fit into the video memory and part of the work runs on the CPU. Choose a smaller model profile or reduce the context size in MikoPBX.

5. The log shows call analysis jobs and voice emotion jobs. On the first voice job the worker downloads the SenseVoice model:

   ```
   INFO [ai-worker] Claimed AI job #1565 call=mikopbx-1792728833.20 model=qwen3.5:4b
   INFO [ai-worker] Submitted AI result for job #1565
   INFO [ai-worker] Claimed Voice Acoustic Emotion job #1567 call=mikopbx-1792739827.22 model=auto
   INFO [voice-analytics] Acoustic model SenseVoice Small (ONNX) is missing. Downloading it before acoustic patch.
   INFO [voice-model] SenseVoice Small (ONNX): ready
   INFO [voice-analytics] Submitted acoustic voice result for job #1567
   ```

6. The call card shows the summary, the scores and, if the component is enabled, the voice emotions.

On a GPU, a call is usually analyzed with `qwen3.5:4b` in 30-60 seconds, and voice emotions take 5-20 seconds.

### Management

| Task                                 | Command                                                              |
| ------------------------------------ | -------------------------------------------------------------------- |
| State                                | `docker compose exec worker ai-supervisor-worker status`             |
| Check connection, key and model      | `docker compose exec worker ai-supervisor-worker check`              |
| Worker log                           | `docker compose logs -f --tail 100 worker`                           |
| Loaded models                        | `docker compose exec ollama ollama ps`                               |
| Stop                                 | `docker compose stop`                                                |
| Start                                | `docker compose start`                                               |
| Restart                              | `docker compose restart`                                             |
| Change settings                      | `docker compose run --rm worker setup`, then `docker compose up -d`  |
| Version                              | `docker compose exec worker ai-supervisor-worker version`            |

Run all commands in `/opt/mikopbx-ai-supervisor-worker`. On stop, the current job finishes or returns to the MikoPBX queue; new calls wait in the queue.

### Updating

Pull the new image versions and recreate the containers. Models and settings are kept in volumes.

```bash
cd /opt/mikopbx-ai-supervisor-worker
docker compose pull
docker compose up -d
docker compose exec worker ai-supervisor-worker status
```

To pin a specific version, use a tag such as `excla1m/mikopbx-ai-supervisor-worker:<version>-cpu` in the `image` line, for example `0.1.7-cpu`. The list of versions is on the **Tags** tab on [Docker Hub](https://hub.docker.com/r/excla1m/mikopbx-ai-supervisor-worker/tags).

{% hint style="info" %}
The `cpu` suffix refers to the worker itself: it does not need a GPU. The GPU is used by the Ollama container, so the same image works on servers with NVIDIA.
{% endhint %}

### Removing

```bash
cd /opt/mikopbx-ai-supervisor-worker
docker compose down -v         # containers and volumes: models, settings, UID
docker image rm excla1m/mikopbx-ai-supervisor-worker:latest ollama/ollama:0.34.2
```

Then delete this worker's access key in MikoPBX in the **Workers** section. Without `-v` the volumes are kept, and the models do not need to be downloaded again after reinstalling.

### Configuration with .env

Instead of `setup`, settings can be set in an `.env` file next to `compose.yaml`. A non-empty environment variable always wins over the value saved by the wizard.

```bash
cat > .env <<'EOF'
PBX_URL=https://<mikopbx-address>
WORKER_TOKEN=<worker-key>
WORKER_NAME=GPU AI worker
AI_MODEL=qwen3.5:4b
EOF
chmod 600 .env
docker compose up -d
```

| Variable                             | Default                  | Purpose                                                                                       |
| ------------------------------------ | ------------------------ | --------------------------------------------------------------------------------------------- |
| `PBX_URL`                            | -                        | MikoPBX address: `https://host[:port]`, no path                                               |
| `WORKER_TOKEN` / `WORKER_TOKEN_FILE` | -                        | Worker key as a string or a path to a file inside the container                               |
| `WORKER_NAME`                        | `Docker AI worker`       | Name in the **Workers** section                                                               |
| `WORKER_UID`                         | generated automatically  | Stable identity; a key is bound to one UID                                                    |
| `PBX_TLS_VERIFY`                     | `false`                  | `true` rejects an untrusted MikoPBX certificate                                               |
| `PBX_CA_FILE`                        | -                        | Extra CA (PEM or DER) when TLS verification is on                                             |
| `AI_BASE_URL`                        | `http://ollama:11434/v1` | Local model address: Ollama or an OpenAI-compatible server                                    |
| `AI_MODEL`                           | `qwen3.5:4b`             | Model the worker downloads before the first job. The analysis model is selected in MikoPBX    |
| `AI_TIMEOUT`                         | `300`                    | Limit for one model request, seconds (the MikoPBX profile may override it)                    |
| `POLL_INTERVAL`                      | `15`                     | Pause between queue polls when idle, seconds                                                  |
| `HEARTBEAT_INTERVAL`                 | `60`                     | Job lease renewal interval, seconds                                                           |
| `API_TIMEOUT`                        | `30`                     | Limit for one MikoPBX request, seconds                                                        |
| `ACOUSTIC_TIMEOUT`                   | `60`                     | Limit for voice emotion analysis of one call, seconds                                         |
| `ONNX_THREADS`                       | `0`                      | CPU threads for voice analysis, `0` - all cores                                               |
| `LOG_LEVEL`                          | `INFO`                   | `DEBUG`, `INFO`, `WARNING`, `ERROR`                                                           |

{% hint style="info" %}
`AI_MODEL` is optional: if MikoPBX selects another model, the worker downloads it on the first job. The variable prepares the model in advance so the first job does not wait for a multi-gigabyte download.
{% endhint %}

### Model on another server

If the language model already runs on another server (Ollama, vLLM, LM Studio or another OpenAI-compatible server), the Ollama container is not needed:

1. Remove the `ollama` service, the `depends_on` line and the `ollama` volume from `compose.yaml`.
2. Set the server address in `.env`: `AI_BASE_URL=http://<server-address>:11434/v1`.
3. Restart the worker: `docker compose up -d`.

The model for each job is still selected in MikoPBX, so it must be available on that server.

### TLS

Certificate verification is off by default: the worker accepts any MikoPBX certificate (self-signed, issued for another name, a PBX reached by IP), and the traffic is still encrypted. This matches AI Supervisor Worker for macOS.

To reject untrusted certificates, turn verification on in `setup` or set `PBX_TLS_VERIFY=true`. The system CA store is used; for your own certificate authority, pass the file with `PBX_CA_FILE` and mount it into the container.

### Troubleshooting

| Symptom                                                                | Cause and fix                                                                                                                                      |
| ---------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| `status`: `update required (module or worker)`                         | The module and worker versions are not compatible. Update **Local AI Supervisor** or the worker image. The worker re-checks every 15 minutes.     |
| `status`: `the PBX rejected the worker key`                            | The key was deleted, mistyped or is already bound to another worker. Generate a new key in **Workers** and run `setup` again.                    |
| `status`: `PBX unreachable, waiting to retry`                          | No connection to MikoPBX. Check `curl -skI https://<mikopbx-address>` from the server. The worker retries with a growing interval up to 30 minutes. |
| `could not select device driver "nvidia"` on start                     | The NVIDIA Container Toolkit is not installed or Docker was not restarted after configuring it. Repeat step 2.                                   |
| `ollama ps` shows `100% CPU` although there is a GPU                   | The Ollama container did not get the GPU: check the `deploy` block in `compose.yaml` and `docker run --rm --gpus all ubuntu nvidia-smi`.         |
| `ollama ps` shows `CPU/GPU`, analysis is slow                          | The model does not fit into the video memory. Select the **Qwen3.5 4B Q4** profile or reduce the context size in MikoPBX.                         |
| Jobs end with a timeout                                                | The model is too slow, usually on a CPU without a GPU. Use a GPU or a server with a faster CPU.                                                    |
| The model is not downloaded                                            | No access to registry.ollama.ai. Allow outbound HTTPS for the server or proxy.                                                                    |
| The SenseVoice model is not downloaded                                 | No access to huggingface.co. Allow outbound HTTPS for the server or proxy.                                                                        |
| `docker pull` fails with `connection reset` or a timeout               | No access to Docker Hub. Allow outbound HTTPS to `registry-1.docker.io` and `production.cloudfront.docker.com` or configure a registry mirror.    |
| TLS error on connect                                                   | Certificate verification is on and the MikoPBX certificate is untrusted. Turn verification off or pass your CA.                                    |
| The container is `unhealthy`                                           | The worker loop has not reported for 10 minutes. Check `docker compose logs --tail 200 worker`.                                                    |

### Data and security

| Data                 | Where it is stored                                                        |
| -------------------- | ------------------------------------------------------------------------- |
| Language models      | `ai-supervisor-worker_ollama` volume                                      |
| SenseVoice model     | `ai-supervisor-worker_models` volume (`/models`)                          |
| Settings and key     | `ai-supervisor-worker_data` volume, `/data/config.json` with mode 600     |
| Worker UID           | `ai-supervisor-worker_data` volume, `/data/worker_uid`                    |
| Logs                 | Docker log and a file in `/data/logs`                                     |
| Temporary recordings | `ai-supervisor-worker_data` volume, deleted after the job                 |

The worker connects to MikoPBX with outbound HTTPS requests only, receives transcripts and recordings only for the jobs assigned to it and does not keep them after processing. The language model runs inside your network and does not send data to external services. The worker key grants access only to the AI Supervisor module API.
