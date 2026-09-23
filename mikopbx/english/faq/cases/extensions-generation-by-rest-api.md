---
description: Example of using REST API to generate employees
---

# Generate extensions by REST API

In release 2023.2, an API for quickly creating a large number of employees was implemented. During testing, we described the algorithm for generating new employees in ChatGPT and conducted stress testing for creating 700 random accounts in different languages. It took about 1 minute to complete the load test, and it was successful.

To perform this task, a request to ChatGPT was written with the following content.

{% code fullWidth="true" %}
```
{"jsonapi":{"version":"1.0"},
    "result":true,
    "data":
        {
            "type":"SIP",
            "show_in_phonebook":"1",
            "is_general_user_number":"1",
            "public_access":"0",
            "number":"275",
            "user_id":"",
            "user_avatar":"",
            "user_username":"",
            "user_email":"",
            "mobile_uniqid":"EXTERNAL-7870338E",
            "mobile_number":"",
            "mobile_dialstring":"",
            "sip_uniqid":"SIP-PHONE-5A230111",
            "sip_secret":"Nk9oOFI2ZXdDMEU9",
            "sip_type":"peer",
            "sip_qualify":"1",
            "sip_qualifyfreq":60,
            "sip_enableRecording":"1",
            "sip_dtmfmode":"auto",
            "sip_transport":" ",
            "sip_networkfilterid":"none",
            "sip_manualattributes":"",
            "fwd_ringlength":45,
            "fwd_forwarding":"",
            "fwd_forwardingonbusy":"",
            "fwd_forwardingonunavailable":""
        },
    "messages":[],
    "function":"getRecord",
    "processor":"MikoPBX\\PBXCoreREST\\Lib\\Extensions\\GetRecord::main",
    "pid":42448,
    "meta":{
        "timestamp":"2023-09-06T08:51:51+03:00",
        "hash":"a16f973985cb3d0cbe75abb2c10a7c39b2203a87"
        }
}
```
{% endcode %}

If the result parameter in the response is true, we will take the data for creating a new user from the data parameter and make a POST request to the address http://127.0.0.1:8081/pbxcore/api/extensions/saveRecord, using the modified array of form fields in x-www-form-urlencoded format.

The script should be written in Python without using NodeJS and should allow creating 100 different employees. In the user\_username field, enter the name and surname of a random person so that national alphabets are used. In the user\_email field, enter any email address. In the mobile\_number field, enter random mobile phone numbers, only digits, without duplicates. In the fwd\_forwarding and fwd\_forwardingonbusy fields, fill in with mobile\_number values.

When creating an employee, print user\_username, user\_email, and number to the console and report the result.

If the result of the saveRecord or getRecord request is false, also print the messages parameter from the received response. If the response results in an error of another type or cannot be parsed, print the complete text received from the API server to the console.

As a result, a Python script was generated that allowed us to perform this test.

{% code title="CreateRandomExtensions.py" fullWidth="true" %}
```python
import requests
import random
from faker import Faker

def generate_random_mobile():
    return ''.join(random.choice("1234567890") for _ in range(11))

def main():
    faker_ru = Faker('ru_RU')
    faker_en = Faker('en_US')
    faker_zh = Faker('zh_CN')
    faker_ja = Faker('ja_JP')

    for number in range(1, 699):
        # Retrieving the data template for a new employee
        get_response = requests.get("http://127.0.0.1:8081/pbxcore/api/extensions/getRecord?id=")

        if get_response.json().get('result', False) == False:
            print(f"getRecord error: {get_response.json().get('messages', 'Unknown error')}")
            continue

        data = get_response.json().get('data', {})

        # Generating random data
        locale = random.choice(['ru_RU', 'en_US', 'zh_CN', 'ja_JP'])
        faker = locals()[f"faker_{locale.split('_')[0].lower()}"]
        data['user_username'] = faker.name()
        data['user_email'] = faker.email()
        data['mobile_number'] = generate_random_mobile()
        data['fwd_forwarding'] = data['mobile_number']
        data['fwd_forwardingonbusy'] = data['mobile_number']

        # Creating a new employee
        post_response = requests.post("http://127.0.0.1:8081/pbxcore/api/extensions/saveRecord", data=data, headers={'Content-Type': 'application/x-www-form-urlencoded'})

        if post_response.json().get('result', False):
            print(f"Created user {number} -> Username: {data['user_username']}, Email: {data['user_email']}, Mobile: {data['mobile_number']}")
        else:
            print(f"saveRecord error for user {number}: {post_response.json().get('messages', 'Unknown error')}")

        if post_response.status_code != 200 or not isinstance(post_response.json(), dict):
            print(f"Unknown error: {post_response.text}")

if __name__ == "__main__":
    main()

```
{% endcode %}

Save file content in /tmp/CreateRandomExtensions.py and run in with the following approach.

```bash
cd /tmp
python -m venv venv --without-pip
source venv/bin/activate
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
python get-pip.py
pip3 install faker requests
python CreateRandomExtensions.py
```

As a result of the script's execution, you will see a list of created employees in the console, and new user accounts will appear in the web interface.

<figure><img src="../../.gitbook/assets/3. Processing.png" alt=""><figcaption><p>MikoPBX stress testing process for the interface and REST API</p></figcaption></figure>

<figure><img src="../../.gitbook/assets/ResultOfGenerationExtensions.png" alt=""><figcaption><p>MikoPBX result of the script for generating a random set of employees</p></figcaption></figure>

It's important to note that since the system is read-only, you need to install Python libraries after each reboot, and the /tmp folder will be cleared. Development and refinement of the script should be done in an external system.

When making REST API requests from external addresses, it's necessary to authenticate with an administrative username and password or create a user with the necessary permissions in the employee rights management module and authenticate with it.

When making requests, as in this example, from the address 127.0.0.1, no authentication is required.
