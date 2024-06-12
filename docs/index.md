# AI-MD SDK Documentation

## Introduction

The AI-MD SDK allows developers to quickly add AI-MD's symptom checker, including a basic vitals face scan, to their application. It is available through a cloud connection to AI-MD's API at `my.ai-md.com`.

## Authorization

Please contact AI-MD for an **API key** generated for your client user. Once registered, provide the API key in the `Authorization` header to all HTTP calls made to the API:

```http
GET my.ai-md.com HTTP/1.1
Content-Type: application/json
...
Authorization: YOUR_API_KEY
```

If authorization is unsuccessful, you may receive one of the following status responses:

| Error Response                                     | Troubleshooting                                                                                       |
| -------------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| `401 Unauthorized {"detail": "Not Authenticated"}` | Please check if the `Authorization` header was passed correctly and the request URL is a valid route. |
| `401 Unauthorized {"detail": "Invalid API Key"}`   | Please ensure the API key is correct.                                                                 |

## Symptom Checker

To access the symptom checker, send a `POST` request to the assessment URL: `my.ai-md.com/client/assess`. The JSON body send with the POST request should contain the user's device type, user demographics and symptom data:

| Parameter Name                        | Description                                                                                       | Possible Values                            |
| ------------------------------------- | ------------------------------------------------------------------------------------------------- | ------------------------------------------ |
| **device_type**                       | The user's operating system (for the face scan).                                                  | `WINDOWS`,`LINUX`,`ANDROID_PHONE`,`IPHONE` |
| **demographics**                      | User demographics JSON object (see keys below).                                                   |                                            |
| **demographics.age**                  | User age.                                                                                         | Integer (years) between 13-120             |
| **demographics.gender**               | User sex assigned at birth.                                                                       | `male`,`female`                            |
| **demographics.height**               | User height in centimeters.                                                                       | Integer (cm) between 120-220               |
| **demographics.weight**               | User weight in kilograms.                                                                         | Integer (kg) between 30-300                |
| **symptoms** (_optional_)             | Symptoms details JSON object (see keys below). If not entered, symptom analysis will not be done. |                                            |
| **symptoms.description** (_optional_) | A detailed description of user symptoms.                                                          | String                                     |
| **symptoms.started_at** (_optional_)  | Datetime when user reports symptoms started.                                                      | Datetime in ISO format with timezone (UTC) |

Example Request body with symptoms (JSON):

```json
{
  "device_type": "WINDOWS",
  "demographics": {
    "age": 30,
    "gender": "male",
    "height": 180,
    "weight": 80
  },
  "symptoms": {
    "description": "I have a sore throat which has become painful, and experience a delibitating cough intermittently throughout the day. My chest also hurts.",
    "started_at": "2024-06-11T01:46:10.630Z"
  }
}
```

Without symptoms:

```json
{
  "device_type": "WINDOWS",
  "demographics": {
    "age": 30,
    "gender": "male",
    "height": 180,
    "weight": 80
  }
}
```

#### Example Request

```bash
curl -X 'POST' \
  'https://my.ai-md.com/client/assess' \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{
  "device_type": "WINDOWS",
  "demographics": {
    "age": 30,
    "gender": "male",
    "height": 180,
    "weight": 80
  },
  "symptoms": {
    "description": "I have a sore throat which has become painful, and experience a delibitating cough intermittently throughout the day. My chest also hurts.",
    "started_at": "2024-06-11T01:46:10.630Z"
  }
}'
```

#### Response

If the request is successful, a `200` status response will be returned containing a `redirect_url` for the face scan, as well as a `session_id` to identify the unique assessment:

```json
{
  "redirect_url": "https://scan.ai-md.com/c/...",
  "session_id": "abc..."
}
```

Please redirect the user's browser to this URL to complete the face scan, and store the `session_id` for session identification upon completion (see Redirect URL below).

### Redirect URL

On registration with `AI-MD`, you provide a **redirect URL** to receive the symptom checker results. Upon completion of the assessment, the user's browser will redirect to your provided redirect URL with two query parameters:

| Parameter Name | Description                                                                       |
| -------------- | --------------------------------------------------------------------------------- |
| **results**    | base64-encoded JSON string. On decoding, will consist of:                         |
|                | a `vitals` key consisting of user's vitals measured by the face scan.           |
|                | If `symptoms` were provided, a `conditions` key exists with possible diagnoses. |
| **session_id** | A string to identify the assessment that was provided with the `redirect_url`.    |
