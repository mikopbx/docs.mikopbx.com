---
description: Module description
---

# Cloud Speech-to-Text

The **Cloud Speech-to-Text** module automatically transcribes recorded MikoPBX calls and saves the completed transcripts on the PBX.

{% hint style="info" %}
The module is fully autonomous: no separate worker application is required. If audio recordings cannot be sent to an external service, use the [Local Speech To Text module](../module-local-speech-to-text/).
{% endhint %}

{% hint style="warning" %}
Submitting a recording for recognition is a paid action: the cost is deducted from your MIKO license balance. You can check prices and top up your balance at [lm.miko.ru](https://lm.miko.ru).
{% endhint %}

SCREENSHOT: Example of a call transcript created by the Cloud Speech-to-Text module.

### Requirements and compatibility

* An active MIKO license with the cloud transcription option and a positive balance.
* Call recording enabled for the required routes, queues, or employees.
* Direct outbound HTTPS access to `speech.mikolab.ru:443`.

### Installing the module

1. Open the MikoPBX web interface. Go to **Modules** → **Module Marketplace**.

SCREENSHOT: The Module Marketplace section in MikoPBX.

2. Find the **Cloud Speech-to-Text** module and install it.

SCREENSHOT: The Cloud Speech-to-Text module in the Marketplace.

3. Open the list of installed modules and enable the module. Click the settings button to the right of the module version.

SCREENSHOT: Enabling the module and opening its page from the installed modules list.

### First launch

The setup wizard opens when you launch the module for the first time. On the first page, the module warns that selected call recordings are sent to `speech.mikolab.ru` for recognition. Confirm your consent by clicking **Continue**.

SCREENSHOT: The Privacy step of the first-launch setup wizard.

At the second onboarding step, click **Check connection**: the module checks the service API availability, license authorization, and balance status.

SCREENSHOT: The Connection step of the first-launch setup wizard.

At the next step, review the call selection rules (directions, employees, and limits). Confirm that automatic selection should be enabled and click **Activate module**. The first submission will occur only after an eligible recording appears.

SCREENSHOT: Call selection rules in the Rules and activation step of the first-launch setup wizard.

SCREENSHOT: Confirmation of automatic call selection and module activation in the Rules and activation step.

### Overview tab

This tab opens by default and shows the module status:

* **Processing**, **Transcripts**, **Incidents**, and **Balance** cards with a refresh button;
* the **Successfully transcribed in the last 7 days** chart, showing minutes of recognized audio by day;
* the **Recent jobs** list with statuses and a link to the queue.

SCREENSHOT: The module's main screen, showing the Overview tab.

### Transcripts tab

This tab contains recognition results stored locally. You can filter the list by call period using the calendar or **Today**, and search by number, employee, or transcript text. The table shows the date, direction, counterparty, employee, status (complete or partial), duration, and update time.

SCREENSHOT: The Transcripts tab in the Cloud Speech-to-Text module.

Click a transcript to open its card. It contains:

* conversation turns with timestamps and participant roles: employee/client;
* a built-in player with playback speed controls and recording download;
* search;
* export to **JSON** or **TXT**;
* transcript deletion.

Clicking a turn moves the player to the corresponding point in the recording.

SCREENSHOT: A transcript card with conversation turns and the recording player.

### Queue tab

This tab manages the local job queue. It provides a status filter, a period filter (1 day or all time), search by number or direction, and pagination.

| Filter group | Job states |
| ------------ | ---------- |
| **Waiting** | Discovered, Waiting for recording, Ready to submit, Waiting for safe submission |
| **Processing** | Submitting, Accepted by service, Recognizing |
| **Needs attention** | Submission outcome unknown, Local failure, Service failure, Invalid result, Processing stalled, Result expired |
| **Completed** | Completed, Result discarded after deletion, Cancelled |

Technical information is available for each job: job ID, run number (generation), processing stage, error code, HTTP status and network error type, CDR ID, audio size, and audio format.

| Action | Availability and behavior |
| ------ | ------------------------- |
| **Cancel job** | Available only before audio upload begins. No confirmation is required. |
| **Retry job** | Requires explicit confirmation of a paid action. Creates a new job generation and may cause another charge. The module does not retry jobs automatically. |
| **Open transcript** | Opens the completed transcript on the Transcripts tab. |

SCREENSHOT: The Queue tab in the Cloud Speech-to-Text module.

### Settings tab

This tab configures call recognition settings: which calls to recognize and which to skip. The parameters in each settings section are summarized below.

#### **Directions and participants**

<table><thead><tr><th width="248.546875">Setting</th><th>Purpose</th></tr></thead><tbody><tr><td><strong>Process incoming calls</strong></td><td>Whether to recognize calls received by internal extensions.</td></tr><tr><td><strong>Process outgoing calls</strong></td><td>Whether to recognize calls initiated by employees to external numbers.</td></tr><tr><td><strong>Process internal calls</strong></td><td>Whether to recognize conversations between employees.</td></tr><tr><td><strong>Employees</strong></td><td>Which employees' calls to recognize or exclude from recognition.</td></tr></tbody></table>

#### **Number rules**

Rules include or exclude numbers from processing. Each rule specifies the direction (either direction, incoming, or outgoing), match type (exact number or prefix), number, and action (include or exclude).

SCREENSHOT: The Number rules settings section.

#### **Processing limits**

This section also defines which calls to process and which to skip. It sets the daily limit and the number of concurrent jobs. The table below describes these settings in more detail.

<table><thead><tr><th width="312.39453125">Setting</th><th>Purpose</th></tr></thead><tbody><tr><td><strong>Minimum duration</strong></td><td>Calls shorter than this value are not submitted.</td></tr><tr><td><strong>Maximum duration</strong></td><td>Calls longer than this value are not submitted.</td></tr><tr><td><strong>Daily limit</strong></td><td>How many minutes of audio can be submitted for recognition per day.</td></tr><tr><td><strong>Concurrent cloud jobs</strong></td><td>How many recordings the service recognizes simultaneously; the remaining recordings wait in the local queue.</td></tr></tbody></table>

{% hint style="info" %}
When the daily limit of minutes is exhausted, new jobs wait until the next day. If the administrator increases the limit, submissions resume immediately.
{% endhint %}

#### **Mode and retention**

This section sets the recognition language and mode, as well as the retention period for completed transcripts.

<table><thead><tr><th width="174.875">Setting</th><th>Purpose</th></tr></thead><tbody><tr><td><strong>Recognition mode</strong></td><td>Deferred (deferred-general) costs less, but processing a recording can take up to approximately 12 hours. Standard (general) is faster but costs more.</td></tr><tr><td><strong>Recognition language</strong></td><td>The language used to recognize the recordings.</td></tr><tr><td><strong>Transcript retention</strong></td><td>Limited (1–3650 days) or unlimited.</td></tr></tbody></table>

{% hint style="info" %}
After the retention period expires, transcripts are deleted during scheduled daily cleanup. MikoPBX CDR records and audio recordings are not affected.
{% endhint %}

### Pausing processing

The **Pause recording processing** button in the module header temporarily stops the selection and submission of new recordings, for example during maintenance or while investigating an incident. Jobs already accepted by the service continue to be polled until completion. The **Resume recording processing** button returns the module to normal operation.

SCREENSHOT: The button for pausing recording processing in the module header.

### Transcript in call history

In the MikoPBX **Call history** section, a call with a completed transcript provides a **Call transcript** dialog containing the conversation text and an **Open transcript** button to open it in the module.

SCREENSHOT: The button for opening a call transcript from the MikoPBX call history.

SCREENSHOT: A transcript opened from the MikoPBX call history.
