# Module auto dialer

## Auto Dialer Module

This module provides a REST API for creating campaigns to automatically dial external numbers:

* When an employee is available, the module dials a phone number from the campaign pool.
* When the client answers, the call is connected to the internal queue or the employee’s extension.
* If the internal number is busy, the call is automatically terminated.

### REST API Authentication

For all requests, except local ones, authentication is required.

By default, you can use the web interface's administrative password for authentication.

To restrict access to specific PBX settings, the "System Access Management" module can be used.

Example request:

```bash
curl 'http://172.16.156.223/admin-cabinet/session/start' \
-X 'POST' --cookie-jar auth-cookies.txt \
-H 'Content-Type: application/x-www-form-urlencoded; charset=UTF-8' \
-H 'X-Requested-With: XMLHttpRequest' \
--data 'login=admin&password=adminpassword'
```

Where:

* `login` – admin, the web interface username.
* `password` – adminpassword, the web interface password.
* `172.16.156.223` – the MikoPBX server address.

Response example:

```json
{
  "success": true,
  "reload": false,
  "message": []
}
```

Example of executing requests:

```bash
curl -b auth-cookies.txt 'http://172.16.156.223pbxcore/api/sip/getPeersStatuses'
```

### Polls

Allows you to organize automatic client surveys. Using the API, you can define a set of questions, their connections, and answer options.

It is possible to use text-to-speech generation or pre-recorded mp3/wav files.

### **Adding a Poll**

Example cURL request:

```bash
JSON='{
  "name": "New polling",
  "questions": [
    {
      "questionId": "1",
      "questionText": "Hello, we have a new product. To order, press 1. Not interested, press 2. To unsubscribe, press 3.",
      "press": [
        {
          "key": "1",
          "action": "playback",
          "value": "\"Thank you for choosing this product\"",
          "valueOptions": "text"
        },
        {
          "key": "3",
          "action": "playback",
          "value": "\"Thank you for your response\"",
          "valueOptions": "text"
        }
      ],
      "nextQuestion": ""
    }
  ]
}';
curl -X POST -d "$JSON" http://127.0.0.1/pbxcore/api/module-dialer/v1/polling
```

### **Fetching Poll List**

```bash
curl 'http://127.0.0.1/pbxcore/api/module-dialer/v1/polling'
```

Response example:

```json
{
  "result": true,
  "data": {
    "results": [
      {
        "id": "8",
        "crmId": "8",
        "name": "New polling"
      },
      {
        "id": "9",
        "crmId": "9",
        "name": "New polling"
      },
      {
        "id": "10",
        "crmId": "10",
        "name": "Product Survey"
      }
    ]
  }
}
```

### **Creating a Task**

```bash
JSON="{
  "name": "New polling task",
  "state": 0,
  "innerNum": "9",
  "innerNumType": "polling",
  "maxCountChannels": 1,
  "dialPrefix": "999",
  "numbers": [
    {
      "number": "79522233446"
    },
    {
      "number": "79522233416"
    }
  ]
}";
curl -X POST -d "$JSON" http://127.0.0.1/pbxcore/api/module-dialer/v1/task
```

Response example:

```json
{
  "result": true,
  "data": {
    "id": "41",
    "crmId": "41",
    "name": "New polling task",
    "innerNum": "9",
    "innerNumType": "polling",
    "maxCountChannels": 1,
    "state": 0,
    "dialPrefix": "999"
  }
}
```

### Media Files

**Uploading Media Files**

```bash
curl -F "file=@1.mp3" 'http://127.0.0.1/pbxcore/api/module-dialer/v1/audio'
```

Response example:

```json
{
  "result": true,
  "data": {
    "filename": "/storage/usbdisk1/mikopbx/custom_modules/ModuleAutoDialer/db/audio/filename.mp3"
  }
}
```
