---
title: "Understanding Content Security Policy (CSP): How it Works and Why It Matters"
date: 2025-05-07 13:00:00 +0530
categories: [Cyber Security, Web Security]
tags: [Web Security, Content Security Policy, CSP, Cross Site Scripting, XSS, XSS Protection, Web Application Security, HTTP Headers, Cybersecurity, Browser Security, CSP Header, Frontend Security, Web Development Best Practices, kundan dhupkar]
author: KD
comments: true
image: "assets/img/diagrams/writeup_seven/title.png"
toc: true
description: Learn what Content Security Policy (CSP) is, how it works under the hood, and why it’s a critical layer of defense against attacks like Cross-Site Scripting (XSS). This guide breaks down CSP concepts and directives in a practical, easy-to-understand way. 
---

## **Introduction**

One of the most common web application vulnerabilities is Cross-Site Scripting (XSS) which comes under OWASP Top 10 in A3: Injection. There are multiple ways to prevent XSS and one of them is Content Security Policy (CSP).

As cybersecurity professionals, understanding security related HTTP headers is very important and CSP is one of them. In this blog, I’ll explain what CSP is, how it works and why it is a valuable tool in the world of web security. I will try to make it as clear and beginner friendly as possible. So, let’s get started!

---

## **What is Content Security Policy (CSP)?**

Content Security Policy (CSP) is a browser security mechanism which instructs the browser about which resources are allowed to load and execute. It is implemented using HTTP response headers or meta elements. It allows developers to define some set of rules that browsers will enforce to restrict the sources of various type of content(scripts, images, stylesheets etc.). It helps in defending against some web application vulnerabilities like XSS and ClickJacking.


![Content Security Policy (CSP)](assets/img/diagrams/writeup_seven/csp.jpg)

_From imperva.com_

---

## **Purpose of CSP**

Content Security Policy (CSP) aims to mitigate risks from:

1. **Cross-Site Scripting (XSS):** Prevents malicious scripts injected by attackers from executing
2. **Data Injection Attacks:** Limits where data can be loaded from, reducing the risk of loading malicious content. Data in this case can be anything like images, scripts etc.
3. **ClickJacking:** It can restrict other pages from framing a web page
4. **Mixed Content Issues:** It ensures that secure (HTTPS) resources are prioritized on secure pages

---

## **Working of CSP**

Let's now see how CSP works. For this you need to understand some key components of it.

**Key Components:**

1. **Directives:** They defines the rules for specific types of resources.
2. **Sources:** They specify where the resources can be loaded from.
3. **Policies:** These are combinations of directives and sources.

#### **Directives:**

Directives are building blocks of CSP. They specify the type of resources and the allowed sources for it. Some common directives are as follows:

- `default-src`: Specifies the default content sources.
- `script-src`: Specifies the trusted sources for JS files.
- `style-src`: Specifies the trusted sources for CSS files.
- `img-src`: Specifies the trusted sources for images.
- `connect-src`: Used to restrict URLs for XHR, WebSockets and EventSource coonections.
- `font-src`: Specifies trusted sources for fonts.
- `frame-src`: Controls frame/iframe sources (deprecated in CSP3, use child-src).
- `child-src`: Controls the workers and embedded frame contents.
- `worker-src`: Used to control the worker scripts.
- `manifest-src`: Specifies trusted sources for manifest files.
- `prefetch-src`: Used to control prefetched or preloaded resources

and many more...

#### **Sources(Source Values):**

These are the values which are used by directives to define the allowed origins.

- `'self'`: Matches the same origin (protocol, host, and port).
- `'none'`: Blocks all sources for the directive.
- `'unsafe-inline'`: Allows inline scripts or styles (less secure, use cautiously).
- `'unsafe-eval'`: Allows dynamic code evaluation (e.g., eval()), which is risky.
- `https:`: Restricts resources to HTTPS protocol.
- `data:`: Allows data URIs (e.g., for images like `data:image/png;base64,...`).
- Specific domains: Explicit URLs like `https://trusted.com`
- **Nonce:** A cryptographic nonce (`nonce-<base64-value>`) for allowing specific inline scripts/styles. Example: `script-src 'nonce-abc123'`
- **Hash:** A cryptographic hash (`'sha256-<hash>'`) for allowing specific inline scripts/styles. Example: `script-src 'sha256-xyz789'`

---

## **CSP Modes**

Before we move ahead looking for actual implementation of CSP you should know the modes which are there of CSP. CSP has two modes: **Report-Only** and **Enforced**. Here is what each means:
#### **Report-Only Mode:**

- Browser will log the violations but won't block anything.
- It helps in testing the policies without breaking your site.
- Header for this mode is: `Content-Security-Policy-Report-Only`
- Note that this can't be delivered using `<meta>` element.
#### **Enforced Mode:**

- Browser will strictly block all violations
- Header for this mode is: `Content-Security-Policy`

**It is better to test with Report-Only mode first before enforcing CSP.**

---

## **Implementation of CSP**

As I mentioned earlier, CSP can be implemented in two ways:

#### **1. Using HTTP Header (Preferred):**

**Examples:**

Restricting all content to be loaded from the same domain:
```http
Content-Security-Policy: default-src 'self';
```

Allowing scripts from trusted sources and inline styles:
```http
Content-Security-Policy: script-src 'self' https://trusted-cdn.com; style-src 'self' 'unsafe-inline';
```

Frame Ancestors for **Clickjacking Protection** - Prevent embedding except by specific domains:
```http
Content-Security-Policy: frame-ancestors 'self' https://example.com
```

#### **2. Using HTML Meta Tag (Limited, no reporting):**

Example:

Allowing browser to load all resources(by default) only from the same origin and JS scripts only from same origin.
```html
<meta http-equiv="Content-Security-Policy" content="default-src 'self'; script-src 'self'">
```

---

## **Some Advance Features of CSP**

#### **1. Nonce**

HTML:
```html
<script nonce="EDNnf03nceIOfn39fn3e9h3sdfa">
  // Some inline code
</script>
```

CSP header:
```http
Content-Security-Policy: script-src 'nonce-EDNnf03nceIOfn39fn3e9h3sdfa'
```

It allows only inline scripts with a mathcing nonce value to run and prevents attackers from injecting and running their own scripts because they won't know the correct nonce.

#### **2. Hashes**

CSP header:
```http
Content-Security-Policy: script-src 'sha256-abc123...'
```

It allows only specific inline scripts to run based on their hash. The browser will calculate the SHA-256 hash of the inline script and checks if it matches the one in the CSP. If matched then allows the script ro run otherwise blocks.

#### **3. Reporting**

CSP header:
```http
Content-Security-Policy: default-src 'self'; report-uri https://example.com/csp-report
```

It blocks anything that violates the policy and sends the report to the given URL when a policy is violated. It helps in tracking and fixing any CSP issues and also detects any attack attempts.

*Note: report-uri is deprecated, it is better to use report-to in newer CSP versions.*

#### **4. Report-Only Mode**

CSP header:
```http
Content-Security-Policy-Report-Only: default-src 'self'; report-uri https://example.com/csp-report
```

It doesn't block anything and only reports the violations. It is mostly used for testing new policy before implementing it. It helps in avoiding breaking the site and still seeing what would be blocked.

The main difference between Reporting and Report-Only Mode is about whether the CSP is enforced or just monitored.

---

## **Browser Support**

CSP is supported by all modern browsers (Chrome, Firefox, Safari, Edge). Key versions:

- **CSP Level 1**: Basic directives (`script-src`, `style-src`, etc.).
- **CSP Level 2**: Added `frame-ancestors`, `report-uri`, and `nonces`.
- **CSP Level 3**: Introduced `'strict-dynamic'`, `worker-src`, and enhanced reporting.

You can check compatibility on [CanIUse](https://caniuse.com/contentsecuritypolicy) for specifics.

---

## **Common Challenges and Limitations**

- **Inline scripts:** CSP blocks inline scripts by default, which can break many sites. For this you can use `nonce` or `hash` and may need refactoring for inline scripts.
- **Complexity:** Creating a policy for large applications with many resources can be difficult.
- **False Positives:** Overly restrictive policies can block legitimate resources.

---

## **CSP Best Practices**

- **Start with a restrictive policy:** Use `default-src 'none'` and gradually loosen it to allow only necessary sources.
- **Avoid `'unsafe-inline'` and `'unsafe-eval'`:** Use nonces or hashes for inline scripts/styles to minimize XSS risks.
- **Use HTTPS:** Prefer https: sources and enforce it with upgrade-insecure-requests.
- **Test with Report-Only Mode:** Identify issues without breaking functionality.
- **Monitor Violations:** Set up report-uri to collect and analyze violation reports.
- **Use Nonces or Hashes:** For inline scripts/styles, use cryptographic nonces or hashes to avoid `'unsafe-inline'`
- **Regularly Review Policies:** Update policies as your application evolves to include new resources or domains.
- **Combine with Other Headers:** Use CSP alongside headers like `X-Frame-Options` and `X-Content-Type-Options` for layered security.

---

## Final Thoughts

**Content Security Policy (CSP)** is one of the most effective defenses against common web security vulnerabilities like **XSS**. While it requires proper implementation, the security benefits are worth the effort. If you are serious about securing your web applications then understanding and applying CSP is must.

I hope you found this blog helpful. If you notice any mistakes or have suggestions, feel free to reach out. I'm always learning too! For more such content, follow **Security Mates**.

Stay safe and keep learning! 🛡️

{% include comment.html %}
