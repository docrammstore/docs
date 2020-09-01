---
title: Trading API
category: API
order: 3
---

Trading API предназначен для передачи торговых приказов в систему RAMM.

Для упрощения настройки советников, использующих торговый API (т.к. требуется руками задавать исключения в настройках МТ), реализован всего один метод, покрывающий весь функционал передачи сигналов и получения всей нужной информации.

Например, https://ramm.store/api/trading/v1/signals.send

**Заголовок запроса**

Заголовок запроса имеет вид:

~~~
Content-Type: application/json
Token: 099b40eb-cca6-4316-8d35-cc333899f9a2
AppToken: 7f33625b-0e13-4d87-abf4-42eef85c6d99,
~~~

где Token относится к конкретной стратегии RAMM, а AppToken идентифицирует приложение, из которого отправлен сигнал.

**Команды**

Все команды имеют свой тип.

Type:

| Код | Значение |
| --- | --- |
| 0 | heartbeat |
| 1 | signal |
| 2 | sync |

Ответ содержится в коде http.

Коды ответов http:

| Код | Значение | Комментарий |
| --- | --- | --- |
| 200 | OK | Запрос выполнен успешно. |
| 400 | Bad Request | Запрос содержит синтаксическую ошибку. |
| 401 | Unauthorized | Токен стратегии и/или токен приложения не указаны или устарели. |
| 404 | Not Found | Инструмент не найден. |

Дополнительная информация возвращается в JSON.

**heartbeat** Данный сигнал сообщает серверу об активности приложения, передающего торговые приказы.

Рекомендуется отправлять регулярно, раз в 1 минуту.

Пример сигнала:

~~~json
{
    "Type": 0,
    "GetInfo": true
}
~~~

**Response**

В случае запроса с признаком GetInfo = false возвращается сокращенный набор параметров:

~~~json
{
    "App": {
        "CurrentVersion": "1.0.0.0",
        "NewestVersion": "1.0.0.1",
        "UpdateImportance": 0
    }
}
~~~

В случае запроса с признаком GetInfo = true возвращается полный набор параметров:

~~~json
{
    "App": {
        "CurrentVersion": "1.0.0.0",
        "NewestVersion": "1.0.0.1",
        "UpdateImportance": 0
    },
    "Strategy": {
        "Yield": 123.45,
        "Accounts": 123,
        "Status": 0,
        "NeedSync": false
    },
    "Account": {
        "Equity": 12333.45,
        "Margin": 454.56,
        "Factor": 1,
        "ProfitBase": 10000,
        "MCReached": "2018-12-11 08:22",
        "Protection": 0.2,
        "ProtectionEquity": 2133.23,
        "ProtectionReached": "2018-12-11 08:22",
        "Target": 1,
        "TargetEquity": 20000,
        "TargetReached": "2018-12-11 08:22",
        "Asset": "USD"
    }
}
~~~

Status:

| Код | Значение |
| --- | --- |
| 0 | not activated |
| 1 | active |
| 2 | paused |
| 3 | disabled |
| 4 | closed |

**signal** Отправлять сразу после исполнения ордеров на торговом счете. В случае, если по какой-либо причине накопилось несколько сигналов, – допускается отправить их все разом в одной коллекции.

~~~json
{
    "Type": 1,
    "Symbol": "MSFT",
    "Lot": 3.2,
    "Equity": 10025,
    "Ticket": "1234568",
    "Comment": "EA2"
}
~~~

**Response**

В случае успешного выполнения тело ответа сдержит ID сигнала. В случае неуспеха см. раздел "Ошибки".

**sync** Синхронизация. Отправляется при инициализации или при получении ответа от сервера о необходимости синхронизации.

~~~json
{
    "Type": 2,
    "Signals": [
        {
            "Symbol": "EURUSD",
            "Lot": 3.2,
            "Equity": 10025,
            "Ticket": "1234567",
            "Comment": "EA1"
        },
        {
            "Symbol": "MSFT",
            "Lot": 3.2,
            "Equity": 10025,
            "Ticket": "1234568",
            "Comment": "EA2"
        }
    ]
}
~~~

**Response**

В случае успешного выполнения тело ответа пустое. В случае неуспеха см. раздел "Ошибки".

Ответ на любой из запросов:

\{

Yield

Accounts

Status

Symbols

\}

**Ошибки и возможные коды ответа** Базовые ошибки возвращаются с помощью кодов HTTP. При необходимости, более подробный код ошибки возвращается в объекте error (в виде строки JSON).

Пример возвращаемого значения:

400

Bad Request

~~~json
{
    "error": {
        "code": "invalid_input",
        "message": "invalid input in field 'ID'"
    }
}
~~~

**Внутренние коды ошибок**

| Код | Расшифровка |
| --- | --- |
| incorrect\_method | The method you are trying to invoke doesn't exist |
| access\_denied | No rights to use the invoked method |
| invalid\_input | Invalid input in the field |

<field name=""></field>

&nbsp;

&nbsp;

&nbsp;

\| \| bad\_credentials \| Login/Password incorrect or not found \| \| login\_blocked \| The login you are using was blocked \|

&nbsp;

&nbsp;

&nbsp;