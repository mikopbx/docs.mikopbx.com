# REST API

Base path:

```
/pbxcore/api/v3/module-local-speech-to-text
```

Requests are authorized with a Bearer token.

#### Transcripts for integrations

* `GET /transcripts?limit=50&offset=0&date_from=YYYY-MM-DD&date_to=YYYY-MM-DD&search={text}`
* `GET /transcripts/{result_id}`
* `GET /call-transcripts/{call_transcript_id}?revision={revision}`
* `GET /call-transcripts/events?cursor=created_at:event_id&limit=100&include_deleted=false`

`transcripts` returns recognition results for individual recordings. The `limit` parameter accepts values from 1 to 200, and `search` is a query against the transcript text. A detailed transcript contains stable `segment_id` values, source segments, merged `turns`, and plain text.

`call-transcripts` combines multiple recordings from one logical call into a versioned transcript. Its part manifest preserves `cdr_start_ms` and `cdr_end_ms`, while segments contain relative and absolute timestamps. Adjacent segments from the same participant and channel are combined into a single turn without losing their source `segment_id` values. The response includes `source_instance_id`, a permanent identifier of the module's data set, and `contract_version`.

`call-transcripts/events` publishes idempotent `call-transcript.completed` and `call-transcript.updated` events. To read from the beginning, use the cursor `0:0`; store `next_cursor` only after all returned events have been processed successfully. With `include_deleted=true`, the stream also returns `call-transcript.deleted` events with the `deleted_at` and `deletion_reason` fields: `manual` for manual deletion, `retention` for an expired retention period.

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

The `GET /worker-processing-settings` response contains the centralized processing profile and a `selected_model` object with the model identifier, repository, engine, Core ML artifact type, and display name. Jobs also contain `model_engine` and `model_artifact_type`, which the worker uses to choose Parakeet or WhisperKit. Arbitrary engine, model, and artifact combinations are rejected.

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
