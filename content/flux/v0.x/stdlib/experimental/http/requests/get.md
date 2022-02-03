---
title: requests.get() function
description: >
  `requests.get()` makes an HTTP request using the GET request method.
menu:
  flux_0_x_ref:
    name: requests.get
    parent: requests
weight: 401
flux/v0.x/tags: [http, inputs]
introduced: 0.152.0
---

`requests.get()` makes an HTTP request using the GET request method.

```js
import "experimental/http/requests"

requests.get(
    url: "http://example.com",
    params: [:],
    headers: [:],
    body: bytes(v: ""),
    config: requests.defaultConfig,
)
```

`requests.get()` returns a record with the following properties:

- **statusCode**: HTTP status code of the request.
- **body**: Response body. A maximum size of 100MB is read from the response body.
- **headers**: Response headers.

## Parameters

### url {data-type="string"}
URL to send the request to.

{{% note %}}
The URL should not include any query parameters.
Use [`params`](#params) to specify query parameters.
{{% /note %}}

### params {data-type="dict"}
Set of key-value pairs to add to the URL as query parameters.
Query parameters are URL-encoded.
All values for a key are appended to the query.

### headers {data-type="dict"}
Set of key values pairs to include as request headers.

### body {data-type="bytes"}
Data to send with the request.

### config {data-type="record"}
Set of request configuration options.

## Examples

### Make a GET request
```js
import "experimental/http/requests"

requests.get(url:"http://example.com")
```