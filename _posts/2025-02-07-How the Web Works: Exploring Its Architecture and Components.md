---
title: "How the Web Works: Exploring Its Architecture and Components"
date: 2025-02-07 13:00:00 +0800
categories: [Cyber Security, Web Security]
tags: [Web Security, Web Architecture, HTTP/HTTPS, Client-Server Model, DNS, CDN, Web Components, Web Technologies, APIs, Web Fundamentals, Web Servers, Browsers, Web Requests, HTTP Status Codes, Cookies, Sessions, DNS Resolution, Load Balancers, Reverse Proxies, Static vs Dynamic Websites, Web Applications, Content Delivery Networks (CDN), kundan dhupkar]
author: KD
comments: true
image: "assets/img/diagrams/writeup_six/title.png"
toc: true
description: This blog provides a deep dive into the web request flow, explaining each step from the browser to the server and the various components involved. It highlights key elements such as DNS resolution, proxies, load balancing, CDNs, and WAFs, while addressing potential challenges and solutions for building a secure and efficient web infrastructure.
---

# Understanding Web Architecture: What Happens When You Enter a URL?

Most of us interact with the web daily, but as a cybersecurity professional or someone interested in web security, it's important to understand how the web fundamentally works.  

Do you know what happens when you type a URL and press enter?  

Many people have a basic idea of the client-server architecture and may know that the browser sends a request and the server responds. Some might also be aware of a few additional steps like DNS resolution or requests passing through firewalls. However, a web request isn’t just a simple exchange between a browser and a server—it travels through multiple layers, each with its own tasks and responsibilities.  

Understanding these components is crucial from a web security perspective because each layer introduces potential attack vectors.  

In this blog, I’ll cover most of these components to give you a basic idea about different elements of web architecture, their working, and how they impact web performance and reliability.

---

## The First Step: Opening the Browser

Let's start from the beginning. What’s the first thing a user does when they want to access the web?  
They open a **browser**.

### Browser

A browser is a software application used to access, retrieve, and display content from the internet.  

When you type a URL or click a link, the browser sends a request to the server hosting that website. Once it receives data from the server, it renders the content (HTML, CSS, JavaScript) into a readable format for the user.  

Some of the main components of a browser include:  

- **User Interface (UI):** The visible part (address bar, back/forward buttons, etc.)
- **Rendering Engine:** Converts HTML and CSS into a visual display.
- **JavaScript Engine:** Executes JavaScript code to make websites interactive.
- **Browser Storage:** Stores cached files, cookies, and session data.  

Now that we understand what a browser does, let's dive into what happens when you enter a URL in the address bar.

---

## What Happens When You Enter a URL?

### 1. URL Parsing

The browser first parses the **Uniform Resource Locator (URL)** you entered. It breaks it into components such as:

- **Protocol:** `http://` or `https://`
- **Domain Name:** `example.com`
- **Path:** `/index.html`
- **Query Parameters:** `?id=123`

### 2. DNS Resolution

The browser needs to find the **IP address** of the server hosting the website. It does this by querying a **Domain Name System (DNS) server**.  

DNS is responsible for translating domain names into corresponding IP addresses.  

- If the browser **has the IP address cached**, it skips this step.
- If not, it sends a request to a **DNS server** to resolve the domain name (e.g., `example.com`) into an IP address.  

There are several types of DNS servers that play different roles in resolving domain names into IP addresses. Here are the main types:

1. Recursive Resolver (Recursive DNS Server)
- It receives the query from the client (e.g., your browser), and if it doesn't have the result cached, it will make additional queries to other DNS servers to fully resolve the domain name.

2. Authoritative Nameserver
- These servers hold the DNS records for specific domains and respond with the actual IP addresses for those domains. They provide the definitive answer to a DNS query.

3. TLD Nameserver (Top-Level Domain Nameserver)
- These servers manage domain extensions like .com, .org, .net, etc. They point to the authoritative nameservers for domains under their respective TLD.

4. Root Nameserver
- The root nameservers are at the highest level of the DNS hierarchy. They do not contain specific domain records but direct DNS queries to the appropriate TLD nameservers.

![DNS Resolution](assets/img/diagrams/writeup_three/dns_resolution.png)

_From researchgate.net_

Once the browser obtains the IP address, it establishes a connection with the **web server**.

---

## Web Server

A **web server** is a software or hardware system that serves web content to users over the internet. It handles requests from clients (typically browsers) and delivers web pages or other resources (like images, scripts, etc.)

### How It Works:
1. The browser sends an **HTTP/HTTPS request** to the web server.
2. The web server **processes the request** and determines which file (HTML, CSS, JavaScript, etc.) to send based on the URL path.
3. The server **responds** by sending the requested content.
4. The browser receives the content and **renders** the webpage.

If the content is **static**, the web server serves it directly. However, if the content is **dynamic**, the web server forwards the request to an **application server**.

---

## Application Server

The **application server** handles backend processing. It contains the business logic required to process requests, such as:

- Interacting with databases
- Executing server-side scripts

Some dynamic requests may require data, in that case **application server** may need to interact with the **database server**.

### Request Flow So Far:
1. **User enters a URL**
2. **Browser parses URL**
3. **DNS Resolution**
4. **Browser establishes a connection & sends request to the web server**
5. **For dynamic content → Request forwarded to application server**
6. **If required → Database request**
7. **Response sent back to web server**
8. **Web server responds to the browser**

---

## Addressing Potential Concerns

### Problem: Directly Exposing the Web Server to the Internet

Exposing a web server directly to the internet is risky. Attackers can launch **Denial of Service (DoS) / Distributed Denial of Service (DDoS) attacks**, overload the server, or exploit vulnerabilities.  

**Solution:** Use a **Reverse Proxy**.

### Proxy

A **proxy** is an intermediary that relays requests between two parties. It can be used to hide identities, filter traffic, cache content, distribute load, or improve security. There are two main types:

1. **Forward Proxy**  
   - Sits between clients and the internet.
   - Hides the client's identity.

2. **Reverse Proxy**  
   - Sits between clients and backend servers.
   - Protects backend servers from direct exposure.

### How Reverse Proxy Works:
1. User requests `example.com`
2. Request goes to the **reverse proxy** instead of the web server.
3. Reverse proxy **forwards the request** to the appropriate backend server.
4. Backend server processes the request and responds.
5. Reverse proxy sends the response to the client.

![Reverse Proxy](assets/img/diagrams/writeup_three/reverse_proxy.png)

_From imperva.com_

This prevents direct exposure of the server to the internet.

---

Although, despite having multiple backend servers, if all requests go to one server, it may slow down or crash under high load.

### Problem: High Server Load and Slow Performance  

**Solution:** Use a **Load Balancer (LB)**.

## Load Balancer

A **Load Balancer** distributes incoming traffic across multiple backend servers to:

- Prevent server overload
- Improve performance & reliability
- Ensure high availability
- Handle traffic spikes efficiently

The Load Balancer decides which server should handle a request using different algorithms like:
- **Round Robin**
- **Least Connections**
- **IP Hash**
- **Weighted Round Robin**
- **Least Response Time**  

![Load Balancer](assets/img/diagrams/writeup_three/load_balancer.png)

_From geeksforgeeks.org_

There are also multiple types of Load Balancer based on deployment (Hardware, Software, Cloud-based) and based on load balancing method (Layer 4, Layer 7)

A **reverse proxy** can also act as a **load balancer** because it already sits between clients and backend servers, and it can be configured to distribute traffic. A load balancer can also act as a reverse proxy, but only if it's an application-layer (Layer 7) load balancer. This solves the problem of high server load & slow performance.

---

When a website is hosted far from the user, data takes longer to travel, causing latency. This is especially noticeable for example, when the website is hosted in the US but the user is in India.


### Problem: High **latency** when a user is geographically distant from the web server.

Latency increases when:

- Geographical Distance → Data takes longer to travel between the user and the origin server.
- Network Congestion → High traffic causes delays in data transmission.
- Server Load → A single origin server handling all requests may become slow.

**Solution:** Use a **Content Delivery Network (CDN)**.

## Content Delivery Network (CDN)

A **CDN** is a network of **edge servers** that cache and serve content closer to users. This reduces:

- **Geographical latency**
- **Network congestion**
- **Server load**

![Content Delivery Network(CDN)](assets/img/diagrams/writeup_three/cdn.png)

_From cloudns.net_

CDNs basically stores static assets (HTML, CSS, JavaScript, images, videos) on multiple servers worldwide. Users get content from the nearest server, reducing response time.

Some of the popular CDNs are **Cloudflare, Akamai, AWS Cloudfront** etc.

Many CDNs (Content Delivery Networks) also function as a WAF (Web Application Firewall) by providing security features alongside content delivery.

---

## Web Application Firewall (WAF)

A **Web Application Firewall (WAF)** is a security solution that monitors, filters, and blocks malicious HTTP/S traffic to and from a web application. It protects against web-based attacks like **SQL Injection**, **Cross-Site Scripting (XSS)** and more by analyzing HTTP requests before they reach the web application.

A WAF inspects incoming HTTP/S requests and outgoing responses based on predefined security rules (policies). If a request matches a malicious pattern, the WAF can:
- Block the request (if it's an attack).
- Allow the request (if it's legitimate).
- Log and monitor suspicious traffic for analysis.

Filtering Process:
- User Sends an HTTP Request : A client (browser, API, mobile app) sends a request to a web server.
- WAF Intercepts & Analyzes : The WAF examines request headers, body, and parameters.
- WAF Applies Rules : Based on pre-configured rules (signature-based, behavior-based, etc.), WAF determines if the request is safe.
- Decision is Made : The request is either blocked, allowed, or flagged for review.
- Web Server Processes Safe Requests : If allowed, the request reaches the web server as usual.

![Web Application Firewall(WAF)](assets/img/diagrams/writeup_three/waf.png)

_From indusface.com_

Popular WAFs include **Cloudflare WAF, AWS WAF, Imperva WAF etc.**


---

## Ideal Web Request Flow

When users make a request to a website, a well-organized flow ensures that the request is processed efficiently, securely, and quickly. This flow depends on various components working together to handle traffic, protect the servers, and deliver content in the most optimal way.

When all components are in place, an ideal request flow looks like:

1. User Request → CDN
- If cache available → Responds immediately (request does NOT go forward).
- If cache NOT available → Request goes to WAF.

2. CDN → WAF
- If malicious request → Blocked/logged (request does NOT go forward).
- If clean request → Forwarded to Reverse Proxy.

3. WAF → Reverse Proxy
- If Reverse Proxy is also a Load Balancer → Directs traffic to the appropriate App Server.
- If Reverse Proxy is separate → Passes request to Load Balancer.

4. Reverse Proxy → Load Balancer
- If Load Balancer is configured → Distributes traffic based on load, region, or health checks.
- If Reverse Proxy is acting as Load Balancer → It handles distribution itself.

5. Load Balancer → Web Server (Optional)
- If Web Server is separate → Static content is served, dynamic requests go to App Server.
- If no separate Web Server → Load Balancer directly forwards to App Server.

6. Web Server → Application Server
- If request needs dynamic processing → Passed to App Server.
- If request is for static content (and Web Server is present) → Served directly.

7. Application Server → Database Server
- If data is required → Query is sent to DB Server.
- If no DB interaction needed → Response generated at App Server.

Response Flow (Reverse Process)
1. DB Server → Application Server (Response with Data)
2. App Server → Load Balancer (Processed Response Sent)
3. Load Balancer → Reverse Proxy
4. Reverse Proxy → WAF (Security Check, Logging)
5. WAF → CDN (Edge Caching, Optimization)
6. CDN → User (Final Response Served)

---

## Final Thoughts

This blog covered key components of web architecture and how they impact security. There are additional topics like **Microservices, Containerization, and Cloud Infrastructure**, which I’ll cover in another blog.  

If you find any mistakes or have suggestions, feel free to provide feedback. I’m always open to learning and improving. Stay tuned for more insightful cybersecurity blogs. Follow us for updates! 🚀  

{% include comment.html %}
