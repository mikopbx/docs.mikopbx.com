---
description: How to fetch finished cloud speech-to-text transcripts over REST API
---

# REST API

The Cloud Speech-to-Text module delivers finished call transcripts in a readable form — to a CRM, an analytics system, or another module on the same PBX (this is exactly how the AI Supervisor module receives its transcripts).

Base path:

```
/pbxcore/api/v3/module-cloud-speech-to-text
```

## Integration key

Reading transcripts requires a dedicated key — regular keys with broad permissions are rejected by the module.

1. Open **System → API keys** and create a key **without** full permissions.
2. Set the key's network filter to “localhost”.
3. In the permissions, keep a single row — read access to `/api/v3/module-cloud-speech-to-text/call-transcripts`.

Such a key only accepts requests made from the PBX itself. If your external system runs on another server, open an SSH tunnel to the PBX (`ssh -L 8080:127.0.0.1:80 pbx-address`) and talk to `http://127.0.0.1:8080`.

{% hint style="info" %}
Responses are wrapped in a standard envelope: `result` (success flag), `data` (the payload) and `messages` (error text). The sections below describe the contents of `data`.
{% endhint %}

## List of transcripts

```bash
curl -H "Authorization: Bearer INTEGRATION_KEY" \
  "https://pbx-address/pbxcore/api/v3/module-cloud-speech-to-text/call-transcripts?limit=50&offset=0"
```

Parameters: `limit` (1 to 200, default 50), `offset`, plus `from` and `to` — a time window as Unix timestamps. Each item contains the transcript identifier `aggregate_id` (you need it to fetch the full text), `call_id`, `revision` and the call date. The `pagination` object shows the total count.

## Full transcript

```bash
curl -H "Authorization: Bearer INTEGRATION_KEY" \
  "https://pbx-address/pbxcore/api/v3/module-cloud-speech-to-text/call-transcripts/0192f0c1-a2b3-4c5d-8e9f-0a1b2c3d4e5f"
```

Pass an `aggregate_id` from the list or an event. The response contains:

* `transcript.segments` — every phrase with timings, channel and speaker;
* `transcript.turns` — phrases merged into turns;
* `transcript.text` — the full text in one piece;
* `call` — the call direction, the employees' extension numbers and the counterparty numbers;
* `recognition` — the language and the recognition model.

Add `?revision=N` to request a specific revision; by default the latest published one is returned.

## New-transcript events

```bash
curl -H "Authorization: Bearer INTEGRATION_KEY" \
  "https://pbx-address/pbxcore/api/v3/module-cloud-speech-to-text/call-transcripts:events?limit=100"
```

* Make the first request without the `cursor` parameter — it will return events from the very beginning. Pass the `next_cursor` from the response in the next request, and store it only after you have successfully processed all received events.
* `call-transcript.completed` and `call-transcript.updated` — a transcript is ready or was updated; after these, fetch the full text.
* `call-transcript.deleted` — a transcript was deleted; in these events `call_id` and the hashes are hidden.

## How to avoid processing the same call twice

* `semantic_hash` — a hash of the transcript's content. If it has not changed, the text, participants and timings are the same, and the call does not need to be analyzed again.
* `source_instance_id` — a permanent identifier of the module's data set. It survives updates and helps you tell that the data is still the same rather than recreated.
* Events are not repeated: it is enough to remember the event identifier together with `source_instance_id`.

{% hint style="info" %}
All other module requests (settings, jobs, diagnostics) serve its web interface and are not intended for integrations.
{% endhint %}
