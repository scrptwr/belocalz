
## Order methods

- [Create order](#create-order)
- [Retrieve orders](#retrieve-orders)
- [Retrieve order](#retrieve-order)
- [Approve order](#approve-order)
- [Delete order](#delete-order)

## Job methods

- [Retrieve job](#retrieve-job)
- [Retrieve jobs](#retrieve-jobs)

---

### Create order
`POST` /v1/order

#### Header parameters

- `Authorization` *\<string\>* "**Bearer XXXXX**" where XXXXX stands for your token. 

#### Body parameters

application/json

- `language_source` *\<string\>* Culture code e.g. `en_US`
- `language_target` *\<string\>* Culture code e.g. `es_ES`
- `jobs` *\<array of objects\>*
  - `type` *\<string\>* One of `text` or `html`
  - `body_source` *\<string\>* String for translation
  - [`callback_analysis`] *\<string\>* URL to call after analysis is ready
  - [`callback_translation`] *\<string\>* URL to hit after translation is ready
  - [`reference`] *\<string\>* Reference for local administration

#### Example request

```json
{
  "language_source": "en_US",
  "language_target": "es_ES",
  "jobs": [
    {
      "reference": "2342",
      "type": "html",
      "body_source": "<p>Hello <b>world</b></p>"
    }, {
      "reference": "2343",
      "type": "text",
      "body_source": "Hello moon"
    }
  ]
}
```

#### Response

```json
200 OK

{
  "order_id": "685665",
  "job_count": "2",
  "credits": "",
  "status": "awaiting_analysis",
  "jobs": [
    {
      "job_id": "567643",
      "reference": "2342",
      "credits": ""
    }, {
      "job_id": "987043",
      "reference": "2343",
      "credits": ""
    }
  ]
}
```

---

### Retrieve orders

`GET` /v1/orders

#### Header parameters

- `Authorization` *\<string\>* "**Bearer XXXXX**" where XXXXX stands for your token.

#### Query parameters

- [`status`] *\<string\>* One of `awaiting_analysis`, `awaiting_approvement`, `in_translation` or `available`
- [`id`] *\<string\>*
- [`reference`] *\<string\>*

#### Example request

```
GET /v1/orders?status=awaiting_analysis
```

#### Response

```json
200 OK

[
  {
  	"order_id": "685665",
    "reference": "2342",
    "status": "awaiting_analysis"
  }
]
```

---

### Retrieve order

`GET` /v1/order/{id}

#### Header parameters

- `Authorization` *\<string\>* "**Bearer XXXXX**" where XXXXX stands for your token.

#### Query parameters

- `id` *\<string\>* Order id

#### Example request

```
GET /v1/order/873409
```

#### Response

```json
200 OK

{
  "order_id": "873409",
  "job_count": "3",
  "credits": "543",
  "status": "awaiting_approvement",
  "jobs": [
    {
      "job_id": "457821",
      "reference": "",
      "credits": "270"
    }, {
      "job_id": "565432",
      "reference": "",
      "credits": "220"
    }, {
      "job_id": "906543",
      "reference": "",
      "credits": "53"
    }
  ]
}
```

---

### Approve order

`POST` /v1/approve/{id}

Start the translation of an order.

#### Header parameters

- `Authorization` *\<string\>* "**Bearer XXXXX**" where XXXXX stands for your token.

#### Query parameters

- `id` *\<string\>* Order id

#### Example request

```
POST /v1/approve/873409
```

#### Response

```
200 OK

{
  "order_id": "873409",
  "job_count": "3",
  "credits": "543",
  "status": "in_translation",
  "jobs": [
    {
      "job_id": "457821",
      "reference": "",
      "credits": "270"
    }, {
      "job_id": "565432",
      "reference": "",
      "credits": "220"
    }, {
      "job_id": "906543",
      "reference": "",
      "credits": "53"
    }
  ]
}
```

#### Out of credits response

```
402 Payment Required

{
  "error": "Insufficient credits"
}
```

#### Delete order

`DELETE` /v1/order/{id}

Delete an unauthorized order.

#### Header parameters

- `Authorization` *\<string\>* "**Bearer XXXXX**" where XXXXX stands for your token.

#### Query parameters

- `id` *\<string\>* Order id

#### Example request

```
POST /v1/authorize/873409
```

#### Response

```
200 OK

{
}
```

---

### Retrieve job

`GET` /v1/job/{id}

#### Header parameters

- `Authorization` *\<string\>* "**Bearer XXXXX**" where XXXXX stands for your token.

#### Query parameters

- `id` *\<string\>* Job id

#### Example request

```
GET /v1/job/654532
```

#### Response

```
200 OK

{
  "job_id": "654532",
  "order_id": "345465"
  "reference": "",
  "status": "available"
  "type": "html",
  "credits": "32",
  "language_source": "en_US",
  "language_target": "de_DE",
  "body_source": "<p>Hello world</p>",
  "body_target": "<p>Hallo Welt</p>"
}
```

---

### Retrieve jobs
`GET` /v1/jobs

#### Header parameters

- `Authorization` *\<string\>* "**Bearer XXXXX**" where XXXXX stands for your token.

#### Query parameters

- [`reference`] *\<string\>*
- [`id`] *\<string\>* Job id
- [`status`] *\<string\>* One of `awaiting_analysis`, `awaiting_approvement`, `in_translation` or `available```