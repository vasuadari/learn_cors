# Cross-Origin Resource Sharing (CORS)

## Contents

0. [Summary](#summary)
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
6. [Contributing](#contributing)
7. [License](#license)
8. [TODO](#todo)
9. [References](#references)

## 0. Summary

*   CORS (Cross-Origin Resource Sharing) is a security mechanism that allows or restricts cross-origin HTTP requests initiated from scripts.
*   It works by adding HTTP headers (e.g., `Origin`, `Access-Control-Allow-Origin`) that let servers describe which origins are permitted to read information from a web browser.
*   Understanding CORS is crucial for web developers to build secure and functional applications that interact with resources from different domains, preventing common web security vulnerabilities.

## 1. Pre-Requisites

Know difference between [Simple HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview) and [XHR](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest) request.

## 2. Why do we need CORS?

Imagine a web application loaded from `https://domain-a.com`.

When you open this website, your browser makes standard HTTP requests to `https://domain-a.com` to fetch its content (HTML, CSS, JavaScript, images). CORS is not applicable here, as these are same-origin requests.

However, JavaScript loaded by `https://domain-a.com` might want to make an XHR (XMLHttpRequest) or Fetch API request to a different server, say `https://api.domain-b.com`, to get some data. This is a cross-origin request.
Without CORS, such requests would be blocked by the browser's same-origin policy by default. This policy is a crucial security feature that prevents malicious scripts on one site from accessing sensitive data on another site.

CORS provides a controlled mechanism for servers (like `https://api.domain-b.com`) to tell browsers that it's okay to allow requests from certain other origins (like `https://domain-a.com`). This prevents unauthorized cross-origin access while enabling legitimate use cases. For example, without CORS, `https://domain-a.com` couldn't securely request data from `https://api.domain-b.com`, potentially limiting functionality or forcing less secure workarounds.

CORS helps prevent scenarios like a script on a malicious website trying to steal data by making requests to your bank's website in the background. It ensures that `https://api.domain-b.com` explicitly permits requests from `https://domain-a.com` before the browser allows the script on `https://domain-a.com` to access the response.

## 3. Introduction

Cross-Origin Resource Sharing (CORS) is a standard defined by the [W3C (World Wide Web Consortium)](https://en.wikipedia.org/wiki/World_Wide_Web_Consortium).
Its primary purpose is to enable controlled cross-origin requests between a client (typically a web browser) and a server.

This mechanism allows servers to specify which other origins (domains, schemes, or ports) are permitted to access their resources, thereby maintaining security. Modern web browsers comply with CORS standards to prevent unauthorized loading of resources from third-party servers, which could otherwise lead to security vulnerabilities.

## 4. Basics

### 4.1. Request Headers

#### 4.1.1. Origin

The `Origin` request header indicates the origin (scheme, hostname, and port) from which the cross-origin request is initiated.
This header is automatically added by the browser for all CORS requests, as well as for POST requests.

**Syntax:**

```http
Origin: <scheme> "://" <hostname> [":" <port> ]
```

**Examples:**

```http
Origin: https://netflix.com
Origin: http://netflix.com:443
Origin: http://localhost:1443
```

#### 4.1.2. Access-Control-Request-Method

This header is used by browser when issuing a [pre-flight](#44-preflight-requests) requests to indicate which
request method will be used when the actual request is made.

**Syntax:**

```http
Access-Control-Request-Method: <method>
```

`<method>` could be common HTTP methods like `GET`, `POST`, `PUT`, or `DELETE`.

**Example:**

```http
Access-Control-Request-Method: POST
```

#### 4.1.3. Access-Control-Request-Headers

This header is again used in the [pre-flight](#44-preflight-requests) requests by browser to indicate which
request headers are to be used when the actual request is made. To use multiple headers,
it has to be separated with comma.

**Syntax:**

```http
Access-Control-Request-Headers: <header-name>
Access-Control-Request-Headers: <header-name>, <header-name>
```

**Example:**

```http
Access-Control-Request-Headers: X-PINGOTHER, Content-Type
```

**Important:** All the headers listed in `Access-Control-Request-Headers` must be [CORS-safelisted](#414-cors-safelisted) (e.g., `Content-Type`) or custom headers (e.g., `X-Custom-Header`).

#### 4.1.4. CORS-safelisted

- `Accept`

- `Accept-Language`

- `Content-Language`

- `Content-Type`

  Allowed values are `application/x-www-form-urlencoded`, `multipart/form-data` and
  `text/plain`.

### 4.2. Response Headers

The following headers are returned in the [pre-flight](#44-preflight-requests) requests.

#### 4.2.1. Access-Control-Allow-Origin

The `Access-Control-Allow-Origin` response header is crucial for CORS. It indicates whether the response can be shared with the requesting code from the given [Origin](#411-origin).
Your browser will compare the value of this header with the requesting origin and will allow or deny access to the response based on this comparison.

**Syntax:**

```http
Access-Control-Allow-Origin: *
```
For requests *without* [credentials](#45-request-with-credentials), the server can respond with `*` as a wildcard. This tells the browser to allow requests from any origin.

```http
Access-Control-Allow-Origin: <origin>
```
Alternatively, the server can specify a single origin. This means only that particular origin is allowed to access the resource.
If the server supports a list of allowed origins, it should dynamically set this header value based on the `Origin` header in the request. In such cases, the server should also include the `Vary: Origin` HTTP header to indicate that the response differs based on the request's `Origin`.

**Example:**

```http
Access-Control-Allow-Origin: https://developer.mozilla.org
```

#### 4.2.2. Access-Control-Allow-Methods

Your browser will make actual request if one of the value in this header matches.
When wildcard `*` is returned, it means any method is allowed (though this is not recommended for credentialed requests).

**Syntax:**

```http
Access-Control-Allow-Methods: <method>, <method>, ...
Access-Control-Allow-Methods: *
```

**Example:**

```http
Access-Control-Allow-Methods: GET, POST, OPTIONS
```

#### 4.2.3. Access-Control-Allow-Headers

You browser will make actual request if all of the requested headers are allowed.

**Syntax:**

```http
Access-Control-Allow-Headers: <header-name>, <header-name>
Access-Control-Allow-Headers: *
```

**Example:**

```http
Access-Control-Allow-Headers: Content-Type, X-Custom-Header
```

The wildcard `*` tells the browser that any header is allowed (though this is not recommended for credentialed requests).

#### 4.2.4. Access-Control-Max-Age

Tells how long the results of [pre-flight](#44-preflight-requests) can
be cached.

**Syntax:**
```http
Access-Control-Max-Age: <delta-seconds>
```

`<delta-seconds>` is the maximum number of seconds the results can be cached.

Each browser has max limit,
- Firefox caps this at 24 hours(86400 seconds).
- Chromium (prior to v76) caps at 10 minutes (600 seconds).
- Chromium (starting in v76) caps at 2 hours (7200 seconds). Chromium-based browsers also specify a default value of 5 seconds if the header is not present.
- A value of -1 will disable caching, requiring a preflight OPTIONS check for all calls.

#### 4.2.5. Vary

The `Vary` HTTP response header is generally used for caching purposes. It indicates the set of request headers that determines whether a cached response can be used, or if a new response needs to be fetched from the origin server.

In the context of CORS, if a server sends varying `Access-Control-Allow-Origin` values (e.g., allowing multiple specific origins based on the request's `Origin` header), it **must** include `Vary: Origin` in its response. This informs caches that the response is dependent on the `Origin` request header.

**Syntax:**

```http
Vary: <header-name>, <header-name>, ...
```

**Example in CORS:**

Consider a server `api.example.com` that allows requests from both `https://site-a.com` and `https://site-b.com`.

**Scenario 1: Request from `site-a.com`**

A preflight request from `https://site-a.com`:
```bash
curl -X OPTIONS https://api.example.com/data \
  -H "Origin: https://site-a.com" \
  -H "Access-Control-Request-Method: GET" \
  -H "Access-Control-Request-Headers: Content-Type"
```

Server response:
```http
HTTP/1.1 204 No Content
Access-Control-Allow-Origin: https://site-a.com
Vary: Origin
Access-Control-Allow-Methods: GET, POST, OPTIONS
Access-Control-Allow-Headers: Content-Type
Access-Control-Max-Age: 600
```
The browser caches this preflight result for `https://site-a.com` for 10 minutes.

**Scenario 2: Request from `site-b.com`**

A preflight request from `https://site-b.com`:
```bash
curl -X OPTIONS https://api.example.com/data \
  -H "Origin: https://site-b.com" \
  -H "Access-Control-Request-Method: GET" \
  -H "Access-Control-Request-Headers: Content-Type"
```

Server response:
```http
HTTP/1.1 204 No Content
Access-Control-Allow-Origin: https://site-b.com
Vary: Origin
Access-Control-Allow-Methods: GET, POST, OPTIONS
Access-Control-Allow-Headers: Content-Type
Access-Control-Max-Age: 300
```
Notice how `Access-Control-Allow-Origin` reflects `https://site-b.com`, and the `Access-Control-Max-Age` is different (5 minutes). The `Vary: Origin` header ensures that caches (including browser caches) correctly store and serve these different responses based on the `Origin` of the request.

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
```http
GET /api/v1/public-data/ HTTP/1.1
Host: bar.com
User-Agent: curl/7.69.1
Accept: */*
Origin: https://foo.com
```

*Response Headers:*
```http
HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
Connection: Keep-Alive
Transfer-Encoding: chunked
Content-Type: application/xml
```

When a request returns `*` in `Access-Control-Allow-Origin` header. It means a request
can be made from any `Host`. In this case, your browser won't make pre-flight request.

### 4.4. Preflight Requests

A "preflight" request is a CORS mechanism where the browser sends an initial HTTP request using the `OPTIONS` method to the server hosting the cross-origin resource.
This is done to determine if the actual request (e.g., a `POST` request with custom headers) is safe to send.
Preflight requests are triggered for requests that are not "simple requests" – for example, if they use methods other than GET/POST/HEAD, or if they include custom headers or non-standard `Content-Type` values.

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

```http
OPTIONS /api/v1/post-here/ HTTP/1.1
Host: bar.com
User-Agent: curl/7.69.1
Accept: */*
Origin: https://foo.com
Access-Control-Request-Method: POST
Access-Control-Request-Headers: X-PINGOTHER, Content-Type
```

Server's response to the preflight request:
```http
HTTP/1.1 204 No Content
Connection: Keep-Alive
Content-Type: application/json
Access-Control-Allow-Origin: https://foo.com
Access-Control-Allow-Methods: GET, POST, OPTIONS
Access-Control-Allow-Headers: X-PINGOTHER, Content-Type, Accept
Access-Control-Max-Age: 86400
```

*Actual Request (sent after successful preflight):*

```http
POST /api/v1/post-here/ HTTP/1.1
Host: bar.com
User-Agent: curl/7.69.1
Accept: */*
Origin: https://foo.com
X-PINGOTHER: pingpong
Content-Type: application/json;charset=UTF-8

{ "email": "foo@bar.com" }
```

Server's response to the actual request:
```http
HTTP/1.1 200 OK
Connection: Keep-Alive
Content-Type: application/json
Access-Control-Allow-Origin: https://foo.com

{ "status": "success" }
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

```http
GET /api/v1/credentialed-content/ HTTP/1.1
Host: bar.com
User-Agent: curl/7.69.1
Accept: */*
Origin: https://foo.com
Cookie: _session=NyV14oKXiS6HHopaf9nT
```

When a request is made using `XMLHttpRequest` and the `withCredentials` flag is set to `true` (or using `fetch` with `credentials: 'include'`), the browser will include any relevant cookies for the target domain (`bar.com` in this case) in the request header.

*Response:*
```http
HTTP/1.1 200 OK
Connection: Keep-Alive
Content-Type: application/json
Access-Control-Allow-Origin: https://foo.com
Access-Control-Allow-Credentials: true
Cache-Control: no-cache
Pragma: no-cache
Set-Cookie: _session=AjBSqxj4T7bSySNTWeEm; expires=Wed, 31-May-2025 00:00:00 GMT

{ "data": "sensitive user data" }
```

When your browser notices `Access-Control-Allow-Credentials` set to true. It will
respect `Set-Cookie` header and sets the cookie.

**Important:** When responding to a credentialed request, the server must specify an actual origin in the `Access-Control-Allow-Origin` header, not the wildcard `*`. Using `*` in this context will cause the browser to block the response. This is detailed further in the [Credentialed requests and wildcards](#52-credentialed-requests-and-wildcards) section.

## 5. Advanced Topics

### 5.1. Additional HTTP Response Headers

#### 5.1.1. Access-Control-Expose-Headers

The `Access-Control-Expose-Headers` response header allows a server to indicate which of its response headers (other than the [CORS-safelisted response headers](https://fetch.spec.whatwg.org/#cors-safelisted-response-header-name)) should be made accessible to client-side scripts.

By default, only CORS-safelisted response headers are exposed. If your client-side code needs to access other headers (e.g., a custom `X-Session-Token` header), the server must include them in the `Access-Control-Expose-Headers` value.

**Syntax:**
```http
Access-Control-Expose-Headers: <header-name>, <header-name>, ...
```

**Example:**
```http
Access-Control-Expose-Headers: Content-Length, X-Kuma-Revision
```
This allows the client-side JavaScript to access the `Content-Length` and `X-Kuma-Revision` headers from the response.
This is particularly relevant for credentialed requests where sensitive information might be part of headers.
In the pre-CORS era, most response headers were not programmatically accessible by the browser in a cross-origin context. Making this explicit ensures that older browser behaviors are not broken and that access to potentially sensitive headers is granted deliberately by the server.

#### 5.1.2. Access-Control-Allow-Credentials

This is returned in the [pre-flight](#44-preflight-requests) requests. When your browser
sees this, it can access the `Set-Cookie` header (if present) and process cookies sent by the server.
As mentioned earlier, in normal XHR/Fetch requests without credentials, the browser typically doesn't send cookies in the request header for cross-origin requests, nor does it process `Set-Cookie` headers from cross-origin responses.

**Syntax:**

```http
Access-Control-Allow-Credentials: true
```

You can find example in the [Request with credentials](#45-request-with-credentials) section.

### 5.2. Credentialed requests and wildcards

When a request is "credentialed" (meaning it involves cookies, authorization headers, or TLS client certificates), there are stricter rules regarding the use of wildcards (`*`) in CORS response headers.

Specifically, if a request is made with credentials (e.g., `xhr.withCredentials = true` or `fetch({ credentials: 'include' })`):
*   The server **must not** use the wildcard `*` for `Access-Control-Allow-Origin`. It must specify the exact origin that is allowed (e.g., `Access-Control-Allow-Origin: https://domain-a.com`).
*   Similarly, wildcards should generally be avoided for `Access-Control-Allow-Headers` and `Access-Control-Allow-Methods` in credentialed scenarios for tighter security, though the primary restriction is on `Access-Control-Allow-Origin`.

If a credentialed request receives a response with `Access-Control-Allow-Origin: *`, the browser will block the response for security reasons. This prevents overly permissive configurations where any site could make credentialed requests to an endpoint that reflects the wildcard origin.
The server must explicitly state which origin(s) are permitted to make credentialed requests and access the corresponding responses.

### 5.3. Preflighted requests and redirects

When a pre-flight request responds with 301/302, some browsers may not support this
currently. You might get errors like,

`The request was redirected to 'https://example.com/foo', which is disallowed for cross-origin requests that require preflight`

`Request requires preflight, which is disallowed to follow cross-origin redirect`

**Note:** For workarounds check [Preflighted requests and redirects](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) docs by Mozilla.

### 5.4. Third-party cookies

Browser has settings to reject all `third-party` cookies, when a user enables that. For example, if a request is
made from `https://foo.com` and server is at `https://bar.com`, your browser will not set cookies sent by `https://bar.com`.

## Contributing

Contributions are welcome! If you have improvements or additions, please follow these steps:

1. Fork the repository.
2. Create a new branch (`git checkout -b feature/your-feature-name`).
3. Make your changes.
4. Commit your changes (`git commit -am 'Add some feature'`).
5. Push to the branch (`git push origin feature/your-feature-name`).
6. Create a new Pull Request.

## License

This project is licensed under the MIT License. See the [LICENSE.md](LICENSE.md) file for details.

## 6. Contributing

Contributions are welcome! If you have improvements or additions, please follow these steps:

1. Fork the repository.
2. Create a new branch (`git checkout -b feature/your-feature-name`).
3. Make your changes.
4. Commit your changes (`git commit -am 'Add some feature'`).
5. Push to the branch (`git push origin feature/your-feature-name`).
6. Create a new Pull Request.

## 7. License

This project is licensed under the MIT License. See the [LICENSE.md](LICENSE.md) file for details.

## 8. TODO

- Consider adding diagrams to illustrate complex CORS flows (e.g., preflight requests, requests with credentials).

## 9. References

- [Cross-Origin Resource Sharing (CORS) by Mozilla](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)
- [W3C - World Wide Web Consortium](https://en.wikipedia.org/wiki/World_Wide_Web_Consortium)
