# Web Cache Poisoning – Fundamentals & Cache Key Concepts

## High-Level Summary

This lesson introduces the foundational principles of web cache poisoning by exploring how web caching works across various layers and how attackers can exploit improper cache key configurations. It explains which parts of an HTTP request are used to determine a cache key and how unkeyed inputs—those not considered in cache matching—can be abused to poison the cache. This results in malicious content being served to multiple users, especially when the vulnerable cache is shared (e.g., CDN, proxy). Understanding what gets cached, how it’s keyed, and which headers are evaluated is essential for identifying cache poisoning opportunities.

>[!summary]
> **Web Cache Poisoning (Broken Cache Key) – Simple Explanation**
> 
> When a cache (like a CDN) is misconfigured to ignore query parameters, it only caches based on the URL path (e.g., `/product`).
> 
> If an attacker sends a malicious request like `/product?id=<script>alert(1)</script>`, the full request (with the parameter) is still forwarded to the backend.
> 
> The backend includes the script in the HTML response. The cache stores that full HTML page, but only under `/product` — ignoring the `id`.
> 
> Later, normal users visiting `/product?id=123` get the cached (poisoned) response and see the attacker’s script.
> 
> **Analogy:** It’s like a mailroom told to treat all mail for apartment 101 the same. An attacker sneaks in a fake letter for 101. The mailroom stores it as "official mail" for that apartment. Now everyone who checks that mailbox gets the fake letter — even if it was addressed differently.

---

## Caching Fundamentals

Web caching sits between clients and servers, storing HTTP responses to reduce server load and latency, and to serve repeated requests faster. When a user requests a resource:

1. The cache checks if a fresh (valid) cached version exists.
    
2. If yes, the response is returned directly to the user.
    
3. If not, the request is forwarded to the origin server, and the new response may be cached.


### Common Caching Layers

- **Browser Cache**  
    Stores assets like images, scripts, and pages locally. It's user-specific and doesn't impact other users.
    
- **Proxy Cache**  
    Shared caches in corporate or ISP networks. If poisoned, all users in the network may receive the malicious response.
    
- **CDN Cache**  
    Services like **Cloudflare** and **Akamai** deploy edge caches globally. If compromised, they can deliver a poisoned response to all users retrieving content through that CDN.


> [!warning]  
> Poisoning a CDN cache poses a **high-impact threat** since it affects a large user base across geographies.

- **Application-Level Cache**  
    Caching at the app level—e.g., in-memory or DB-backed—used to store expensive or frequently accessed resources. When shared, it can be a vector for poisoning.


Examples:

- Caching in frameworks or proxies like **Nginx**, **Varnish**, or custom backend logic.


---

## What Gets Cached?

Not all HTTP responses are cached. Several factors determine cacheability:

- **Request Methods**: Usually only `GET` and `HEAD` responses are cached.
    
- **Caching Headers**:
    
    - Cacheable: `Cache-Control: public`, status codes like `200`, `301`, `302`
        
    - Non-cacheable: `Cache-Control: private`, `no-store`, `no-cache`
        
- **Other Influential Headers**:
    
    - `Expires`
        
    - `ETag`
        
    - `Last-Modified`

> [!note]  
> **Dynamic content** is often marked with headers to **prevent caching**, but misconfigurations can lead to sensitive pages being cached inadvertently.

---
## Cache Keys and Unkeyed Inputs

### What is a Cache Key?

A **cache key** is the set of request components a cache uses to identify whether a cached response matches a new request. Common components include:

- **URL Path**
    
- **Query String**
    
- **Protocol** (`http` or `https`)
    
- **Host Header**
    

In advanced configurations, it may also include:

- `Accept-Language`
    
- `Cookie`
    
- Custom headers (if configured via `Vary` or custom logic)
    

### What is an Unkeyed Input?

An **unkeyed input** is any part of the request not included in the cache key. Examples:

- `User-Agent`
    
- `Cookies` (if not keyed)
    
- `POST` bodies
    
- Headers like `X-Forwarded-Host` or `X-User`

> [!tip]  
> If an application uses an unkeyed input to generate a response, but the cache **ignores** it, the input can be **manipulated** to poison the cache. The poisoned response will then be served to any user with a matching cache key.

---
## Cache Poisoning via Unkeyed Inputs

Here’s how the attack works:

1. **Identify** an unkeyed input (e.g., a query param or header not used in the cache key).
    
2. **Manipulate** the input to modify the response (e.g., inject script).
    
3. If the cache **stores** this modified response, it can be **served to all users** with matching key values.

### Example 1: Poisoning via Ignored Query String

Assume:

- Cache key = only Host + URL path
    
- Query string is **not included**

Steps:

```http
# Attacker request
GET /product?id=456 HTTP/1.1
Host: example.com
```

- The attacker controls `id=456` to inject payload.
    
- The cache **does not distinguish** it from:

```http
# Victim request
GET /product?id=123 HTTP/1.1
Host: example.com
```

- Result: victim sees cached (poisoned) content intended for `id=456`.


> [!note]  
> This effectively turns a **reflected input** (like an ID) into a **persistent attack** when stored in cache. For example, stored **XSS** becomes possible.

### Example 2: Poisoning via Unkeyed Header

Assume the cache does **not** include custom headers like:

- `X-Forwarded-Host`
    
- `X-User`


If the backend **uses these headers** (e.g., to build links, inject user info, etc.), then they become dangerous if not in the cache key.

```http
# Attacker request
GET / HTTP/1.1
Host: target.com
X-User: <script>alert(1)</script>
```

- If the backend reflects `X-User` in the response...
    
- And the cache stores it...
    
- The script will be served to every user who accesses `/`.


---

## Tools Mentioned

|Tool/Concept|Description|
|---|---|
|**Nginx**|Reverse proxy that may implement application-level caching|
|**Varnish**|HTTP accelerator used for caching at the edge or application level|
|**Cloudflare**|Popular CDN service with global caching capability|
|**Akamai**|Enterprise CDN with caching features|
|_(Implied)_ **Burp Suite**|For testing, modifying headers, and inspecting cache behaviors|
|_(Implied)_ **Proxy Tools**|Used to simulate network cache poisoning (e.g., mitmproxy)|
