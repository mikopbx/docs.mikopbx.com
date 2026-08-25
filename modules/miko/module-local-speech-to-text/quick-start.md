---
description: Quick start for the Local Speech To Text module and Local STT Worker
---

# Quick Start

### Installing the module

1. Open the MikoPBX web interface.
2. Go to **Modules** → **Module marketplace**.

<figure><img src="../../../.gitbook/assets/MikoPBXModuleMarketplace.png" alt=""><figcaption><p>Module marketplace</p></figcaption></figure>

3. Find **Local Speech To Text** and install it.
4. Open the **Installed modules** tab and enable the module.

<figure><img src="../../../.gitbook/assets/STTModuleInstalledModulesSection.png" alt=""><figcaption><p>Enabling Local Speech To Text</p></figcaption></figure>

5. Click the settings button to the right of the module version.

<figure><img src="../../../.gitbook/assets/STTModuleOpen.png" alt=""><figcaption><p>Opening the module page</p></figcaption></figure>

### Initial module setup

1. On the **Settings** tab, select the main language used in calls or leave **Detect automatically** selected.
2. Check **Maximum recognizable recording duration, min.** The default is 60 minutes.
3. Select the **Recording processing window** and **Transcript retention**. The processing window cannot exceed the retention period.
4. If necessary, add internal names and abbreviations under **Recognition terms**.
5. Save the settings.

<figure><img src="../../../.gitbook/assets/STTModuleMain.png" alt=""><figcaption><p>Module settings</p></figcaption></figure>

### Selecting a model

1. Open the **Model marketplace** tab.
2. Select the model for call transcription.
3. Click **Save model**.

{% hint style="info" %}
The model is downloaded to the Mac when the worker receives its first job. The first download requires access to Hugging Face.
{% endhint %}

<figure><img src="../../../.gitbook/assets/STTModuleChoosingModel.png" alt=""><figcaption><p>Selecting a recognition model</p></figcaption></figure>

### Creating a Worker API key

1. Open the **Workers** tab.
2. Click **Create API key**.
3. Copy the displayed token immediately. It cannot be viewed again after the page is refreshed.

{% hint style="info" %}
Create a separate key for each Mac. One key cannot be bound to multiple worker UIDs.
{% endhint %}

<figure><img src="../../../.gitbook/assets/STTModuleCreatingANewWorkerKey.png" alt=""><figcaption><p>Creating a Worker API key</p></figcaption></figure>

### Connecting Local STT Worker

Install and open **Local STT Worker**. A three-step onboarding wizard appears on first launch.

#### Step 1. Language

Select English or Russian as the interface language, then click **Next**. Changing the language may require a restart; after restarting, the onboarding wizard opens again.

<figure><img src="../../../.gitbook/assets/UPDSTTOnboardLang.png" alt=""><figcaption><p>Selecting the Local STT Worker language</p></figcaption></figure>

#### Step 2. Connect to MikoPBX

Complete these fields:

| Field                | What to enter                                                            |
| -------------------- | ------------------------------------------------------------------------ |
| **MikoPBX address**  | An address such as `https://pbx.example.com`, without credentials in the URL. |
| **Worker name**      | A recognizable Mac name that will be displayed in MikoPBX.               |
| **STT worker token** | The key created on the **Workers** tab.                                   |

The token is stored in macOS Keychain and is never written to logs.

<figure><img src="../../../.gitbook/assets/UPDSTTOndoardConnection.png" alt=""><figcaption><p>Configuring the MikoPBX connection</p></figcaption></figure>

Expand **Advanced settings** to configure:

* TLS certificate verification;
* a custom CA PEM file;
* a custom worker UID.

<figure><img src="../../../.gitbook/assets/UPDSTTOnboardingAdditionalSettings.png" alt=""><figcaption><p>Advanced MikoPBX connection settings</p></figcaption></figure>

Click **Connect and continue**. The application checks the connection and validates the supplied settings.

#### Step 3. Ready to use

The final step displays the MikoPBX and Speech to Text status and lets you enable two options:

* **Launch at login** — open Local STT Worker automatically when you sign in to macOS.
* **Keep worker running** — resume the worker automatically after the network connection is restored or the Mac wakes from sleep.

<figure><img src="../../../.gitbook/assets/UPDSTTOnboardingFinalPage.png" alt=""><figcaption><p>Final Local STT Worker setup step</p></figcaption></figure>

Click **Open Local STT Worker**. On the **Overview** page, click **Start Worker** if the worker is stopped.

### Verifying operation

1. Make a call for which call recording is enabled.
2. Wait for a job to appear on the module's **Queue** tab.
3. On first use, wait for the model to download to the Mac.
4. Check local processing stages under **Overview**, or review events under **Diagnostics**.
5. Open the completed result on the **Transcripts** tab in MikoPBX.

<figure><img src="../../../.gitbook/assets/STTWorkerTranscribationProcess.png" alt=""><figcaption><p>Call processing on the worker Overview page</p></figcaption></figure>

See [Local STT Worker](miko-ai-worker.md) for a detailed description of the application sections.
