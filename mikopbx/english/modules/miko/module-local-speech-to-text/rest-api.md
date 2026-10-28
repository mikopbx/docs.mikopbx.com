# Module REST API reference

Base path:

```
/pbxcore/api/v3/module-local-speech-to-text
```

Requests are authorized with a Bearer token. Integration responses include `source_instance_id`, a permanent identifier of the module's data set, and `contract_version` (the current contract version is `3`).

#### Dialog and summary of a call

The simplest way to get the result for a specific call is to request it by the PBX call identifier. `{call_id}` can be the call's `linkedid` or the `UNIQUEID` of any of its recorded parts. The response is always built from the latest published transcript revision.

* `GET /call-dialogs/{call_id}` - the call dialog. Each turn in `turns` contains `start_ms`, `end_ms`, `channel`, the participant role `speaker` (`client` or `employee`), the employee name `speaker_name`, and `text`. The `text` field holds the whole dialog as plain text with speaker labels, and `direction` is the call direction (`incoming`, `outgoing`, `internal`).
* `GET /call-summaries/{call_id}` - the call summary: `topic`, `summary`, `model`, and the completion time `finished_at`. The `status` field takes these values:

| Status        | Meaning                                                                                  |
| ------------- | ---------------------------------------------------------------------------------------- |
| `done`        | The summary is ready.                                                                    |
| `pending`     | The summary is queued or in progress, including a new run after the call content changed. |
| `skipped`     | The conversation is too short to summarize.                                              |
| `error`       | The call could not be summarized.                                                        |
| `unavailable` | Summarization is off, or the call was transcribed before it was turned on.               |

#### Transcripts for integrations

* `GET /transcripts?limit=50&offset=0&date_from=YYYY-MM-DD&date_to=YYYY-MM-DD&search={text}`
* `GET /transcripts/{result_id}`
* `GET /call-transcripts/{call_transcript_id}?revision={revision}`
* `GET /call-transcripts/events?cursor=0:0&limit=100&include_deleted=false&include_summaries=false`

`transcripts` returns recognition results for individual recordings. The `limit` parameter accepts values from 1 to 200, and `search` is a query against the transcript text. A detailed transcript contains stable `segment_id` values, source segments, merged `turns`, and plain text.

`call-transcripts` combines multiple recordings from one logical call into a versioned transcript. Its part manifest preserves `cdr_start_ms` and `cdr_end_ms`, while segments contain relative and absolute timestamps. Adjacent segments from the same participant and channel are combined into a single turn without losing their source `segment_id` values. The response also contains the call language `language`, the PBX time zone `pbx_timezone`, the `is_current` flag (the requested revision is the current one), and the revision creation time `created_at`.

`call-transcripts/events` publishes idempotent `call-transcript.completed` and `call-transcript.updated` events. To read from the beginning, use the cursor `0:0` (the default); `limit` is 1 to 200, 100 by default. Store `next_cursor` only after all returned events have been processed successfully. Additional events are turned on with parameters:

* `include_deleted=true` - `call-transcript.deleted` events with the `deleted_at` and `deletion_reason` fields: `manual` for manual deletion, `retention` for an expired retention period;
* `include_summaries=true` - `call-summary.completed` events when a call summary is finished (`done`, `skipped`, or `error`). Then get the summary itself with `GET /call-summaries/{call_id}`.

#### Administration

| Operation                         | Endpoint                            |
| --------------------------------- | ----------------------------------- |
| List jobs                         | `GET /jobs`                         |
| Get a job                         | `GET /jobs/{job_id}`                |
| Create a job for a recording file | `POST /jobs`                        |
| Retry or reset a failed job       | `PATCH /jobs/{job_id}`              |
| Retry or reset several jobs       | `PATCH /jobs`                       |
| Delete a job                      | `DELETE /jobs/{job_id}`             |
| Bulk delete by status             | `DELETE /jobs`                      |
| List workers                      | `GET /workers`                      |
| Get a worker                      | `GET /workers/{worker_id}`          |
| Update a worker                   | `PUT`, `PATCH /workers/{worker_id}` |
| Delete a worker                   | `DELETE /workers/{worker_id}`       |

The `retry` action keeps the attempt counter, while `reset` clears it; only failed jobs can be changed. Only a pending or failed job without saved results can be deleted. A worker can be deleted only when it has no active lease; job and result history is preserved. Worker credentials do not grant access to job management.

#### Worker API v2

The worker first calls `GET /worker-api-contract`, then sends the `X-MikoPBX-Worker-API-Version: 2` header with every Worker API request. A request without a matching version is rejected with code `426` and the `worker_upgrade_required` error.

On registration, the worker sends its platform `worker_platform` (`macos`, `linux`, or `windows`; `macos` if omitted) and a list of optional features in `capabilities`, for example `call-summaries`. The same list can be sent in `worker_capabilities` when requesting a job.

The `GET /worker-processing-settings` response contains the processing profile, a `selected_model` object with the model selected for this worker's platform, and a `summarization` block with the summarization settings. Jobs also contain `model_engine` and `model_artifact_type`, which the worker uses to choose the recognition engine. Arbitrary engine, model, and artifact combinations are rejected.

| Operation             | Endpoint                          |
| --------------------- | --------------------------------- |
| Registration          | `POST /workers`                   |
| Processing profile    | `GET /worker-processing-settings` |
| Worker update license | `GET /worker-update-license`      |
| Acquire lease         | `POST /job-leases`                |
| Download recording    | `GET /job-recordings/{job_id}`    |
| Renew lease           | `PATCH /job-leases/{job_id}`      |
| Release lease         | `DELETE /job-leases/{job_id}`     |
| Submit result         | `PUT /job-results/{job_id}`       |
| Submit failure        | `PUT /job-failures/{job_id}`      |

Summary tasks are given to a worker only when there are no transcription jobs for it:

| Operation              | Endpoint                      |
| ---------------------- | ----------------------------- |
| Acquire a summary task | `POST /summary-leases`        |
| Renew                  | `PATCH /summary-leases/{id}`  |
| Release                | `DELETE /summary-leases/{id}` |
| Submit summary         | `PUT /summary-results/{id}`   |
| Submit summary failure | `PUT /summary-failures/{id}`  |
