---
title: "SameSite vs. Same-Origin: Breaking Down Web Security Confusions and Nuances"
date: 2025-01-07 13:00:00 +0800
categories: [Cyber Security, Web Security]
tags: [web security, same-origin policy, same-site cokies, cookies, browser securtiy, HTTP Headers, CSRF, XSS, web application security, security best practices, kundan dhupkar]
author: KD
comments: true
image: "assets/img/diagrams/writeup_five/title.png"
toc: true
description: This blog simplifies the key differences between SameSite and Same-Origin policies, focusing on their impact on web security and how they prevent common vulnerabilities like CSRF and XSS.
---

## Introduction

In this blog, we will cover three essential concepts related to web security: SameSite, Same-Origin Policy (SOP), and Cross-Origin Resource Sharing (CORS). These are foundational mechanisms that help protect web applications from various attacks and ensure secure interactions between different domains.

---

## SameSite

SameSite is a cookie attribute that serves as a browser security mechanism. It determines when to include a website's cookies in requests originating from other websites. This attribute helps protect against attacks such as Cross-Site Request Forgery (CSRF).

Before diving into how SameSite works, it is crucial to understand the context of "SameSite" and how it is interpreted. 

### Defining "SameSite"

A "site" is defined as the top-level domain (TLD), such as `.com` or `.in`, plus one additional level of the domain name, often referred to as TLD+1. When determining whether a request is SameSite, the URL scheme is also considered.

The [**Public Suffix List**](https://publicsuffix.org/) can be used to determine what constitutes as a "site" or "domain" in the context of SameSite behavior. To check if a request is SameSite, we consider the URL scheme, TLD+1, and TLD.

![Example SameSite Comparisons](assets/img/diagrams/writeup_five/samesite.png)

_From portswigger.net_

### SameSite Attribute Values

The SameSite attribute can take the following values, which control cookie behavior:

1. **SameSite=Strict**
   - Cookies are only sent in requests originating from the same site.
   - Example: A user on `example.com` will have cookies sent only with requests directly from `example.com`. 
   - Cookies are not sent for requests from other sites.

2. **SameSite=Lax**
   - Cookies are sent for GET requests if they are part of a top-level navigation, such as clicking a link.
   - GET requests for cross-site resources (e.g., images, scripts) will not send the cookie unless triggered by top-level navigation.
   - **Default Behavior:** If the SameSite attribute is not specified, this value is applied.

3. **SameSite=None**
   - Cookies are sent with all requests, without restrictions.
  
**Note:** The default behavior of SameSite can vary across different browsers.

### Explicit vs. Default SameSite=Lax

When SameSite=Lax is set by a developer explicitly, its behavior slightly differs from when browsers apply it by default. Let’s explore the specifics:

#### "Safe" vs. "Unsafe" HTTP Methods

- **Safe Methods (GET, HEAD, OPTIONS, TRACE):** These do not modify server data. Cookies with `SameSite=Lax` are included in cross-site requests initiated from top-level navigation.
- **Unsafe Methods (POST, PUT, DELETE, PATCH):** These modify server data. Cookies with `SameSite=Lax` are not sent for cross-site requests using these methods.

#### The SSO Exception: The 2-Minute Time Window for POST Requests

Google Chrome introduced an exception for Single Sign-On (SSO) workflows:

- During a login process, an Identity Provider (IdP) may redirect the user to a Service Provider (SP) using a POST request.
- If cookies required for authentication on the SP domain have `SameSite=Lax`, they would typically not be sent in cross-site POST requests, breaking SSO.

To prevent this, Chrome temporarily allows cookies with default `SameSite=Lax` to be sent in cross-site POST requests within a 2-minute window after a user-initiated navigation.

**Example Workflow:**
1. A user clicks "Sign in with Google" on `example.com`.
2. The user is redirected to `accounts.google.com` for authentication.
3. After logging in, Google redirects the user back to `example.com` using a POST request.
4. Within the 2-minute window, Chrome allows cookies (with default `SameSite=Lax`) to be sent to `example.com` for authentication.

### Preventing Web Attacks

Attacks like CSRF rely on initiating requests from an attacker's site to the target site. Properly configuring the SameSite attribute ensures that cookies are not sent with cross-site POST requests, preventing such attacks.

---

## Same-Origin

The Same-Origin Policy (SOP) is a browser-enforced security measure that restricts how documents and scripts from one origin interact with resources from another origin. This prevents malicious websites from accessing sensitive data on another site.

---

Let's first understand how origin is defined.

### Defining "Origin"

An origin consists of three parts:
1. URI Scheme
2. Host
3. Port

Two URLs share the same origin if all three components match exactly.

#### Example Origin Comparisons:
![Example Origin Comparisons](assets/img/diagrams/writeup_five/origin.png)

_From MDN Web Docs_

### SOP in Action

People often get confused about how it works so I will try to keep it as simple as possible.

Note that SOP is Browser-Enforced, Not Server-Enforced. 

SOP says that you can't read the response of a request which has different origin.

It's important to understand that when SOP is enforced the response can't be read. Many people thought that the request gets blocked but its not like that. For example when SOP is enforced, if a.com sends a request to b.com then following will happen:

- A request from a.com will be sent to b.com
- b.com will respond according to the request received.
- Browser will check the origin of the request and response
- As it is not same-origin but cross-origin, browser will not allow to read the response from b.com
(So it gets blocked on browser level)

### Preventing Web Attacks

The Same-Origin Policy (SOP) is effective in preventing Cross-Site Scripting (XSS) attacks, as it restricts malicious scripts from accessing sensitive data or resources from different origins. However, SOP does not prevent Cross-Site Request Forgery (CSRF) attacks. In CSRF, the attacker does not need to read the response from the server, but instead exploits the user's authenticated session to perform unintended actions. Therefore, while SOP is crucial for XSS prevention, additional measures such as anti-CSRF tokens are necessary to defend against CSRF attacks.

---

#### Exceptions to SOP
There are some exceptions to SOP. One of them is **CORS (Cross-Origin Resource Sharing)**.

---

## CORS (Cross-Origin Resource Sharing)

As we saw, SOP is very restrictive and doesn't allow to read responses from different origin. 

However, in modern web applications, there are many legitimate cases for cross-origin requests, such as:

- Using APIs hosted on a different domain.
- Loading resources (images, fonts, scripts) from a Content Delivery Network (CDN).
- Integrating with third-party services like payment gateways or OAuth providers. 

In such scenarios it can make it difficult to manage the proper functioning of web application.

CORS is a mechanism that allows servers to specify which origins can access their resources when SOP would otherwise block the response. This enables legitimate cross-origin interactions in modern web applications.

### How CORS Works

1. **Preflight Requests:** For certain types of cross-origin requests, the browser sends an HTTP OPTIONS request to the server before the actual request. This is called a **preflight request**.
   - The server responds with headers indicating whether the request is allowed.

2. **CORS Headers:** When the server receives a cross-origin request, it includes specific headers in its response, such as:
   - `Access-Control-Allow-Origin`: Specifies permitted origins.
   - `Access-Control-Allow-Methods`: Lists allowed HTTP methods.
   - `Access-Control-Allow-Headers`: Specifies allowed custom headers.
   - `Access-Control-Allow-Credentials`: Indicates if credentials (e.g., cookies) can be sent.

3. **Response Validation:**
   - If the server's response contains valid CORS headers, the browser allows access to the response. If the headers are missing or incorrect, the browser blocks the response, enforcing SOP.

### Example of CORS in Action

**Scenario:**
- `frontend.example.com` (frontend) needs to make an API call to `api.example.com` (backend).

**Without CORS:**
1. The browser sends a request to api.example.com.
2. The browser sees that this is a cross-origin request.
3. Because no CORS headers are returned, the browser blocks the response.

**With CORS:**
1. The browser sends a request to api.example.com
2. The server responds with:
   ```
   Access-Control-Allow-Origin: https://frontend.example.com
   Access-Control-Allow-Methods: GET, POST
   ```
3. The browser validates the headers and allows access to the response.

CORS is a critical mechanism for enabling controlled cross-origin resource sharing in modern web applications. It allows servers to define specific rules for which origins can access their resources while ensuring SOP is not violated. When used properly, CORS enables seamless integration between frontend and backend systems hosted on different domains.

---

## Final Thoughts

I hope this blog helped you understand the concepts of SameSite, Same-Origin Policy, and CORS. These mechanisms play a critical role in securing web applications and managing cross-origin interactions. 

If you find any mistakes or have any suggestions, please feel free to provide feedback. I'm always open to learning and improving. Stay tuned for more insightful blogs on cybersecurity concepts. Follow us for updates!

---

{% include comment.html %}
