---
description: How to fetch AI call analysis results and manage call workflows over REST API
---

# REST API

The Cloud AI Supervisor module analyzes calls and shows the results in its own interface. The same data can be pulled into an external system through the REST API — for example, to display the summary and quality score in a CRM next to the customer card, and to manage the follow-up work on the call there as well.

Base path:

```
/pbxcore/api/v3/module-cloud-ai-supervisor
```

## Access key

1. Open **System → API keys** and create a new key.
2. Grant it permissions on this module's rows: read — to fetch data, and write on the calls row — if you plan to change statuses and add notes.
3. Send the key with every request in the header:

```bash
Authorization: Bearer YOUR_KEY
```

{% hint style="info" %}
Every response is wrapped in a standard envelope: `result` (success flag), `data` (the payload) and `messages` (error text). The sections below describe the contents of `data`.
{% endhint %}

## List of analyzed calls

```bash
curl -H "Authorization: Bearer YOUR_KEY" \
  "https://pbx-address/pbxcore/api/v3/module-cloud-ai-supervisor/calls?period=7d&limit=50&offset=0"
```

Main parameters:

* `period` — a preset range: `1d`, `7d` or `30d`; narrow it further with `date_from` and `date_to`;
* `search` — search over the number, employee, topic and status; a specific call can be found with the `call_id` parameter;
* `employee_numbers` — comma-separated employee extension numbers;
* `direction` — `incoming`, `outgoing` or `internal`;
* `sentiment`, `quality`, `risk` — filters by tone and by assessment and risk levels (`high`, `medium`, `low`);
* `review_status` — `reviewed` or `unreviewed`; `workflow_status` — `open`, `in_progress` or `closed`;
* `limit` and `offset` — paginated output (50 records by default).

Each list item contains: the `id`, the call identifier `call_id`, the date, the duration, the numbers and names of both parties, and — most importantly — the `ai_result` block with the `summary`, the `sentiment`, the risk score `risk_score` (0–100) and the quality score `quality_score`.

## Call card

```bash
curl -H "Authorization: Bearer YOUR_KEY" \
  "https://pbx-address/pbxcore/api/v3/module-cloud-ai-supervisor/calls/12"
```

Take `id` from the list (or from the exact `call_id` lookup). Besides the list fields, the card contains the full transcript `stt_detail.transcript` — turns with participant roles (“customer”/“employee”) and timings — the call's workflow history `workflow_history`, and the quality assessment details.

## Call workflow: statuses and notes

```bash
curl -X PATCH -H "Authorization: Bearer YOUR_KEY" -H "Content-Type: application/json" \
  -d '{"workflow_status":"closed","close_reason":"Resolved","note":"Customer confirmed the fix"}' \
  "https://pbx-address/pbxcore/api/v3/module-cloud-ai-supervisor/calls/12"
```

Available fields:

* `reviewed` — mark the call as reviewed (`true`/`false`);
* `workflow_status` — the workflow status: `open`, `in_progress`, `closed`;
* `assignee_id`, `assignee_label` — the responsible person;
* `due_at` — the deadline (a Unix timestamp; `0` clears it);
* `is_escalated` — escalation flag;
* `close_reason` — the closing reason, `note` — a note (appended to the history).

The same fields can be applied to several calls at once: a `PATCH /calls` request with an `ids` array.

## Team overview

```bash
curl -H "Authorization: Bearer YOUR_KEY" \
  "https://pbx-address/pbxcore/api/v3/module-cloud-ai-supervisor/dashboard?period=30d"
```

Accepts the same periods as the list. The response contains the number of calls and analyzed conversations, breakdowns by day, hour and employee, the average risk, the sentiment split, contact topics, and the calls that need attention. A convenient source for your own dashboard.

## Spending and health

* `GET /usage` — how many calls were analyzed, token and money spending over a period (`today`, `7d`, …), and the remaining balance.
* `GET /health` — whether the module is running and its transcript source is available; handy for monitoring.

{% hint style="info" %}
The module's remaining requests (settings, the job queue, transcript import, quality scripts, diagnostics) serve its web interface. The module has no worker protocol of its own — the analysis runs in the cloud.
{% endhint %}
