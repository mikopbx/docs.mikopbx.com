---
description: How to fetch call transcripts and summaries from the Local Transcription module over REST API
---

# REST API

The module delivers finished call transcripts to external systems — a CRM, a Telegram bot, corporate analytics. There is no need to touch the PBX database: everything is available through a few simple requests.

Base path:

```
/pbxcore/api/v3/module-local-speech-to-text
```

## Access key

1. In the PBX web interface, open **System → API keys** and create a new key.
2. In the key permissions, keep a single row — read access to `/api/v3/module-local-speech-to-text/call-transcripts`.
3. Send the key with every request in the header:

```bash
Authorization: Bearer YOUR_KEY
```

Requests made from the PBX itself (for example, by another installed module) do not require a key.

{% hint style="info" %}
Every response is wrapped in a standard envelope: `result` (success flag), `data` (the payload) and `messages` (error text). The sections below describe the contents of `data`.
{% endhint %}

## Call dialog

The most common scenario is fetching a finished dialog by the call identifier. Use the call's `linkedid` or the `UNIQUEID` of any of its recorded legs (visible in the call history and in the recording file name):

```bash
curl -H "Authorization: Bearer YOUR_KEY" \
  "https://pbx-address/pbxcore/api/v3/module-local-speech-to-text/call-transcripts/mikopbx-1791275140.999663_claea922:dialog"
```

The response always contains the latest published transcript revision:

* `turns` — dialog turns: `text`, `start_ms`/`end_ms` timings, the `channel`, and the speaker `speaker` (`client` or `employee`) with the employee's name `speaker_name`;
* `text` — the whole dialog as plain text, one turn per line, each starting with the speaker's name;
* `direction` — call direction: `incoming`, `outgoing` or `internal`;
* `language` — the detected conversation language.

If the call has not been transcribed yet, you get a `404` with the code `call_transcript_not_found`.

## Call summary

```bash
curl -H "Authorization: Bearer YOUR_KEY" \
  "https://pbx-address/pbxcore/api/v3/module-local-speech-to-text/call-transcripts/mikopbx-1791275140.999663_claea922:summary"
```

Returns a single summary for the whole call: `topic`, the `summary` text, the `model`, and the completion time `finished_at`. The `status` field tells you what to do with it:

| Status        | Meaning                                                                  |
| ------------- | ------------------------------------------------------------------------ |
| `done`        | The summary is ready.                                                    |
| `pending`     | Queued or still being generated; ask again later.                        |
| `skipped`     | The conversation is too short; there will be no summary.                 |
| `error`       | The summary could not be produced.                                       |
| `unavailable` | Summarization is off, or the call was transcribed before it was enabled. |

## List of transcribed calls

```bash
curl -H "Authorization: Bearer YOUR_KEY" \
  "https://pbx-address/pbxcore/api/v3/module-local-speech-to-text/call-transcripts?limit=50&offset=0&date_from=2026-10-01&date_to=2026-10-31&search=delivery"
```

The list is ordered from the newest calls to the oldest. Parameters:

* `limit` — how many records to return (1 to 200, default 50), `offset` — how many to skip;
* `date_from`, `date_to` — filter by call date;
* `search` — full-text search over the transcripts.

Each list item contains `call_id`, `aggregate_id`, `revision`, the call date `call_started_at` and the number of legs `part_count`; the `pagination` object holds the total record count.

## Full transcript

```bash
curl -H "Authorization: Bearer YOUR_KEY" \
  "https://pbx-address/pbxcore/api/v3/module-local-speech-to-text/call-transcripts/77"
```

Instead of the number, you can pass the same call identifier as for the dialog. The response contains:

* `transcript.segments` — every phrase with exact timings, both within the call and within the recording;
* `transcript.turns` — phrases merged into turns by channel and speaker;
* `transcript.text` — the full text;
* `part_manifest` — which recordings the call was assembled from;
* `revision` and `is_current` — the transcript revision and whether it is the current one (an older revision can be fetched with `?revision=N`).

## New-transcript events

To avoid polling the list, subscribe to the event feed:

```bash
curl -H "Authorization: Bearer YOUR_KEY" \
  "https://pbx-address/pbxcore/api/v3/module-local-speech-to-text/call-transcripts:events?limit=100"
```

* Make the first request without the `cursor` parameter — it will return events from the very beginning. Store the `next_cursor` from the response only after you have successfully processed all received events, and pass it in the next request.
* `call-transcript.completed` and `call-transcript.updated` — a call was transcribed or its transcript was updated; after these, request the dialog or the full transcript.
* `include_deleted=true` — also adds `call-transcript.deleted` events with the deletion reason: `manual` (deleted by an administrator) or `retention` (retention period expired).
* `include_summaries=true` — also adds `call-summary.completed` events telling you a call summary is ready.

## Identifiers

* `call_id` — the permanent PBX call identifier (`linkedid`); the `UNIQUEID` of a recorded leg also works for lookups.
* `aggregate_id` — the module's number of the call transcript.
* `source_instance_id` — a permanent identifier of this module's data set; store it in your system so that after a module reinstall you do not process the same calls again.
* `contract_version` — the response format version (currently `4`).

{% hint style="info" %}
The rest of the module's API (job and worker management) is for internal use: it serves the module's web interface and the worker applications. Only the requests described above are intended for integrations.
{% endhint %}
