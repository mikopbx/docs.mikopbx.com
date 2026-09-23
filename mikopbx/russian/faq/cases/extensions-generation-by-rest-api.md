---
description: Пример использования REST API для генерации сотрудников
---

# Программное создание сотрудников

В релизе 2023.2 реализовано API для быстрого создания большого количества сотрудников, в процессе тестирования мы описали алгоритм генерации нового сотрудников в ChatGPT и провели нагрузочное тестирование по созданию 700 произвольных учетных записей на разных языках. На загрузку ушло около 1 минуты, тест прошел успешно.

Для выполнения этой задачи был написан запрос к ChatGPT следущего содержания.

{% code fullWidth="true" %}
```
Мы разрабатываем сервер телефонии, и нам нужно протестировать функцию 
программного создания учетных записей сотрудников. 

Для получения структуры данных необходимо выполнить следующий GET запрос с пустым ID

http://127.0.0.1:8081/pbxcore/api/extensions/getRecord?id=

В ответ мы получим JSON примерно такого содержания:

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

Если параметр result в ответе будет равен true, то возьмем данные для создания нового пользователя 
из параметра data и выполним POST запрос на адрес http://127.0.0.1:8081/pbxcore/api/extensions/saveRecord,
используя полученный после модификации массив полей формы отправив его в формате x-www-form-urlencoded.

Скрипт должен быть написан на Python без использования NodeJS и позволять создать 100 различных сотрудников.
В поле user_username вписываем имя и фамилию произвольного человека, чтобы использовались национальные алфавиты.
В поле user_email произвольный адрес электронной почты.
В поле mobile_number произвольные мобильные номера телефонов, только цифры, без дублей.
В поле fwd_forwarding, fwd_forwardingonbusy заполнить значениями mobile_number.

При создании сотрудника, вывести user_username, user_email и number в консоль и сообщить о результате.

Если в результате запроса к saveRecord или getRecord мы получили result равный false, 
то также в консоль вывести параметр messages из полученного ответа. 
Если в результате ответа мы получили ошибку другого типа, или ответ невозможно разобрать,
то вывести в консоль полный текст полученный от API сервера.
```
{% endcode %}

В результате был сгенерирован Python скрипт, который позволил выполнить этот тест.

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
        # Получение шаблона данных для нового сотрудника
        get_response = requests.get("http://127.0.0.1:8081/pbxcore/api/extensions/getRecord?id=")

        if get_response.json().get('result', False) == False:
            print(f"getRecord error: {get_response.json().get('messages', 'Unknown error')}")
            continue

        data = get_response.json().get('data', {})

        # Генерация произвольных данных
        locale = random.choice(['ru_RU', 'en_US', 'zh_CN', 'ja_JP'])
        faker = locals()[f"faker_{locale.split('_')[0].lower()}"]
        data['user_username'] = faker.name()
        data['user_email'] = faker.email()
        data['mobile_number'] = generate_random_mobile()
        data['fwd_forwarding'] = data['mobile_number']
        data['fwd_forwardingonbusy'] = data['mobile_number']

        # Создание нового сотрудника
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

Для запуска подобного скрипта, сохраним его в папке /tmp внутри MikoPBX. Т.к. скрипт имеет несколько зависимостей, добавим их через установщик пакетов pip3 и запустим скрипт для генерации.

```bash
cd /tmp
python -m venv venv --without-pip
source venv/bin/activate
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
python get-pip.py
pip3 install faker requests
python CreateRandomExtensions.py
```

В результате работы скрипта мы увидим вывод в консоль списка создаваемых сотрудников, а в веб интерфейсе появятся новые учетные записи.

<figure><img src="../../.gitbook/assets/3. Processing.png" alt=""><figcaption><p>MikoPBX процесс нагрузочного тестирования интерфейса и REST API</p></figcaption></figure>

<figure><img src="../../.gitbook/assets/Screenshot 2023-09-06 at 14.04.52.png" alt=""><figcaption><p>MikoPBX результат работы скрипта генерации произольного набора сотрудников</p></figcaption></figure>

Важно отметить, т.к. система read-only, то установку Python библиотек необходимо выполнять посое каждой перезагрузки, а папка /tmp будет очищена. Разработку и доработку скрипта необходимо вести во внешней системе.

При выполнении REST API запросов из внешних адресов, необоходимо выполнить авторизацию административным логином и паролем или создать пользователя с правами на указанные действия в модуле [управления правами сотрудников](../../modules/miko/module-users-u-i.md) и авторизовываться им.

При выполнении запросов, как в этом примере, с адреса 127.0.0.1 авторизация не требуется.
