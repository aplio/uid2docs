---
title: POST /token/refresh
description: Use the refresh token to generate an updated UID2 token.
hide_table_of_contents: false
sidebar_position: 04
---

# POST /token/refresh
Generate a new [UID2 token](../ref-info/glossary-uid.md#gl-uid2-token) by sending the corresponding unexpired refresh token, returned by the [POST /token/generate](post-token-generate.md) endpoint.

Used by: This endpoint is used mainly by publishers.

>NOTE: This endpoint can be called from the client side (for example, a browser or a mobile app) because it does not require using an API key.

## Request Format 

`POST '{environment}/v2/token/refresh'`

Add the content of the `refresh_token` value, returned in the response from the previous [POST /token/generate](post-token-generate.md) or `POST /token/refresh` operation, as the POST body.

Here's what you need to know about this endpoint:

- No encryption is required for token refresh requests.
- If the request is successful, with an HTTP status code of 200, a new UID2 token or opt-out information is returned.
- Successful responses, whether the response includes a new token or opt-out information, are encrypted. Error responses are not encrypted.
- To decrypt responses, use the most recent `refresh_response_key` value for this token. The `refresh_response_key` value is returned in the response to the [POST /token/generate](post-token-generate.md) and `POST /token/refresh` operations. Each time a token is refreshed, a new `refresh_response_key` is returned. Be sure to use the most recent one to decrypt the current response.

### Path Parameters

| Path Parameter | Data Type | Attribute | Description |
| :--- | :--- | :--- | :--- |
| `{environment}` | string | Required | Testing environment: `https://operator-integ.uidapi.com`<br/>Production environment: `https://prod.uidapi.com`<br/>For a full list, including regional operators, see [Environments](../getting-started/gs-environments.md). |

>NOTE: The integration environment and the production environment require different [API keys](../ref-info/glossary-uid.md#gl-api-key).

#### Testing Notes

Using either of the following parameters in a [POST /token/generate](post-token-generate.md) request always generates an identity response with a `refresh_token` that results in a logout response when used with the `POST /token/refresh` endpoint:

- The `optout@email.com` email address
- The `+00000000000` phone number

### Request Example

For details and Python script examples, see [Encrypting Requests and Decrypting Responses (Python script example)](../getting-started/gs-encryption-decryption#uid2_requestpy).

## Decrypted JSON Response Format

A decrypted successful response includes a new UID2 token (`advertising_token`) and associated values for the user, or indicates that the user has opted out. 

>NOTE: The responses are encrypted only if the HTTP status code is 200. Error responses are not encrypted.

This section includes the following sample responses:

* [Successful Response With Tokens](#successful-response-with-tokens)
* [Successful Response With Opt-Out](#successful-response-with-opt-out)
* [Error Response](#error-response)

#### Successful Response With Tokens

If all values are valid and the user has not opted out, the response is successful and a new UID2 token is returned, with associated values. The following example shows a decrypted successful response with tokens:

```json
{
    "body": {
        "advertising_token": "NewAdvertisingTokenIjb6u6KcMAtd0/4ZIAYkXvFrMdlZVqfb9LNf99B+1ysE/lBzYVt64pxYxjobJMGbh5q/HsKY7KC0Xo5Rb/Vo8HC4dYOoWXyuGUaL7Jmbw4bzh+3pgokelUGyTX19DfArTeIg7n+8cxWQ=",
        "refresh_token": "NewRefreshTokenAAAF2c8H5dF8AAAF2c8H5dF8AAAADX393Vw94afoVLL6A+qjdSUEisEKx6t42fLgN+2dmTgUavagz0Q6Kp7ghM989hKhZDyAGjHyuAAwm+CX1cO7DWEtMeNUA9vkWDjcIc8yeDZ+jmBtEaw07x/cxoul6fpv2PQ==",
        "identity_expires": 1633643601000,
        "refresh_from": 1633643001000,
        "refresh_expires": 1636322000000,
        "refresh_response_key": "yptCUTBoZm1ffosgCrmuwg=="
    },
    "status": "success"
}
```

#### Successful Response With Opt-Out

If the user has opted out, the response is successful but a new UID2 token is not returned. The following example shows a decrypted opt-out response:

```json
{
  "status": "optout"
}
```

#### Error Response

An error response might look like the following:

```json
{
  "status": "client_error",
  "message": "Client Error"
}
```

### Response Body Properties

| Property | Data Type | Description |
| :--- | :--- | :--- |
| `advertising_token` | string | The [UID2 token](../ref-info/glossary-uid.md#gl-uid2-token) (also known as advertising token) for the user. |
| `refresh_token` | string | An encrypted token that can be exchanged with the UID2 Service for the latest set of identity tokens. |
| `identity_expires` | double | The UNIX timestamp (in milliseconds) that indicates when the UID2 token expires. |
| `refresh_from` | double | The UNIX timestamp (in milliseconds) that indicates when the UID2 SDK for JavaScript (see [UID2 SDK for JavaScript Reference Guide](../sdks/client-side-identity.md)) will start refreshing the UID2 token, if the SDK is in use.<br/>TIP: If you are not using the SDK, consider refreshing the UID2 token from this timestamp, too. |
| `refresh_expires` | double | The UNIX timestamp (in milliseconds) that indicates when the refresh token expires. |
| `refresh_response_key` | string | A key to be used in a new [POST /token/refresh](post-token-refresh.md) request for response decryption. |

### Response Status Codes

The following table lists the `status` property values and their HTTP status code equivalents.

| Status | HTTP Status Code | Description |
| :--- | :--- | :--- |
| `success` | 200 | The request was successful and a new UID2 token, with associated values, is returned in the response. The response is encrypted. |
| `optout` | 200 | The user opted out. This status is returned only for authorized requests. The response is encrypted. |
| `client_error` | 400 | The request had missing or invalid parameters.|
| `invalid_token` | 400 | The `refresh_token` value specified in the request was invalid. This status is returned only for authorized requests. |
| `expired_token` | 400 | The `refresh_token` value specified in the request was an expired token. |
| `unauthorized` | 401 | The request did not include a bearer token, included an invalid bearer token, or included a bearer token unauthorized to perform the requested operation. |

If the `status` value is anything other than `success` or `optout`, the `message` field provides additional information about the issue.
