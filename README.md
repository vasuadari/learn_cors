# Cross-Origin Request Sharing (CORS)

## Contents
1. [Pre-Requisites](#user-content-pre-requisites)
2. [Why do we need CORS?](#user-content-why-do-we-need-cors?)
3. [Introduction](#user-content-introduction)
4. [Basics](#user-content-basics)
    - [Request Headers](#user-content-request-headers)
      - [Origin](#user-content-origin)
      - [Access-Control-Request-Method](#user-content-allow-control-request-method)
      - [Access-Control-Request-Headers](#user-content-allow-control-request-headers)
      - [CORS-safelisted](#user-content-cors-safelisted)
    - [Response Headers](#user-content-response-headers)
      - [Access-Control-Allow-Origin](#user-content-access-control-allow-origin)
      - [Access-Control-Allow-Methods](#user-content-access-control-allow-methods)
      - [Access-Control-Allow-Headers](#user-content-ontrol-allow-headers)
      - [Access-Control-Max-Age](#user-content-access-control-max-age)
      - [Vary](#user-content-vary)
    - [Simple Requests](#user-content-simple-requests)
    - [Preflight Requests](#user-content-preflight-requests)
    - [Request with credentials](#user-content-request-with-credentials)
5. [Advanced Topics](#user-content-advanced-topics)
    - [Preflighted requests and redirects](#user-content-preflighted-requests-and-redirects)
        - [Access-Control-Allow-Credentials](#user-content-access-control-allow-credentials)
    - [Credentialed requests and wildcards](#user-content-credentialed-requests-and-wildcards)
    - [Third-party cookies](#user-content-third-party-cookies)
    - [Additional HTTP Response Headers](#user-content-additional-http-response-headers)
        - [Access-Control-Expose-Headers](#user-content-access-control-expose-headers)
6. [References](#user-content-references)

## Pre-Requisites

Know difference between [Simple HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview) and [XHR](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest) request.

## Why do we need CORS?

Origin Web App - https://github.com

When you open a website by typing Origin Web App, your browser makes Simple HTTP requests. CORS is not applicable here.
As we are voluntarily requesting to serve content from Origin Web App. Your browser will/should serve it without any
trouble. But, when it comes to XHR requests, JavaScript which is loaded by Origin Web App in your browser can make a
request to any server(https://netflix.com) requesting for a resource. Now, Origin Web App is owned by Github
but https://netflix.com is owned by a company named Netflix, that server could pontentially serve anything.
Github cannot take control of a server which is owned by Netflix. This has lot of security implications pontentially stealing a
content from one website(it could be a financial website) to any remote server.

Luckily, CORS tackles this problem very well with a given set of rules.

## Introduction

Its a standard defined by [W3C](https://en.wikipedia.org/wiki/World_Wide_Web_Consortium) to enable cross-origin requests between client(browser) and the server to share
resources at the same time maintaining security. Any browser will comply with these standards to prevent
loading resources from any third-party servers.

## Basics

### Request Headers

#### Origin

A header indicates where a request originates from. It is sent with CORS requests,
as well as with POST requests.

**Syntax:**

```
Origin: <scheme> "://" <hostname> [":" <port> ]
```

**Examples:**

```
Origin: https://netflix.com
Origin: http://netflix.com:443
Origin: http://localhost:1443
```

#### Access-Control-Request-Method

This header is used by browser when issuing a [pre-flight](#user-content-preflight-requests) requests to indicate which
request method will be used when the actual request is made.

**Syntax:**

```
Access-Control-Request-Method: <method>
```

`<method>` could be either `GET`, `POST` or `DELETE`.

**Example:**

```
Access-Control-Request-Method: POST
```

#### Access-Control-Request-Headers

This header is again used in the [pre-flight](#user-content-preflight-requests) requests by browser to indicate which
request headers are to be used when the actual request is made. To use multiple headers,
it has to be separated with comma.

**Syntax:**

```
Access-Control-Request-Headers: <header-name>
Access-Control-Request-Headers: <header-name>, <header-name>
```

**Example:**

```
Access-Control-Request-Headers: X-PINGOTHER, Content-Type
```

**Important:** All the header should be [CORS-safelisted](#user-content-cors-safelisted) or a customer header like `X-Custom-Header`.

#### CORS-safelisted

- `Accept`

- `Accept-Language`

- `Content-Language`

- `Content-Type`

  Allowed values are `application/x-www-form-urlencoded`, `multipart/form-data` and
  `text/plain`.

- `Connection`

- `User-Agent`

### Response Headers

The following headers are return in the [pre-flight](#user-content-pre-flight-request) requests.

#### Access-Control-Allow-Origin

This header indicates that if the requested [Origin](#user-content-origin) is allowed. Your browser will
choose to succeed/fail the request by matching the requested origin with this.

**Syntax:**

```
Access-Control-Allow-Origin: *
```

For requests without [credentails](#user-content-request-with-credentials), the value `*` can be specified as a wildcard.
This tells your browser to allow requests from any [Origin](#user-content-origin).

```
Access-Control-Allow-Origin: <origin>
```

When you receive only one origin in the response header, it means your server/web-app
based on the requested `Origin` it responds with same `Origin` if its allowed.
Your server should also respond with [Vary](#user-content-vary) to indicate that
it varies based on a request header.

**Example:**

```
Access-Control-Allow-Origin: https://github.com
```

#### Access-Control-Allow-Methods

Your browser will make actual request if one of the value in this header matches.
When wildcard `*` is returned, it means any method is allowed.

**Syntax:**

```
Access-Control-Allow-Methods: <method>, <method>, ...
Access-Control-Allow-Methods: *
```

**Example:**

```
Access-Control-Allow-Methods: GET, POST
```

#### Access-Control-Allow-Headers

You browser will make actual request if all of the requested headers are allowed.

**Syntax:**

```
Access-Control-Allow-Headers: <header-name>, <header-name>
Access-Control-Allow-Headers: *
```

**Example:**

```
Access-Control-Allow-Headers: Accept, Content-Type
```

Wildcard `*` tells browser that any header is allowed.

#### Access-Control-Max-Age

Tells how long the results of [pre-flight](#user-content-pre-flight-requests) can
be cached.

```
Access-Control-Max-Age: <delta-seconds>
```

<delta-seconds> maximum no. of seconds the results can be cached.

Each browser has max limit,
- Firefox caps this at 24 hours(86400 seconds).
- Chromium (prior to v76) caps at 10 minutes (600 seconds).
- Chromium (starting in v76) caps at 2 hours (7200 seconds).
- Chromium also specifies a default value of 5 seconds.
- A value of -1 will disable caching, requiring a preflight OPTIONS check for all calls.

#### Vary

Vary header in general used for caching purpose to determine whether a cached response
can be used or it has to be re-cached based on the header value.

In CORS, lets say server allows multiple origins based on the requested `Origin` it will
return particular URL in `Access-Control-Allow-Origin`.

**Syntax:**

```
Vary: <header-name>
```

**Example:**

Lets say github.com allows both https://github.com as well as https://netflix.com to request for resources.
Consider following scenarios,

**Scenario 1:**

curl -X OPTIONS https://github.com/api/v1/gists/1

```
Origin: https://github.com
Access-Control-Request-Method: GET
Access-Control-Request-Headers: Content-Type
```

```
Access-Control-Allow-Origin: https://github.com
Vary: Origin
Access-Control-Allow-Methods: GET, POST
Access-Control-Allow-Headers: Content-Type
Access-Control-Max-Age: 600
```

Now in this scenario, browser will cache this pre-flight request results for
10 minutes.

**Scenario 2:**

curl -X OPTIONS https://github.com/api/v1/gists/1

```
Origin: https://netflix.com
Access-Control-Request-Method: GET
Access-Control-Request-Headers: Content-Type
```

```
Access-Control-Allow-Origin: https://netflix.com
Vary: Origin
Access-Control-Allow-Methods: GET, POST
Access-Control-Allow-Headers: Content-Type
Access-Control-Max-Age: 300
```

Now you could notice that `Access-Control-Allow-Origin` contains https://netflix.com,
this is example of how the response varies based on given `Origin`. So does the
max-age of this response which is cached for only 5 minutes unlike first scenario.

### Simple Requests

### Preflight Requests

### Request with credentials

## Advanced Topics

### Preflighted requests and redirects

#### Access-Control-Allow-Credentials

### Credentialed requests and wildcards

### Third-party cookies

### Additional HTTP Response Headers

#### Access-Control-Expose-Headers

## References

- [Cross-Origin Resource Sharing (CORS) by Mozilla](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)
