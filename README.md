# Cross-Origin Request Sharing (CORS)

## Contents
1. [Pre-Requisites](#user-content-pre-requisites)
2. [Why do we need CORS?](#user-content-why-do-we-need-cors?)
3. [Introduction](#user-content-introduction)
4. [Basics](#user-content-basics)
    - [Simple Requests](#user-content-simple-requests)
    - [Preflight Requests](#user-content-preflight-requests)
        - [Request Headers](#user-content-request-headers)
            - [Origin](#user-content-origin)
            - [Access-Control-Request-Method](#user-content-allow-control-request-method)
            - [Access-Control-Request-Headers](#user-content-allow-control-request-headers)
        - [Response Headers](#user-content-response-headers)
            - [Access-Control-Allow-Origin](#user-content-access-control-allow-origin)
            - [Access-Control-Allow-Methods](#user-content-access-control-allow-methods)
            - [Access-Control-Allow-Headers](#user-content-ontrol-allow-headers)
            - [Access-Control-Max-Age](#user-content-access-control-max-age)
            - [Vary](#user-content-vary)
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

- Developer
- [Simple HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview) VS [XHR](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest) request.(Optional)

## Why do we need CORS?

Origin Web App - https://github.com

When you open a website by typing Origin Web App, your browser makes Simple HTTP requests. CORS is not applicable here.
As we are voluntarily requesting to serve content from Origin Web App. Your browser will/should serve it without any
trouble. But, when it comes to XHR requests, JavaScript which is loaded by Origin Web App in your browser can make a
request to any server(https://any.server) requesting for a resource. Now, Origin Web App is owned by Github
but https://any.server is owned by a company named Foo(for an argument), that server could pontentially serve anything.
Github cannot take control of a server which is owned by Foo. This has lot of security implications pontentially stealing a
content from one website(it could be a financial website) to any remote server.

Luckily, CORS tackles this problem very well with a given set of rules.

## Introduction

Its a standard defined by W3C to enable cross-origin requests between client(browser) and the server to share
resources at the same time maintaining security. Any browser will comply with these standards to prevent
loading resources from any third-party servers.

## Basics

### Simple Requests

### Preflight Requests

#### Request Headers

##### Origin

##### Access-Control-Request-Method

##### Access-Control-Request-Headers

#### Response Headers

##### Access-Control-Allow-Origin

##### Access-Control-Allow-Methods

##### Access-Control-Allow-Headers

##### Access-Control-Max-Age

##### Vary

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
