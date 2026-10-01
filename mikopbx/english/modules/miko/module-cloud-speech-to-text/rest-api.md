---
description: Route descriptions
---

# REST API

The module lets you retrieve completed transcripts and perform other actions through its REST API. This can be useful for integrations with external systems. The most important REST API information is provided below.\
\
Base path:

```
/pbxcore/api/v3/module-cloud-speech-to-text
```

| Purpose | Endpoint |
| ------- | -------- |
| Summary and status | `GET /dashboard`, `GET /health` |
| Selection rules | `GET/PATCH /rule-sets/current` |
| Mode and retention | `GET/PATCH /settings` |
| Connection check | `POST /connection-checks` |
| Processing control | `GET/PATCH /processing-controls` |
| Job queue | `GET /jobs` |
| Job cancellation and retry | `POST /job-cancellations` |
| Transcript list | `GET /transcripts` |
| Transcript export | `GET /transcript-exports/{id}` |
| Module log | `GET /logs` |
| Diagnostic archives | `GET/POST /diagnostic-reports` |
| Transcript lookup by call | `GET /cdr-transcript-lookups` |

**Integrations: call-transcripts**

External modules read completed transcripts through two dedicated resources: `GET /call-transcripts` and `GET /call-transcript-events`. These resources accept only a dedicated integration key and return the complete immutable transcript contract: segments, merged turns, and plain text.

The identification scheme ensures that an external system does not repeat its analysis after a clean reinstallation of the module:

* `call_id` — a stable Asterisk call identifier (`linkedid`, or `UNIQUEID` when `linkedid` is empty);
* `source_instance_id` — the module installation identifier;
* `semantic_hash` — a hash of the transcript content using the `call-transcript-analysis-v1` profile: it matches only if the text, participants, and timings have not changed.

The `call-transcript.completed`, `call-transcript.updated`, and `call-transcript.deleted` events are delivered through cursor-based pagination and are idempotent. The public contract schema is `call-transcript.v1`. After a transcript is deleted, events no longer expose `call_id` or hashes.
