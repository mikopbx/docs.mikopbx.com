---
description: >-
  Local STT Worker for macOS: connecting to MikoPBX, local transcription,
  history, models, diagnostics, settings, and Worker API updates.
---

# Local STT Worker

**Local STT Worker** is an application that locally recognizes MikoPBX call recordings. It downloads one assigned job, prepares the audio with the bundled `ffmpeg` and `ffprobe`, runs the Parakeet or WhisperKit engine selected in MikoPBX with a Core ML model, and sends timestamped segments and technical diagnostics back to the PBX.

<figure><img src="../../../.gitbook/assets/STTWorkerTranscribationProcess.png" alt=""><figcaption><p>Overview page in Local STT Worker</p></figcaption></figure>

### Requirements

* A Mac with Apple silicon.
* macOS 14.0 or later.
* MikoPBX 2025.1.1 or later.
* Network access to the PBX.
* Internet access for the first download of the selected model and its supporting files.

The worker uses WhisperKit or FluidAudio for recognition, depending on the model in the job. `ffmpeg` and `ffprobe` are included in the worker build and do not need to be installed separately.

### Engines and models

Models are selected only in MikoPBX. Each job provides the engine, model, repository, and artifact type identifiers. The worker validates their compatibility and starts the corresponding engine automatically.

* **Parakeet TDT 0.6B v3** runs through FluidAudio and is the default model. It is intended for long-form speech and supports 25 European languages, including Russian and Ukrainian.
* **Whisper Large V3 Turbo**, **Whisper Podlodka Turbo**, and **Whisper Large V3** run through WhisperKit.
* Arbitrary custom models are not supported. The worker accepts only models in the current MikoPBX catalog.

Recognition terms are passed to both engines. WhisperKit uses them as context, while Parakeet compiles them into a local decoder bias.

### First launch

{% hint style="info" %}
The onboarding process is described in detail in the [Quick Start guide](quick-start.md#step-1.-language).
{% endhint %}

The onboarding wizard has three steps:

1. **Choose your language** — select the English or Russian interface. After changing the language, the application may prompt you to restart and will continue setup when it opens again.
2. **Connect to MikoPBX** — enter the `https://...` address, worker name, and token from the **Workers** tab. Advanced settings contain TLS verification, a custom CA, and the worker UID.
3. **Ready to use** — verify MikoPBX and Speech to Text readiness, configure launch at login, and open the application.

### Navigation

| Section         | Purpose                                                                                  |
| --------------- | ---------------------------------------------------------------------------------------- |
| **Overview**    | Worker status, connection, current activity, system readiness, and recent calls.         |
| **History**     | Local history of jobs handled by this Mac, with filters and search.                      |
| **Models**      | Locally downloaded WhisperKit and Parakeet Core ML models and storage usage.             |
| **Diagnostics** | Connection check, UID, heartbeat, and local events.                                      |
| **Settings**    | MikoPBX connection and settings that apply only to this Mac.                             |

### Overview section

The top panel displays one of the main states:

| State                       | Meaning                                                               |
| --------------------------- | --------------------------------------------------------------------- |
| **Everything is working**   | The worker is connected and automatically polls the queue.            |
| **Transcribing a call**     | A job is currently being processed.                                   |
| **Attention required**      | A connection, compatibility, or processing error occurred.            |
| **Ready to start**          | The connection is configured, but the worker is stopped.              |
| **Setup required**          | The PBX address, token, or registration is missing.                    |

The primary button changes with the state: **Start Worker**, **Stop Worker**, or **Open Settings**. When the worker is stopped, new calls remain in the MikoPBX queue.

The summary shows the number of calls processed today, the percentage of successfully completed jobs, and the time of the latest call. Below it are the current activity, connection readiness, selected model, launch-at-login status, and three most recent jobs. The selected model name comes from the centralized MikoPBX settings; the worker interface has no separate local default-model selection.

During active processing, the current stage and progress are displayed:

1. contract check and registration;
2. job lease acquisition;
3. recording download;
4. audio analysis and preparation;
5. engine selection and model download or loading;
6. local transcription through Parakeet or WhisperKit;
7. result or failure submission;
8. temporary file cleanup.

Before submitting a result, the worker normalizes it and rejects empty, corrupted, and clearly repetitive fragments. Technical diagnostics include accepted and rejected segment counts and rejection reasons; the module then applies its additional server-side quality filter.

### History section

History contains only the jobs received by this Mac. It is not the complete PBX transcript list.

Available periods are **24 hours**, **7 days**, and **30 days**. You can filter completed, problematic, and active jobs, or search by `call_id`. Expand a row to view its stages and technical data. A problematic job includes a shortcut to **Diagnostics**.

Local history is retained between application launches. Completed transcripts and call recordings continue to be stored and managed by MikoPBX.

<figure><img src="../../../.gitbook/assets/STTNEWHistory.png" alt=""><figcaption><p>History in Local STT Worker</p></figcaption></figure>

### Models section

The model for new jobs is selected in MikoPBX on the **Model marketplace** tab. The worker synchronizes the selection with the PBX and automatically downloads the required files when it receives the first job that uses the model.

This section shows:

* the number of models and their total size;
* the size of service data;
* the list of locally downloaded models;
* the model storage location in Finder.

For Parakeet, this section may show two sets of files: **ASR**, which contains the Core ML recognition model, and **Tokenizer**, which is used to prepare recognition-term hints. Both sets belong to the same model.

The **Refresh** and **Open Folder** buttons rescan local storage and open it in Finder. An unused local model copy can be deleted only after the worker is stopped. If the model is needed again, it will be downloaded again.

{% hint style="info" %}
Processing with a new model takes longer the first time because the worker must download it and prepare its cache. Keep the internet connection available until the download completes.
{% endhint %}

<figure><img src="../../../.gitbook/assets/STTNEWModels.png" alt=""><figcaption><p>Models in Local STT Worker</p></figcaption></figure>

### Diagnostics section

Diagnostics combines connection status, automatic recovery information, and the local log.

At the top, you can see:

* the overall health assessment;
* the **Check Connection** button;
* the **Open Logs Folder** button.

The technical summary shows the MikoPBX address, connection status, UID, and time of the last successful heartbeat. Copy the UID to match this Mac with its entry in the PBX worker table.

Local events can be filtered by time (**All**, **1 hour**, **Today**) and level (**All**, **Warnings**, **Errors**), searched, copied, or exported. The page displays the latest 250 matching events.

<figure><img src="../../../.gitbook/assets/STTNEWDiagnostic.png" alt=""><figcaption><p>Diagnostics in Local STT Worker</p></figcaption></figure>

### Settings section

#### MikoPBX Connection

| Field               | Purpose                                                         |
| ------------------- | --------------------------------------------------------------- |
| **MikoPBX Address** | Address of the PBX used by the worker.                           |
| **Worker token**    | Restricted module token stored in macOS Keychain.               |
| **Worker name**     | Recognizable Mac name shown in the PBX worker table.             |
| **Worker UID**      | Stable technical identifier for this Mac.                       |

The **Register This Mac** button checks the connection and creates or updates the registration.

#### General settings

| Setting                 | Purpose                                                                                              |
| ----------------------- | ---------------------------------------------------------------------------------------------------- |
| **App language**        | English, Russian, or the system language. A restart is required to apply a change.                   |
| **Start with macOS**    | Start the application after the user signs in.                                                       |
| **Keep worker running** | Preserve the intent to run and recover automatically after a network outage, sleep, or temporary error. |
| **Temporary files**     | Folder containing downloaded recordings and intermediate WAV segments.                              |
| **TLS verification**    | Verify the MikoPBX HTTPS certificate.                                                                |
| **Custom CA**           | Optional PEM/CRT/CER file for a private certificate authority.                                      |

The **Run setup again** button opens the onboarding wizard again.

<figure><img src="../../../.gitbook/assets/STTNEWSettings.png" alt=""><figcaption><p>Settings in Local STT Worker</p></figcaption></figure>

### Menu bar status

The Local STT Worker menu bar icon shows whether Speech to Text is running, a transcription is in progress, or attention is required. The menu contains the status, current stage or last-contact time, and the **Open App** button.

<figure><img src="../../../.gitbook/assets/STTNEWStatusbar.png" alt="" width="330"><figcaption><p>Worker status in the menu bar</p></figcaption></figure>

### Local data and security

| Data            | Location                                                   |
| --------------- | ---------------------------------------------------------- |
| Models          | `~/Library/Application Support/Local STT Worker/Models`    |
| Temporary files | `~/Library/Application Support/Local STT Worker/Temporary` |
| Logs            | `~/Library/Logs/Local STT Worker`                          |
| Passwords/tokens | Keychain service `LocalSTTWorker.PBX`                      |
| Settings        | Application UserDefaults `com.mikopbx.LocalSTTWorker`      |

The worker accepts recordings only from the authorized MikoPBX and stores them in a secure temporary directory. Temporary audio files are deleted after each job completes.

For Parakeet, the worker verifies the expected model and repository, downloads components only over HTTPS, validates supporting files, and does not follow symbolic links inside model directories. Local model paths are not included in failure messages sent to the PBX.

### What is configured in MikoPBX and what is configured on the Mac

| Location                   | Settings                                                                                                                                                                              |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **MikoPBX / module**       | Processing window, retention period, maximum recording duration, engine and model, job language, recognition terms, processing profile, queue, keys, transcripts, and module log. |
| **Local STT Worker**       | Connection settings, Mac UID and name, worker startup and recovery, local model cache, temporary folder, TLS/CA, local history, and diagnostics.                                 |
