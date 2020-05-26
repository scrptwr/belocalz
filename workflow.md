# Workflow

* [Database](#)
* [1. Customer app creates Belocalz order](#)
* [2. Belocalz creates XTM project](#)
* [3. XTM analysis callback](#)
* [4. Get XTM project price](#)
* [5. Belocalz analysis callback](#)
* [6. Approve Belocalz order](#)
* [7. Approve XTM project](#)
* [8. XTM translation callback](#)
* [9. Generate XTM target files](#)
* [10. Get status of generated XTM target files](#)
* [11. Download XTM target files](#)
* [12. Belocalz translation callback](#)
* [13. Customer app requests translation](#)
<br /><br /><br />

## Database
Initial database tables.

### Customers table
id   | xtmCustomerId | name     | email        | token       | password  | credits
:--- | :---          | :---     | :---         | :---        | :---      | :---
4323 | 54384         | Joe Inc. | john@doe.inc | 2cf24dba... | 587568... | 500

* `id` Customer identifier.
* `xtmCustomerId` XTM's customer identifier
* `name` Customer name.
* `email` Customer email address.
* `token` API bearer token to use for api.belocalz.com.
* `password` Password to login to www.belocalz.com.
* `credits` Number of credits the customer has purchased.

### Orders table
id     | xtmProjectId | customerId | reference | languageSource | languageTarget | status | credits | callbackAnalysis | callbackTranslation
:---   | :---         | :---       | :---      | :---           | :---           | :---   | :---    | :---             | :---
&nbsp; | &nbsp;       | &nbsp;     | &nbsp;    |&nbsp;          | &nbsp;         | &nbsp; | &nbsp;  | &nbsp;           | &nbsp; 

* `id` Order identifier.
* `xtmProjectId` Corresponding XTM project identifier.
* `customerId` Customer identifier to which customer this order belongs to.
* `reference` Customer's order reference.
* `languageSource` Language culture code e.g. en_US.
* `languageTarget` Language culture code e.g. es_ES.
* `status` One of *awaiting_analysis*, *awaiting_approvement*, *in_translation* or *available*.
* `credits` The number of credits the order costs.
* `callbackAnalysis` URL to call after xtm analysis is ready.
* `callbackTranslation` URL to call after xtm translation is ready.

### Jobs table
id     | orderId | reference | type   | bodySource | bodyTarget
:---   | :---    | :---      | :---   | :---       | :---
&nbsp; | &nbsp;  | &nbsp;    | &nbsp; | &nbsp;     | &nbsp;

* `id` Job identifier
* `orderId` Order identifier to which order this job belongs to.
* `reference` Customer's job reference.
* `type` Type of text the body uses. One of *text* or *html*.
* `bodySource` Strings for translation.
* `bodyTarget` Translated strings.
<br /><br /><br />

## 1. Customer app creates Belocalz order

### Request
```
METHOD  FROM             TO

POST    app.joe-inc.com  api.belocalz.com/v1/order
```

### Request body
```json
{
  "reference": 54678,
  "language_source": "en_US",
  "language_target": "es_ES",
  "callback_analysis": "https://app.joe-inc.com/analysis-done",
  "jobs": [
  	{
      "reference": 2587,
      "type": "html",
      "body_source": "<p>Hello <b>world</b></p>"
    }, {
      "reference": 6543,
      "type": "text",
      "body_source": "Hello moon"
    }
  ]
}
```

### Response body
```json
{
  "order_id": 6856,
  "job_count": 2,
  "status": "awaiting_analysis",
  "jobs": [
    {
      "job_id": 5467,
      "reference": 2587,
    }, {
      "job_id": 5468,
      "reference": 6543,
    }
  ]
}
```

### Database updates

#### Orders table
id     | xtmProjectId | customerId | reference | languageSource | languageTarget | status            | credits | callbackAnalysis                       | callbackTranslation
:---   | :---         | :---       | :---      | :---           | :---           | :---              | :---    | :---                                   | :---
6856   |              | 4323       | 54678     | en_US          | es_ES          | awaiting_analysis |         | `https://app.joe-inc.com/analysis-done`|

#### Jobs table
id     | orderId | reference | type   | bodySource                        | bodyTarget
:---   | :---    | :---      | :---   | :---                              | :---
5467   | 6856    | 2587      | html   | \<p\>Hello \<b\>world\</b\>\</p\> |
5468   | 6856    | 6543      | text   | hello moon                        |

### Back-end operations
* Save job bodySource string on server's tmp directory. Job id is used as filename. No file extension is used.
<br /><br /><br />

## 2. Belocalz creates XTM project

### Request
```
METHOD  FROM              TO

POST    api.belocalz.com  xtm.scriptware.nl/api/projects
```

### Headers
```
Authorization: XTM-Basic 354dfd4f...
Content-Type: multipart/form-data
```

### Request body
```
customerId: 54384
name: A project name
callbacks.analysisFinishedCallback: https://api.belocalz.nl/order/6856/analysed
callbacks.projectFinishedCallback: https://api.belocalz.nl/order/6856/translated
sourceLanguage: en_US,
targetLanguage: es_ES,
translationFiles[0].file: /ostmp/5467
translationFiles[1].file: /ostmp/5468
translationFiles[0].name: 5467.html
translationFiles[1].name: 5468.txt
```

### Response body
```json
{
  "jobs": [
    {
      "fileName": "5467.html",
      "jobId": 96754,
      "sourceLanguage": "en_US",
      "targetLanguage": "es_ES"
    }, {
      "fileName": "5468.txt",
      "jobId": 96755,
      "sourceLanguage": "en_US",
      "targetLanguage": "es_ES"
    }
  ],
  "name": "A project name"
  "projectId": 65789
}
```

### Database updates

#### Orders table
id     | xtmProjectId | customerId | reference | languageSource | languageTarget | status            | credits | callbackAnalysis                      | callbackTranslation
:---   | :---         | :---       | :---      | :---           | :---           | :---              | :---    | :---                                  | :---
6856   | 65789        | 4323       | 54678     | en_US          | es_ES          | awaiting_analysis |         | `https://app.joe-inc.com/analysis-done` |

#### Jobs table
id     | orderId | reference | type   | bodySource                        | bodyTarget
:---   | :---    | :---      | :---   | :---                              | :---
5467   | 6856    | 2587      | html   | \<p\>Hello \<b\>world\</b\>\</p\> |
5468   | 6856    | 6543      | text   | hello moon                        |

### Back-end operations
* File `/ostmp/5467` and `/ostmp/5468` removed after successful XTM project creation.
<br /><br /><br />

## 3. XTM analysis callback
After analysis is ready, XTM calls the analysis calback on Belocalz.

### Request
```
METHOD  FROM               TO

POST     xtm.scriptware.nl  api.belocalz.nl/order/6856/analysed
```

### Headers
```
Content-Type: application/json
```

#### Request body
```json
{
  "projectDescriptor": {
    "id": 87167
  },
  ...
}
```

#### Response body
```
```
<br /><br /><br />

## 4. Get XTM project price
Belocalz requests the XTM project price on XTM.

### Request
```
METHOD  FROM              TO

GET     api.belocalz.com  xtm.scriptware.nl/api/projects/65789/proposal
```

### Headers
```
Authorization: XTM-Basic 354dfd4f...
```

### Request body
```
```

### Response body
```
{
  "currency": "EUR",
  "deliveryDate": 1589621417889,
  "price": 250,
  "projectId": 65789,
  "taxPrice": 52.50
}
```

### Database updates

#### Orders table
id     | xtmProjectId | customerId | reference | languageSource | languageTarget | status                | credits | callbackAnalysis                        | callbackTranslation
:---   | :---         | :---       | :---      | :---           | :---           | :---                  | :---    | :---                                    | :---
6856   | 65789        | 4323       | 54678     | en_US          | es_ES          | awaiting_approvement  | 250     | `https://app.joe-inc.com/analysis-done` | 
<br /><br /><br />

## 5. Belocalz analysis callback
Belocalz calls the customer's analysis callback.

#### Request
```
METHOD	FROM				      TO

POST	  api.belocalz.com	app.joe-inc.com/analysis-done
```

### Headers
```
Content-Type: application/json
```

#### Request body
```json
{
  "order_id": 6856,
  "status": "awaiting_approvement",
  "credits": 250
}
```

#### Response body
```
```
<br /><br /><br />

## 6. Approve Belocalz order
Customer approves Belocalz order.

### Request
```
METHOD  FROM				     TO

POST    app.joe-inc.com  api.belocalz.com/v1/order/6856/approve
```

### Headers
```
Authorization: Bearer 2cf24dba...
Content-Type: application/json
```

### Request body
```json
{
  "callback_translation": "https://app.joe-inc.com/translation-done"
}
```

### Response body
```json
{
  "order_id": 6856,
  "status": "in_translation",
}
```

### Database updates

#### Orders table
id     | xtmProjectId | customerId | reference | languageSource | languageTarget | status          | credits | callbackAnalysis                        | callbackTranslation
:---   | :---         | :---       | :---      | :---           | :---           | :---            | :---    | :---                                    | :---
6856   | 65789        | 4323       | 54678     | en_US          | es_ES          | in_translation  | 250     | `https://app.joe-inc.com/analysis-done` | `https://app.joe-inc.com/translation-done`
<br /><br /><br />

## 7. Approve XTM project
Belocalz approves XTM project.

### Request
```
METHOD  FROM              TO
PUT     api.belocalz.com  xtm.scriptware.nl/projects/65789
```

### Headers
```
Authorization: XTM-Basic 354dfd4f...
Content-Type: application/json
```

### Request body
```
{
  "paymentStatus": "PAID",
  "proposalApprovalStatus": "CONFIRMED"
}
```

### Response body
```json
{
  "description": "",
  "name": "A project name",
  "paymentStatus": "PAID",
  "projectManagerId": 0,
  "proposalApprovalStatus": "CONFIRMED",
  "referenceId": "",
  "subjectMatterId": 0
}
```
<br /><br /><br />

## 8. XTM translation callback
After translation is ready, XTM calls the translation finished callback on Belocalz.

### Request
```
METHOD  FROM               TO

GET     xtm.scriptware.nl  api.belocalz.com/v1/order/6859/translated
```

### Headers
```
Authorization: XTM-Basic 354dfd4f...
```

### Request body
```
```

### Response body
```
```
<br /><br /><br />

## 9. Generate XTM target files
Start generating XTM target files

### Request
```
METHOD  FROM              TO

POST    api.belocalz.com  xtm.scriptware.nl/projects/65789/files/generate?fileType=TARGET
```

### Headers
```
Authorization: XTM-Basic 354dfd4f...
Content-Type: application/json
```

### Request body
```json
```

### Response body
```json
[
  {
    "fileId": 12879,
    "fileType": "TARGET",
    "jobId": 96754
  }, {
    "fileId": 12880,
    "fileType": "TARGET",
    "jobId": 96755
  }
]
```
<br /><br /><br />

## 10. Get status of generated XTM target files
Check if target files are finished generating regularly.

### Request
```
METHOD  FROM             TO

GET    api.belocalz.com  xtm.scriptware.nl/projects/65789/files/status?fileScope=PROJECT
```

### Headers
```
Authorization: XTM-Basic 354dfd4f...
```

### Request body
```json
```

### Response body
```json
[
  {
    "fileId": 12879,
    "jobId": 96754,
    "message": "",
    "projectId": 6856,
    "status": "IN_PROGRESS"
  },
  {
    "fileId": 12880,
    "jobId": 96755,
    "message": "",
    "projectId": 6856,
    "status": "IN_PROGRESS"
  }
]
```
<br /><br /><br />

## 11. Download XTM target files
Download the target files from XTM once they're available.

### Request
```
METHOD  FROM             TO

GET    api.belocalz.com  xtm.scriptware.nl/projects/65789/files/download?fileScope=PROJECT&fileType=TARGET
```

### Headers
```
Authorization: XTM-Basic 354dfd4f...
```

### Request body
```json
```

### Response body
```
application/octet-stream
```

### Back-end operations
* Save and extract XTM target zip.
* Save contents of individual files to database.
* Remove zip and extracted files.

### Database updates

#### Orders table
id     | xtmProjectId | customerId | reference | languageSource | languageTarget | status    | credits | callbackAnalysis                        | callbackTranslation
:---   | :---         | :---       | :---      | :---           | :---           | :---      | :---    | :---                                    | :---
6856   | 65789        | 4323       | 54678     | en_US          | es_ES          | available | 250     | `https://app.joe-inc.com/analysis-done` | `https://app.joe-inc.com/translation-done`

#### Jobs table
id     | orderId | reference | type   | bodySource                 | bodyTarget
:---   | :---    | :---      | :---   | :---                       | :---
5467   | 6856    | 2587      | html   | `<p>Hello<b>world</b></p>` | `<p>Hola<b>mundo</b></p>`
5468   | 6856    | 6543      | text   | `hello moon`               | `Hola Luna`
<br /><br /><br />

## 12. Belocalz translation callback
After translation is ready, Belocalz calls the translation finished callback on customers app.

### Request
```
METHOD  FROM              TO

POST    api.belocalz.com  app.joe-inc.com/translation-done
```

### Request body
```json
{
  "orderId": 6856
}
```

### Response body
```json
```
<br /><br /><br />

## 13. Customer app requests translation

### Request
```
METHOD  FROM             TO

GET     app.joe-inc.com  api.belocalz.com/v1/job/5467
```

### Headers
```
Authorization: XTM-Basic 354dfd4f...
```

### Request body
```json
```

### Response body
```json
{
  "jobId": 5467,
  "orderId": 6856,
  "reference": 2587,
  "status": "available",
  "type": "html",
  "languageSource": "en_US",
  "languageTarget": "es_ES",
  "bodySource": "<p>Hello<b>world</b></p>",
  "bodyTarget": "<p>Hola<b>Mundo</b></p>"
}
```