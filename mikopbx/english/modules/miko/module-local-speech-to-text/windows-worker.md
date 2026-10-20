---
description: >-
  Windows worker for local transcription: installing the service from the MSI
  or ZIP, connecting it to MikoPBX, checking that it works, updating and
  removing it.
---

# Windows worker

The **Windows worker** is a Windows service that recognizes call recordings locally on your computer or server. It takes jobs from the queue of the **Local Transcription** module, downloads the recording, recognizes it with the model selected in MikoPBX, and sends timestamped segments and technical diagnostics back to the PBX. Recordings never leave your network.

The worker has no graphical interface: it runs as the **MikoPBX Local STT Worker** service and is configured and controlled with `LocalSTTWorker.exe` from an administrator command prompt.

The Windows worker runs at the same time as [Local STT Worker for macOS](miko-ai-worker.md) and the [Docker worker](docker-worker.md): every platform has its own model selected on the **Model marketplace** tab, and every worker receives jobs with the model of its platform.

### Requirements

| Component  | Minimum                                                                                         |
| ---------- | ----------------------------------------------------------------------------------------------- |
| OS         | Windows 10/11 or Windows Server 2019 or newer, 64-bit (x64)                                     |
| Memory     | 8 GB                                                                                            |
| Disk       | about 0.5 GB for the program plus space for the selected model (up to 2.5 GB)                   |
| Components | [Visual C++ 2015-2022 x64 runtime](https://aka.ms/vs/17/release/vc_redist.x64.exe)              |
| MikoPBX    | 2025.1.1 or newer, with a **Local Transcription** module that offers the **Windows** platform in the model marketplace |
| Network    | outbound HTTPS to MikoPBX and huggingface.co (for the first model download)                     |

Recognition runs on the CPU; no graphics card is needed.

{% hint style="warning" %}
The MSI installer does not run without the Visual C++ runtime. Install `vc_redist.x64.exe` first.
{% endhint %}

### Models

The model is selected in MikoPBX: **Modules** → **Local Transcription** → **Model marketplace** tab, **Windows** platform. The worker downloads the model on the first job and keeps it locally. Every model is pinned to a specific version, and the worker checks the SHA-256 of every file.

| Model                                    | Languages                                     | When to choose                                       | Size   |
| ---------------------------------------- | --------------------------------------------- | ---------------------------------------------------- | ------ |
| **GigaAM v3** (default)                  | Russian                                       | Russian calls: best accuracy, punctuation, numbers   | 0.9 GB |
| **Parakeet TDT 0.6B v3**                 | 25 European                                   | European languages, detected automatically           | 2.5 GB |
| **GigaAM Multilingual**                  | Kazakh, Kyrgyz, Uzbek, Russian, English       | Central Asian languages, no punctuation              | 0.9 GB |
| **Whisper Large V3 Turbo (whisper.cpp)** | about 99 languages                            | Any language and mixed calls; the slowest on the CPU | 1.6 GB |

{% hint style="info" %}
GigaAM v3 recognizes Russian speech only: keep the module's default language at automatic or Russian. Vocabulary terms from MikoPBX are applied only by the Whisper model for now.
{% endhint %}

### Step 1. Prepare MikoPBX

1. Open **Modules** → **Local Transcription** → **Model marketplace** tab.
2. Switch the platform to **Windows** and select a model, for example **GigaAM v3**.
3. Go to the **Workers** tab and select the **Windows** platform.
4. Click **Create API key** and copy the key: it is shown once and is not available after the page is reloaded.

### Step 2. Download the worker

The **Workers** tab has two download buttons for the **Windows** platform:

| Package | File                                          | When to choose                                                     |
| ------- | --------------------------------------------- | ------------------------------------------------------------------ |
| **MSI** | `MikoPBX-LocalSTTWorker-<version>-x64.msi`     | Recommended: installs and starts the service automatically         |
| **ZIP** | `MikoPBX-LocalSTTWorker-<version>-win-x64.zip` | Manual installation to a folder of your choice; the `install` command registers the service |

MikoPBX downloads the latest worker version from the MIKO release server with the PBX license. If the download is unavailable, check that the MikoPBX license is active. The MSI installer and the program are digitally signed.

### Step 3. Install

Run all commands in a **command prompt or PowerShell as administrator**.

{% tabs %}
{% tab title="MSI (recommended)" %}
Run the installer by double-clicking it or from the command prompt:

```powershell
msiexec /i MikoPBX-LocalSTTWorker-<version>-x64.msi /qn /l*v install.log
```

The installer copies the program to `C:\Program Files\MikoPBX\LocalSTTWorker`, registers the **MikoPBX Local STT Worker** service (`MikoPBXLocalSTTWorker`) and starts it. Until the worker is configured, the service keeps running and waits for its settings.

The installation is per-machine, so the MSI can also be deployed without user interaction, for example with Group Policy (deploy the Visual C++ runtime first).

{% hint style="warning" %}
Do not run `LocalSTTWorker.exe install` after the MSI: the service is already registered.
{% endhint %}
{% endtab %}

{% tab title="ZIP" %}
Extract the archive, for example to `C:\Program Files\MikoPBX\LocalSTTWorker`. Do not change the files in the `Tools` folder: the worker checks their checksums and does not start if they differ.

The service is registered after setup, in step 4.
{% endtab %}
{% endtabs %}

### Step 4. Connect to MikoPBX (setup)

Go to the program folder and run the setup wizard:

```powershell
cd "C:\Program Files\MikoPBX\LocalSTTWorker"
.\LocalSTTWorker.exe setup
```

The wizard asks for:

1. **MikoPBX address** - `https://<mikopbx-address>`, with a port if needed, without a path.
2. **Whether to accept a self-signed MikoPBX certificate** - no by default (see the TLS section).
3. **Worker UID** - `windows-<computer-name>` by default; a key is bound to one UID.
4. **Worker name** - how it is shown on the **Workers** tab, `Windows STT Worker (<computer-name>)` by default.
5. **Worker key** - the key from step 1; the input is hidden.

The wizard then checks the connection and registers the worker. A successful result looks like this:

```
Settings saved. Checking the connection...
[ OK ] Bundled tools verified (ffmpeg, whisper.cpp, ONNX assets).
[ OK ] Settings: https://pbx.example.com as 'windows-office-pc'.
[ OK ] Worker API key is stored.
[ OK ] Module <version>, worker API v2, Windows platform supported.
[ OK ] Worker key accepted; worker registered.
[ OK ] Selected model 'GigaAM v3' is supported; it is downloaded on the first job.

The running service picks up the new settings within 30 seconds.
```

{% tabs %}
{% tab title="MSI" %}
The service is already running and picks up the settings within 30 seconds; no restart is needed.
{% endtab %}

{% tab title="ZIP" %}
Register and start the service:

```powershell
.\LocalSTTWorker.exe install
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
To change the address, key or name, run `setup` again. Press Enter to keep the current value; an empty key keeps the stored one. The running service picks up the changes within 30 seconds.
{% endhint %}

<details>

<summary>Setup without typing the key (for scripts)</summary>

With the `--token-stdin` flag the wizard reads the key from standard input instead of the keyboard. The key is never passed on the command line.

```powershell
Get-Content .\worker.key | .\LocalSTTWorker.exe setup --token-stdin
Remove-Item .\worker.key     # the key is already stored in the protected store
```

In this mode the wizard reads the other answers from standard input as well, one line per answer.

</details>

### Step 5. Check that it works

```powershell
.\LocalSTTWorker.exe status
```

On the first job the worker downloads the model (GigaAM v3 is about 0.9 GB) and checks its checksums, then starts recognizing.

1. In MikoPBX, the **Workers** tab shows the worker as **Online**, with the **Windows** platform, its IP and model.
2. `status` shows the service and worker state and a growing job counter:

```
Service      : RUNNING
Worker state : polling
Updated      : 2026-10-20 12:30:05 (12 s ago)
Worker UID   : windows-office-pc
Version      : 0.1.22
Model        : GigaAM v3
Jobs         : 50 done, 0 failed, 0 released, 120.5 min of audio
Last job     : #399, 70.4 s audio in 6.8 s (RTF 0.097)
Logs         : C:\ProgramData\MikoPBX\LocalSTTWorker\Logs
```

3. The call card shows the transcript, and its technical information shows the engine and the processing time.

**RTF** is the ratio of processing time to recording length. For example, an RTF of 0.1 means a 10-minute call is recognized in one minute.

| State (`Worker state`)                           | Meaning                                                                    |
| ------------------------------------------------ | -------------------------------------------------------------------------- |
| `not_configured`                                 | The worker is not configured: run `setup`.                                 |
| `negotiating`, `registering`, `loading_settings` | Connecting to MikoPBX and loading the settings.                            |
| `polling`                                        | The worker is waiting for jobs in the queue.                               |
| `processing`                                     | A job is running.                                                          |
| `waiting_for_windows_model`                      | MikoPBX does not serve a Windows model: update the module (see Troubleshooting). |
| `unsupported_model`                              | The selected model is not supported by this worker version.               |
| `stopped`, `failed`                              | The service is stopped or ended with an error; see `Last error` and the log. |

### Management

| Task                          | Command                                                |
| ----------------------------- | ------------------------------------------------------ |
| State                         | `.\LocalSTTWorker.exe status`                          |
| Check the connection and key  | `.\LocalSTTWorker.exe check`                           |
| Stop the service              | `.\LocalSTTWorker.exe stop`                            |
| Start the service             | `.\LocalSTTWorker.exe start`                           |
| Change the settings           | `.\LocalSTTWorker.exe setup`                           |
| Run in the console            | `.\LocalSTTWorker.exe run` (with the service stopped)  |

Run the commands in the program folder as administrator. The service can also be controlled in the **Services** snap-in (`services.msc`). It starts automatically (delayed start) and restarts after a failure: after one minute, then after five minutes for repeated failures. When it stops, the current job finishes or returns to the MikoPBX queue; new calls wait in the queue.

### Updating

{% tabs %}
{% tab title="MSI" %}
Download the new version on the **Workers** tab and run the installer the same way as for the installation. The old version is removed and the new one installed and started. Settings, the key, models and logs are kept.

Installing an older version over a newer one is refused: remove the worker first.
{% endtab %}

{% tab title="ZIP" %}
```powershell
.\LocalSTTWorker.exe stop
```

Replace the files in the program folder with the contents of the new archive and start the service:

```powershell
.\LocalSTTWorker.exe start
```
{% endtab %}
{% endtabs %}

### Removing

{% tabs %}
{% tab title="MSI" %}
Remove **MikoPBX Local STT Worker** in **Settings** → **Apps**, or from an administrator command prompt:

```powershell
msiexec /x MikoPBX-LocalSTTWorker-<version>-x64.msi /qn /l*v uninstall.log
```

Do not remove an MSI-installed service with `LocalSTTWorker.exe uninstall`.
{% endtab %}

{% tab title="ZIP" %}
```powershell
.\LocalSTTWorker.exe uninstall
```

Then delete the program folder.
{% endtab %}
{% endtabs %}

The `C:\ProgramData\MikoPBX\LocalSTTWorker` folder with the settings, key, models and logs is kept on purpose, so a reinstall does not download the model again. Delete this folder to remove everything. Then delete this worker's API key on the **Workers** tab in MikoPBX.

### TLS

Certificate verification is on by default: the worker connects only to a MikoPBX with a trusted certificate. If the PBX has a self-signed certificate or you connect to it by IP address, answer **y** to the self-signed certificate question in `setup`. The traffic is still encrypted. Do this only on a trusted network.

To make the worker trust your own certificate authority, install its root certificate into the computer's **Trusted Root Certification Authorities** store.

### Troubleshooting

To check the worker, run `.\LocalSTTWorker.exe check`: it checks the program files, settings, key, module compatibility, registration and the selected model.

| Symptom                                                                     | Cause and solution                                                                                                                        |
| --------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| `'setup' needs administrator rights`                                        | The command prompt runs without administrator rights. Open it with **Run as administrator**.                                             |
| The MSI installer reports that Visual C++ is missing                        | Install the [Visual C++ 2015-2022 x64 runtime](https://aka.ms/vs/17/release/vc_redist.x64.exe) and run the installer again.               |
| `status`: `not_configured`                                                  | The worker is not configured. Run `setup`.                                                                                               |
| `PBX rejected the worker API token`                                         | The key was deleted, mistyped or is already bound to another worker. Create a new key on the **Workers** tab and run `setup`.            |
| `update ModuleLocalSpeechToText` or `status`: `waiting_for_windows_model`   | The module version does not support the Windows platform. Update **Local Transcription**.                                                |
| `status`: `unsupported_model`                                               | The selected model is not supported by this worker version. Update the worker or select another model.                                   |
| TLS error when connecting                                                   | The MikoPBX certificate is not trusted. Allow a self-signed certificate in `setup` or install your CA (see the TLS section).               |
| The model is not downloaded                                                 | No access to huggingface.co. Allow outbound HTTPS for the computer or the proxy.                                                         |
| `The status is stale: the service is not running or is hung`                | The service has not updated its state for more than 10 minutes. Check the service in `services.msc` and the worker log.                  |

Worker warnings and errors are also written to the Windows **Application** log with the **MikoPBX Local STT Worker** source (Event Viewer, `eventvwr.msc`).

### Data and security

All data is kept in `C:\ProgramData\MikoPBX\LocalSTTWorker`. Only SYSTEM and administrators can access the folder.

| Data            | Where it is kept                                                     |
| --------------- | -------------------------------------------------------------------- |
| Program         | `C:\Program Files\MikoPBX\LocalSTTWorker`                            |
| Settings        | `settings.json`: MikoPBX address, worker UID and name                |
| Key             | `secrets\worker-token.bin`, encrypted with machine-scope DPAPI       |
| Models          | `Models\`, one folder per model and version                          |
| Logs            | `Logs\worker-yyyyMMdd.log`, kept for 14 days                         |
| State           | `status.json`, rewritten at least once a minute                      |
| Temporary audio | `Temporary\`, deleted after the job                                  |

The worker connects to MikoPBX only with outbound HTTPS requests, receives recordings only for jobs assigned to it and does not keep them after processing. The worker key grants access only to the transcription module API and is never passed on the command line.
