---
description: Instructions with examples on creating and using API keys
---

# REST API Usage Examples

Working with the REST API follows the OpenAPI standard. To get the current list of endpoints, use the "**Documentation**" section inside the PBX. Below are examples of working with the main features of the MikoPBX REST interface.

{% hint style="info" %}
If you do not have a trusted certificate — add `verify=False` to each request and disable warnings:

```python
import urllib3
urllib3.disable_warnings()
```

It is strongly recommended to issue a trusted certificate. The easiest way to do this is by using the Let's Encrypt module.
{% endhint %}

### Connection

To run all examples in this guide, create an API key and configure the following access permissions (see the general article for details):

| Resource             | Access Level   | Used for                                 |
| -------------------- | -------------- | ---------------------------------------- |
| Employees Management | Read and write | Creating and editing employees           |
| Providers            | Read and write | Creating and editing providers           |
| SIP                  | Read           | Employee and trunk registration statuses |
| Call Records         | Read           | Call history (CDR)                       |
| PBX Status           | Read           | Active calls in real time                |
| SIP Providers        | Read and write | Creating and editing SIP providers       |

<figure><img src="../../../.gitbook/assets/APIKeyCallRecords.png" alt=""><figcaption><p>Example of access permission configuration (Call Records and Employees permissions)</p></figcaption></figure>

In this article we will be working with Python, so you need to install the required dependencies:

```bash
pip install requests
```

Below is a connection template for accessing the station via an API key. Use it before all scripts in this guide. The API key is passed directly in the request header — no additional authentication is required:

```python
import requests

BASE_URL = 'https://your-mikopbx.com/pbxcore/api/v3'
API_KEY  = 'your-api-key'

HEADERS = {
    'Authorization': f'Bearer {API_KEY}',
    'Content-Type':  'application/json',
}
```

{% hint style="info" %}
In the template, replace the following parameters:

* `your-mikopbx.com` — with the IP address or URL of your station.
* `your-api-key` — with the previously created API key with the required permissions.
{% endhint %}

### Working with Employees

**Endpoint:** `POST /pbxcore/api/v3/employees`

The table below lists the parameters for this request.

<table><thead><tr><th width="254.3671875">Field</th><th width="97.87109375">Req.</th><th width="164.66796875">Type / constraints</th><th>Description</th></tr></thead><tbody><tr><td><code>number</code></td><td>✅</td><td>string, 2–8 digits</td><td>Extension number</td></tr><tr><td><code>user_username</code></td><td>✅</td><td>string, 1–100 characters</td><td>Employee full name</td></tr><tr><td><code>sip_secret</code></td><td>✅</td><td>string, 5–100 characters</td><td>SIP account password</td></tr><tr><td><code>user_email</code></td><td>—</td><td>string email, ≤255</td><td>Email for notifications</td></tr><tr><td><code>mobile_number</code></td><td>—</td><td>string E.164, ≤50</td><td>Mobile number (+1...) for forwarding</td></tr><tr><td><code>mobile_dialstring</code></td><td>—</td><td>string, ≤255</td><td>Mobile dial string</td></tr><tr><td><code>sip_transport</code></td><td>—</td><td><code>udp</code> / <code>tcp</code> / <code>tls</code> / <code>udp,tcp</code></td><td>SIP transport (default: <code>udp</code>)</td></tr><tr><td><code>sip_dtmfmode</code></td><td>—</td><td><code>auto</code> / <code>rfc4733</code> / <code>inband</code> / <code>info</code></td><td>DTMF mode (default: <code>auto</code>)</td></tr><tr><td><code>sip_enableRecording</code></td><td>—</td><td>boolean</td><td>Call recording (default: <code>true</code>)</td></tr><tr><td><code>sip_networkfilterid</code></td><td>—</td><td>number | <code>"none"</code></td><td>Network filter ID</td></tr><tr><td><code>sip_manualattributes</code></td><td>—</td><td>string, ≤1024</td><td>Additional SIP parameters</td></tr><tr><td><code>fwd_ringlength</code></td><td>—</td><td>integer, ≤180</td><td>Ring time before forwarding (sec, default: 45)</td></tr><tr><td><code>fwd_forwarding</code></td><td>—</td><td>number | <code>hangup</code> | <code>busy</code></td><td>Unconditional forwarding</td></tr><tr><td><code>fwd_forwardingonbusy</code></td><td>—</td><td>number | <code>hangup</code> | <code>busy</code></td><td>Forwarding on busy</td></tr><tr><td><code>fwd_forwardingonunavailable</code></td><td>—</td><td>number | <code>hangup</code> | <code>busy</code></td><td>Forwarding on unavailable</td></tr></tbody></table>

#### **Creating a Single Employee**

```python
def create_employee(
    number: str,
    name: str,
    sip_secret: str,
    email: str = '',
    mobile: str = '',
    record_calls: bool = True,
    fwd_ringlength: int = 45,
) -> dict:
    payload = {
        'number':              number,
        'user_username':       name,
        'sip_secret':          sip_secret,
        'sip_enableRecording': record_calls,
        'fwd_ringlength':      fwd_ringlength,
    }
    if email:  payload['user_email']    = email
    if mobile: payload['mobile_number'] = mobile

    r = requests.post(f'{BASE_URL}/employees', headers=HEADERS, json=payload)
    result = r.json()
    if result.get('result'):
        print(f" Created: {number} ({name}), id={result['data']['id']}")
    else:
        print(f" Error: {result.get('messages', {}).get('error', [])}")
    return result


# Minimal example (required fields only)
create_employee(
    number='243',
    name='John Smith',
    sip_secret='Secure#Pass9201',
)

# Full example
create_employee(
    number='244',
    name='Anna Johnson',
    sip_secret='Secure#Pass9202',
    email='anna@company.com',
    mobile='79001234567',
    record_calls=True,
    fwd_ringlength=30,
)
```

Example API response (HTTP 201):

```json
{
  "result": true,
  "data": {
    "number": "201",
    "user_username": "John Smith",
    "sip_secret": "Secure#Pass9201",
    "sip_dtmfmode": "auto",
    "sip_transport": "udp",
    "sip_enableRecording": true,
    "sip_networkfilterid": "none",
    "fwd_ringlength": 45,
    "id": "1",
    "extensions_length": 3
  },
  "messages": {"error": [], "info": [], "warning": []}
}
```

Possible response codes:

| Code | Description                                                           |
| ---- | --------------------------------------------------------------------- |
| 201  | Employee successfully created                                         |
| 400  | Validation error (weak password <5 characters, invalid number format) |
| 401  | Invalid or missing API key                                            |
| 403  | No write permission for the `/employees` resource                     |
| 409  | Conflict — number already in use                                      |

On successful execution, you will see the following console output:

{% code overflow="wrap" %}
```python
 Created: 283 (John Smith), id=113
 Created: 284 (Anna Johnson), id=114

Process finished with exit code 0
```
{% endcode %}

Employees 283 and 284 will be created on the station.

<figure><img src="../../../.gitbook/assets/createdExtensionsWithAPI.png" alt=""><figcaption><p>Employees created via REST API</p></figcaption></figure>

#### **Listing Employees**

```python
def list_employees(search: str = '', limit: int = 100, offset: int = 0) -> list:
    params = {'limit': limit, 'offset': offset}
    if search: params['search'] = search
    r = requests.get(f'{BASE_URL}/employees', headers=HEADERS, params=params)
    return r.json().get('data', {}).get('data', [])

for emp in list_employees():
    print(f"  {emp.get('number'):>6}  {emp.get('user_username', '')}")
```

On successful execution, you will see the following console output:

{% code overflow="wrap" %}
```python
     202  Brown Brandon
     203  Collins Melanie
     201  Smith James
     283  John Smith
     284  Anna Johnson

Process finished with exit code 0
```
{% endcode %}

#### **Group Employee Creation**

```python
import time

employees = [
    {'number': '291', 'name': 'John Smith',   'secret': 'Pass#9201'},
    {'number': '292', 'name': 'Anna Johnson', 'secret': 'Pass#9202'},
    {'number': '293', 'name': 'Peter Brown',  'secret': 'Pass#9203'},
]

created, failed = [], []
for emp in employees:
    r = requests.post(
        f'{BASE_URL}/employees',
        headers=HEADERS,
        json={
            'number':        emp['number'],
            'user_username': emp['name'],
            'sip_secret':    emp['secret'],
        }
    )
    result = r.json()
    if result.get('result'):
        created.append(emp['number'])
        print(f" {emp['number']} {emp['name']}")
    else:
        failed.append(emp['number'])
        print(f" {emp['number']}: {result.get('messages', {}).get('error', [])}")
    time.sleep(0.2)  # small pause between requests

print(f'Created: {len(created)}, Errors: {len(failed)}')
```

On successful execution, you will see the following console output:

{% code overflow="wrap" %}
```python
 291 John Smith
 292 Anna Johnson
 293 Peter Brown
Created: 3, Errors: 0

Process finished with exit code 0
```
{% endcode %}

3 employees will be created on the station.

<figure><img src="../../../.gitbook/assets/created3ExtensionsWithAPI.png" alt=""><figcaption><p>Employees created via REST API</p></figcaption></figure>

### Working with SIP Providers

**Endpoint:** `POST /pbxcore/api/v3/sip-providers`

| Field               | Req. | Type    | Description                                              |
| ------------------- | ---- | ------- | -------------------------------------------------------- |
| `description`       | ✅    | string  | Provider name                                            |
| `host`              | ✅    | string  | Provider SIP server address                              |
| `username`          | —    | string  | Login on the provider's server                           |
| `secret`            | —    | string  | Password                                                 |
| `registration_type` | —    | string  | `inbound` / `outbound` / `none`                          |
| `qualify`           | —    | boolean | Availability monitoring (default: `true`)                |
| `transport`         | —    | string  | `udp` / `tcp` / `tls` / `udp,tcp` (default: `udp,tcp`)   |
| `dtmfmode`          | —    | string  | `auto` / `rfc4733` / `inband` / `info` (default: `auto`) |
| `port`              | —    | integer | Connection port (default: `5060`)                        |
| `disabled`          | —    | boolean | Disable provider (default: `false`)                      |

#### **Creating a Provider**

```python
def create_sip_provider(
    description: str,
    host: str,
    username: str = '',
    password: str = '',
    registration_type: str = 'outbound',
    qualify: bool = True,
) -> dict:
    payload = {
        'description': description,
        'host':        host,
    }
    if username:          payload['username']          = username
    if password:          payload['secret']            = password
    if registration_type: payload['registration_type'] = registration_type
    if not qualify:       payload['qualify']           = qualify

    r = requests.post(f'{BASE_URL}/sip-providers', headers=HEADERS, json=payload)
    result = r.json()
    if result.get('result'):
        print(f" Provider created: {description}")
    else:
        print(f" Error: {result.get('messages', {}).get('error', [])}")
    return result


create_sip_provider(
    description='Zadarma',
    host='sip.zadarma.com',
    username='316811',
    password='mysecretpass',
)
```

On successful execution, you will see the following console output:

{% code overflow="wrap" %}
```python
 Provider created: Zadarma

Process finished with exit code 0
```
{% endcode %}

A provider will be created on the station:

<figure><img src="../../../.gitbook/assets/createdProviderWithAPI.png" alt=""><figcaption><p>Provider created via REST API</p></figcaption></figure>

#### **Listing All Providers**

```python
def list_providers() -> list:
    r = requests.get(f'{BASE_URL}/sip-providers', headers=HEADERS)
    return r.json().get('data', [])

for prov in list_providers():
    print(f"  {prov.get('id'):<20} {prov.get('description', '')}  [{prov.get('type', '')}]")
```

On successful execution, you will see the following console output:

{% code overflow="wrap" %}
```python
  SIP-TRUNK-34F7CAFE     [SIP]
  SIP-TRUNK-7B5977ED     [SIP]

Process finished with exit code 0
```
{% endcode %}

### Retrieving Call History (CDR)

**Endpoint:** `GET /pbxcore/api/v3/cdr` — read-only.

| Parameter     | Type    | Description                                  |
| ------------- | ------- | -------------------------------------------- |
| `offset`      | integer | Pagination offset (default: 0)               |
| `limit`       | integer | Number of records, max. 100                  |
| `dateFrom`    | string  | Period start: `%Y-%m-%dT%H:%M:%S`            |
| `dateTo`      | string  | Period end: `%Y-%m-%dT%H:%M:%S`              |
| `src_num`     | string  | Filter by caller number                      |
| `dst_num`     | string  | Filter by destination number                 |
| `disposition` | string  | `ANSWERED` / `NO ANSWER` / `BUSY` / `FAILED` |

```python
from datetime import datetime, timedelta

def get_cdr(
    offset: int = 0,
    limit: int = 20,
    date_from: str = None,
    date_to: str = None,
    src_num: str = None,
    dst_num: str = None,
    disposition: str = None,
) -> list:
    params = {'offset': offset, 'limit': min(limit, 100)}
    if date_from:   params['dateFrom'] = date_from
    if date_to:     params['dateTo']   = date_to
    if src_num:     params['src_num']  = src_num
    if dst_num:     params['dst_num']  = dst_num
    if disposition: params['disposition'] = disposition

    r = requests.get(f'{BASE_URL}/cdr', headers=HEADERS, params=params)
    return r.json().get('data', {}).get('records', [])


now  = datetime.now()
then = now - timedelta(days=7)

for row in get_cdr(
    date_from=then.strftime('%Y-%m-%dT%H:%M:%S'),
    date_to=now.strftime('%Y-%m-%dT%H:%M:%S'),
):
    print(
        str(row.get('start', ''))[:16],
        row.get('src_num', ''), '→', row.get('dst_num', ''),
        row.get('disposition', ''), row.get('totalBillsec', 0), 's'
    )
```

On successful execution, you will see the following console output:

{% code overflow="wrap" %}
```python
2026-03-17 13:30 252 → 202 ANSWERED 48 s
2026-03-17 13:30 243 → 252 BUSY 0 s
2026-03-17 13:30 243 → 89161111111 CHANUNAVAIL 0 s
2026-03-17 13:29 202 → 243 NOANSWER 0 s
2026-03-17 13:29 202 → 202 ANSWERED 2 s
2026-03-17 13:29 202 → 243 NOANSWER 0 s
2026-03-17 13:29 202 → 10003246 NOANSWER 0 s
2026-03-17 13:28 202 → 243 NOANSWER 0 s

Process finished with exit code 0
```
{% endcode %}

#### **Statistics for a Period**

```python
def cdr_stats(days: int = 1) -> dict:
    now  = datetime.now()
    then = now - timedelta(days=days)
    records = get_cdr(
        date_from=then.strftime('%Y-%m-%dT%H:%M:%S'),
        date_to=now.strftime('%Y-%m-%dT%H:%M:%S'),
        limit=100
    )
    answered  = [r for r in records if r.get('disposition') == 'ANSWERED']
    missed = [r for r in records if r.get('disposition') in ('NO ANSWER', 'NOANSWER')]
    total_dur = sum(r.get('totalBillsec', 0) for r in answered)
    return {
        'total':    len(records),
        'answered': len(answered),
        'missed':   len(missed),
        'avg_sec':  total_dur // len(answered) if answered else 0,
    }

stats = cdr_stats(days=7)
print(f"Calls over 7 days: {stats['total']}")
print(f"Answered:          {stats['answered']}")
print(f"Missed:            {stats['missed']}")
print(f"Avg. duration:     {stats['avg_sec']}s")
```

On successful execution, you will see the following console output:

{% code overflow="wrap" %}
```python
Calls over 7 days: 13
Answered:          2
Missed:            5
Avg. duration:     25s

Process finished with exit code 0
```
{% endcode %}

{% hint style="info" %}
Calls with the `CHANUNAVAIL` status are not counted in the Answered, Missed, or Avg. duration statistics.
{% endhint %}

#### **CDR Record Fields**

| Field                     | Type     | Description                                                               |
| ------------------------- | -------- | ------------------------------------------------------------------------- |
| `linkedid`                | string   | Unique call identifier                                                    |
| `start`                   | datetime | Call start time                                                           |
| `src_num`                 | string   | Caller number                                                             |
| `src_name`                | string   | Caller name                                                               |
| `dst_num`                 | string   | Destination number                                                        |
| `dst_name`                | string   | Destination name                                                          |
| `disposition`             | string   | `ANSWERED` / `NO ANSWER` / `NOANSWER` / `BUSY` / `CHANUNAVAIL` / `FAILED` |
| `totalBillsec`            | integer  | Call duration (seconds)                                                   |
| `totalDuration`           | integer  | Total duration (including ringing)                                        |
| `records`                 | array    | Detailed records for each call leg                                        |
| `records[].recordingfile` | string   | Path to the recording file                                                |
| `records[].playback_url`  | string   | URL for playing back the recording                                        |
| `records[].download_url`  | string   | URL for downloading the recording                                         |
| `records[].dtmf_digits`   | string   | DTMF digits pressed in IVR                                                |

### Monitoring: SIP Statuses and Active Calls

#### **Employee and SIP Provider Registration Statuses**

**Endpoints:** `GET /pbxcore/api/v3/sip` , `GET /pbxcore/api/v3/sip-providers`

```python
from datetime import datetime

def show_employees():
    r = requests.get(f'{BASE_URL}/sip:getStatuses', headers=HEADERS)
    peers = r.json().get('data', {})
    for number, info in peers.items():
        icon = '🟢' if info.get('status') == 'Available' else '🔴'
        print(f"  {icon}  {number:>6}  {info.get('callerid', '')}  [{info.get('status', '')}]")


def show_providers():
    r = requests.get(f'{BASE_URL}/sip-providers:getStatuses', headers=HEADERS)
    providers = r.json().get('data', {}).get('sip', {})
    for prov_id, info in providers.items():
        icon = '🟢' if info.get('state') == 'registered' else '🔴'
        print(f"  {icon}  {info.get('description', prov_id):>20}  {info.get('username', '')}@{info.get('host', '')}  [{info.get('state', '')}]")


if __name__ == '__main__':
    print(f'MikoPBX Monitor [{datetime.now().strftime("%Y-%m-%d %H:%M:%S")}]')
    print('\n── Employees ───────────────────────────────')
    show_employees()
    print('\n── Providers ───────────────────────────────')
    show_providers()
```

On successful execution, you will see the following console output:

{% code overflow="wrap" %}
```python
MikoPBX Monitor [2026-03-17 16:47:35]

── Employees ───────────────────────────────
  🔴     201  Smith James  [Unavailable]
  🟢     202  Brown Brandon  [Available]
  🔴     203  Collins Melanie  [Unavailable]
  🔴     243  John Smith  [Unavailable]
  🟢     244  Anna Johnson  [Available]
  🔴     251  John Smith  [Unavailable]
  🟢     252  Anna Johnson  [Available]
  🔴     253  Peter Brown  [Unavailable]

── Providers ───────────────────────────────
  🔴         Demo provider  SIP-PROVIDER-122642725b9265fd7151c@demo.askozia.ru  [rejected]
  🟢               Zadarma  316811@sip.zadarma.com  [registered]

Process finished with exit code 0
```
{% endcode %}

**Employee statuses (`status` field)**

| Value         | Description              |
| ------------- | ------------------------ |
| `Available`   | Registered and available |
| `Unavailable` | Not registered (offline) |

**Provider statuses (`state` field)**

| Value          | Description                         |
| -------------- | ----------------------------------- |
| `registered`   | Registered on the provider's server |
| `rejected`     | Registration rejected by the server |
| `unregistered` | Not registered                      |

#### **Active Calls in Real Time**

**Endpoint:** `GET /pbxcore/api/v3/pbx-status`

```python
def get_active_calls() -> list:
    r = requests.get(f'{BASE_URL}/pbx-status:getActiveCalls', headers=HEADERS)
    return r.json().get('data', [])

calls = get_active_calls()

print(f'Active calls: {len(calls)}')
for call in calls:
    print(f"  {call.get('src_num', '?')} → {call.get('dst_num', '?')}  [{call.get('src_name', '')} → {call.get('dst_name', '')}]")
```

On successful execution, you will see the following console output:

{% code overflow="wrap" %}
```python
Active calls: 1
  243 → 252  [John Smith → Anna Johnson]

Process finished with exit code 0
```
{% endcode %}

> The full list of endpoints and interactive documentation is available in the [Interactive Documentation and Endpoint List](endpoints.md) section.
