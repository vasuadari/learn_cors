# Cross-Origin Resource Sharing (CORS)

## Contents
1. [Pre-Requisites](#1-pre-requisites)
2. [Why do we need CORS?](#2-why-do-we-need-cors?)
3. [Introduction](#3-introduction)
4. [Basics](#4-basics)
    - 4.1. [Request Headers](#41-request-headers)
      - 4.1.1. [Origin](#411-origin)
      - 4.1.2. [Access-Control-Request-Method](#412-access-control-request-method)
      - 4.1.3. [Access-Control-Request-Headers](#413-access-control-request-headers)
      - 4.1.4. [CORS-safelisted](#414-cors-safelisted)
    - 4.2. [Response Headers](#42-response-headers)
      - 4.2.1. [Access-Control-Allow-Origin](#421-access-control-allow-origin)
      - 4.2.2. [Access-Control-Allow-Methods](#422-access-control-allow-methods)
      - 4.2.3. [Access-Control-Allow-Headers](#423-access-control-allow-headers)
      - 4.2.4. [Access-Control-Max-Age](#424-access-control-max-age)
      - 4.2.5. [Vary](#425-vary)
    - 4.3. [Simple Requests](#43-simple-requests)
    - 4.4. [Preflight Requests](#44-preflight-requests)
    - 4.5. [Request with credentials](#45-request-with-credentials)
5. [Advanced Topics](#5-advanced-topics)
    - 5.1. [Additional HTTP Response Headers](#51-additional-http-response-headers)
      - 5.1.1. [Access-Control-Expose-Headers](#511-access-control-expose-headers)
      - 5.1.2. [Access-Control-Allow-Credentials](#512-access-control-allow-credentials)
    - 5.2. [Credentialed requests and wildcards](#52-credentialed-requests-and-wildcards)
    - 5.3. [Preflighted requests and redirects](#53-preflighted-requests-and-redirects)
    - 5.4. [Third-party cookies](#54-third-party-cookies)
6. [TODO](#6-todo)
7. [References](#7-references)

## 1. Pre-Requisites

Know difference between [Simple HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview) and [XHR](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest) request.

## 2. Why do we need CORS?

Origin Web App - https://github.com

When you open a website by typing Origin Web App, your browser makes Simple HTTP requests. CORS is not applicable here.
As we are voluntarily requesting to serve content from Origin Web App. Your browser will/should serve it without any
trouble. But, when it comes to XHR requests, JavaScript which is loaded by Origin Web App in your browser can make a
request to any server(https://netflix.com) requesting for a resource. Now, Origin Web App is owned by Github
but https://netflix.com is owned by Netflix, that server could pontentially serve anything. Github cannot take control
of a server which is owned by Netflix. This has lot of security implications pontentially stealing a content from one
website(it could be a financial website) to any remote server.

Luckily, CORS tackles this problem very well with a given set of rules.

## 3. Introduction

Its a standard defined by [W3C](https://en.wikipedia.org/wiki/World_Wide_Web_Consortium) to enable cross-origin requests between client(browser) and the server to share
resources at the same time maintaining security. Any browser will comply with these standards to prevent
loading resources from any third-party servers.

## 4. Basics

### 4.1. Request Headers

#### 4.1.1. Origin

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

#### 4.1.2. Access-Control-Request-Method

This header is used by browser when issuing a [pre-flight](#44-preflight-requests) requests to indicate which
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

#### 4.1.3. Access-Control-Request-Headers

This header is again used in the [pre-flight](#44-preflight-requests) requests by browser to indicate which
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

**Important:** All the header should be [CORS-safelisted](#414-cors-safelisted) or a customer header like `X-Custom-Header`.

#### 4.1.4. CORS-safelisted

- `Accept`

- `Accept-Language`

- `Content-Language`

- `Content-Type`

  Allowed values are `application/x-www-form-urlencoded`, `multipart/form-data` and
  `text/plain`.

### 4.2. Response Headers

The following headers are return in the [pre-flight](#44-preflight-requests) requests.

#### 4.2.1. Access-Control-Allow-Origin

This header indicates that if the requested [Origin](#411-origin) is allowed. Your browser will
choose to succeed/fail the request by matching the requested origin with this.

**Syntax:**

```
Access-Control-Allow-Origin: *
```

For requests without [credentials](#45-request-with-credentials), the value `*` can be specified as a wildcard.
This tells your browser to allow requests from any [Origin](#411-origin).

```
Access-Control-Allow-Origin: <origin>
```

When you receive only one origin in the response header, it means your server/web-app
based on the requested `Origin` it responds with same `Origin` if its allowed.
Your server should also respond with [Vary](#425-vary) to indicate that
it varies based on a request header.

**Example:**

```
Access-Control-Allow-Origin: https://github.com
```

#### 4.2.2. Access-Control-Allow-Methods

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

#### 4.2.3. Access-Control-Allow-Headers

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

#### 4.2.4. Access-Control-Max-Age

Tells how long the results of [pre-flight](#44-preflight-requests) can
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

#### 4.2.5. Vary

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

### 4.3. Simple Requests

Some request don't trigger [pre-flight](#44-preflight-requests) request for CORS.
These we call it as *simple requests*. It should meet the following conditions:

- One of the allowed methods:
  - GET
  - POST
  - DELETE

- Other than headers which are automatically by the user agent (for example, `Connection` or
`User-Agent`), the only headers which are allowed to manually set are [CORS-safelisted](#414-cors-safelisted)
request headers and the following:

  - `DPR`
  - `Downlink`
  - `Save-Data`
  - `Viewport-Width`
  - `Width`

- No event listeners are registered on any `XMLHttpRequestUpload` object used in the
request.

- No `ReadableStream` object is used in the request.

**Example:**

*Request:*
```
GET /api/v1/public-data/ HTTP/1.1
Host: bar.com
User-Agent: curl/7.69.1
Accept: */*
Origin: https://foo.com
```

*Response Headers:*
```
HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
Connection: Keep-Alive
Transfer-Encoding: chunked
Content-Type: application/xml
```

When a request returns `*` in `Access-Control-Allow-Origin` header. It means a request
can be made from any `Host`. In this case, your browser won't make pre-flight request.

### 4.4. Preflight Requests

Your browser will make pre-flight request to determine if the actual request is safe
to send.

**Example:**

```JavaScript
const xhr = new XMLHttpRequest();
xhr.open('POST', 'https://bar.com/api/v1/post-here/');
xhr.setRequestHeader('X-PINGOTHER', 'pingpong');
xhr.setRequestHeader('Content-Type', 'application/json;charset=UTF-8');
xhr.onreadystatechange = handler;
xhr.send(JSON.stringify({ "email": "foo@bar.com" }));
```

*Pre-flight Request:*

```
OPTIONS /api/v1/post-here/
Host: bar.com
User-Agent: curl/7.69.1
Accept: */*
Origin: https://foo.com
Access-Control-Request-Method: POST
Access-Control-Request-Headers: X-PINGOTHER, Content-Type

HTTP/1.1 200 OK
Connection: Keep-Alive
Content-Type: application/json
Access-Control-Allow-Origin: https://foo.com
Access-Control-Allow-Methods: GET, POST
Access-Control-Allow-Headers: X-PINGOTHER, Content-Type, Accept
```

*Actual Request:*

```
POST -d '{foo: bar}' /api/v1/post-here/
Host: bar.com
User-Agent: curl/7.69.1
Accept: */*
Origin: https://foo.com
X-PINGOTHER: pingpong
Content-Type: application/json;charset=UTF-8

HTTP/1.1 200 OK
Connection: Keep-Alive
Content-Type: application/json
Access-Control-Allow-Origin: https://foo.com
```

When a non-standard header like `X-PINGOTHER` is set, your browser won't know if its
safe to make the request. In order to make sure its safe, your browser makes OPTIONS
request with `Access-Control-Request-Headers` containing `X-PINGOTHER` and `Content-Type`. Upon validating with [response headers](#42-response-headers) of pre-flight
request, your browser makes actual request.

### 4.5. Request with credentials

In general, when you make a XHR request, cookies are not passed along with request.
When there is need to pass cookies, you'll have to set a flag on the `XMLHttpRequest`
object.

```JavaScript
const xhr = new XMLHttpRequest();
const url = 'http://bar.com/api/v1/credentialed-content/';

function callOtherDomain() {
  if (invocation) {
    xhr.open('GET', url, true);
    xhr.withCredentials = true;
    xhr.onreadystatechange = handler;
    xhr.send();
  }
}
```

*Request:*

```
POST -d '{foo: bar}' /api/v1/post-here/
Host: bar.com
User-Agent: curl/7.69.1
Accept: */*
Origin: https://foo.com
X-PINGOTHER: pingpong
Content-Type: application/json;charset=UTF-8
Cookie: _session=NyV14oKXiS6HHopaf9nT
```

When a request is made `XMLHttpRequest` and `withCredentials` flag is set, your
browser will pass down the `Cookie` in the request header.

*Response:*
```
HTTP/1.1 200 OK
Connection: Keep-Alive
Content-Type: application/json
Access-Control-Allow-Origin: https://foo.com
Access-Control-Allow-Credentials: true
Cache-Control: no-cache
Pragma: no-cache
Set-Cookie: _session=AjBSqxj4T7bSySNTWeEm; expires=Wed, 31-05-2020 00:00:00 GMT
```

When your browser notices `Access-Control-Allow-Credentials` set to true. It will
respect `Set-Cookie` header and sets the cookie.

**Important:** "\*" wildcard should not be set in the `Access-Control-Allow-Origin` like
mentioned in the [Credentialed requests and wildcards](#52-credentialed-requests-and-wildcards) section.

## 5. Advanced Topics

### 5.1. Additional HTTP Response Headers

#### 5.1.1. Access-Control-Expose-Headers

When a credentials request is made by setting `withCredentials` flag, `Access-Control-Expose-Headers`
has to be set by server to let browser know which headers can be accessed.

In pre-cors world, by default response headers are not accessible by browser in the CORS request.
So it is made explicit so that browser will look for this header in order to read exposed headers.
This way CORS specification makes sure old browsers doesn't break.

#### 5.1.2. Access-Control-Allow-Credentials

This is returned in the [pre-flight](#44-preflight-requests) requests. When your browser
sees this, it can access `Set-Cookie` header. As we mentioned above, in normal XHR requests, your
browser won't pass `Cookie` in request header as well as read `Set-Cookie` response header.

**Syntax:**

```
Access-Control-Allow-Credentials: true
```

You can find example in the [Request with credentials](#45-request-with-credentials) section.

### 5.2. Credentialed requests and wildcards

Server must specify origin in the value of `Access-Control-Allow-Origin` header,
instead of specifying the "\*" wildcard. Specifying wildcard would fail the request.

### 5.3. Preflighted requests and redirects

When a pre-flight request responds with 301/302, some browsers may not support this
currently. You might get errors like,

`The request was redirected to 'https://example.com/foo', which is disallowed for cross-origin requests that require preflight`

`Request requires preflight, which is disallowed to follow cross-origin redirect`

**Note:** For workarounds check [Preflighted requests and redirects](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) docs by Mozilla.

### 5.4. Third-party cookies

Browser has settings to reject all `third-party` cookies, when a user enables that. For example, if a request is
made from `https://foo.com` and server is at `https://bar.com`, your browser will not set cookies sent by `https://bar.com`.

## 6. TODO

- Add Screencasts

## 7. References

- [Cross-Origin Resource Sharing (CORS) by Mozilla](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)
