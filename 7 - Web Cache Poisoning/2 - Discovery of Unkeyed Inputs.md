# Web Cache Poisoning – Discovery of Unkeyed Inputs

## High-Level Summary

This section explains how to find **unkeyed inputs**—elements of an HTTP request that influence the server response but are **not included in the cache key**. These inputs are the cornerstone of web cache poisoning attacks, as they allow an attacker to inject malicious content into a cached response, which can then be served to other users. The content walks through practical discovery techniques using manual testing, response inspection, and automated tools like **Param Miner** in Burp Suite. The section ends with a categorized list of common unkeyed headers frequently exploited in real-world cache poisoning attacks.


>[!note]
> **Web Cache Poisoning Timing – Key Concept**
>
> A cache (like a CDN) stores the first **cacheable response** it sees for a given path (e.g., `/news`). If no entry exists (or the old one expired), the next request triggers a **cache MISS** and the new response is stored.
>
> An attacker can exploit this by sending a malicious request with an **unkeyed input** (like a header or query param the cache ignores), causing the backend to generate a poisoned response.
>
> Since the cache doesn't key on that input, it stores the poisoned version under a general path like `/news`, and all future users will receive it — even if they didn’t include that malicious input.
>
> To successfully poison:
> - Be the **first** request after cache expiry (or the first ever)
> - Include malicious input the backend uses but the cache ignores
> - Let the poisoned response get saved and reused for others

---

## Requirements for a Successful Cache Poisoning Attack

To successfully poison a cache, two conditions must be met:

1. **Injectable Response:**  
    You must be able to influence the server response to include malicious or unintended content such as:
    
    - `<script>` tags (XSS)
        
    - Redirects to malicious domains
        
    - Error pages
        
2. **Cache Storage:**  
    The malicious response must be stored by the cache. This may occur if:
    
    - The server sends caching headers like `Cache-Control: public`
        
    - The proxy or CDN is configured to cache it
        

Once both conditions are fulfilled, any user making a request that matches the same **cache key** will receive the poisoned content.

> [!tip]  
> Cache poisoning can lead to **stored XSS**, **open redirects**, or **denial of service**—depending on the payload and application behavior.

---

## Techniques for Finding Unkeyed Inputs

The first step in a cache poisoning attack is identifying inputs that:

- Affect the server’s response
    
- Are **not** included in the cache key
    

### 1. Manual Testing

Manually test headers or parameters by sending variations and observing changes in the response.

#### Example Workflow:

- Use Burp Suite’s **Proxy → HTTP history** to inspect cache behavior.
    
- Monitor the `X-Cache` response header:
    
    - `X-Cache: HIT` → Cached response
        
    - `X-Cache: MISS` → Fresh response
        

```http
GET / HTTP/1.1
Host: localhost:3000
X-Test: 123
```

- If adding `X-Test: 123` does **not** change the response or result in a cache miss, it indicates that `X-Test` is not part of the cache key.
    

> [!note]  
> Manual testing is best done **within Burp Suite** to avoid interference from the **browser cache**, which may obscure your results.

---

### 2. Using Param Miner (Burp Suite)

**Param Miner** is an extension that automates the detection of unkeyed headers.

#### Setup:

- Go to **Burp Suite → Extensions → BApp Store**
    
- Install **Param Miner** by _James Kettle_
    

#### Usage:

1. Go to **Repeater**
    
2. Right-click → `Extensions → Param Miner → Guess headers`
    
3. Start with default settings
    

Param Miner works by injecting a variety of uncommon headers and monitoring for response differences.

> [!warning]  
> Param Miner may not detect all unkeyed inputs in custom or simplistic CTF-style labs. However, it is extremely useful in real-world environments with edge caches like Nginx or CDNs.

---

### 3. Inspecting Server Responses

Carefully analyze HTML and API responses for reflected or dynamic data that may come from:

- Headers
    
- Query parameters
    
- Cookies
    

#### Example Indicators:

- A "Share This Page" link using `X-Forwarded-Host`
    
- Pages showing your `User-Agent`
    
- Dynamic language or theme toggles influenced by cookies or headers
    

Use browser dev tools (`F12`) or inspect page elements to identify poisoned content reflected in the page.

```http
X-Forwarded-Host: test
```

If the application generates links like:

```html
<a href="http://test">Share this page</a>
```

...and this is stored in cache, users clicking that link may be redirected to a phishing site or see injected scripts.

---

## Common Unkeyed Inputs to Test

> [!note]  
> These inputs are **often used by web applications**, but caches may **ignore them** unless explicitly included via `Vary` headers.

### Accept-Language

- Used for localization.
    
- Example (PHP):
    
    ```php
    echo $_SERVER['HTTP_ACCEPT_LANGUAGE'];
    ```
    
- Payload:
    
```http Accept-Language: <script>alert(1)</script>```

### User-Agent

- May appear in footers or analytics pages.
    
- Often ignored by caches.
    
- Useful for injecting malicious strings into dynamically built pages.
    

### X-Forwarded-Host / Host

- Used to construct absolute URLs or perform redirects.
    
- Port or scheme changes (e.g., `https` vs. `http`) may bypass cache keying.
    

### Cookies

- Typically excluded from cacheable responses unless explicitly keyed.
    
- Vulnerabilities include:
    
    - **Authorization bypass**
        
    - **Session hijacking**
        
    - **Private data leakage**
        

### HTTP Method

- Caches may erroneously treat `GET` and `POST` to the same URL as identical.
    
- Some proxies even cache `500` or `404` error pages.
    

---

## Practical Example: Poisoning with X-Forwarded-Host

1. Inject header:
    
    ```
    -Forwarded-Host: test
    ```
    
2. Backend generates:
    
    ```html
    <a href="http://test">Share this page</a>
    ```
    
3. Cache stores this.
    
4. New users receive poisoned link.
    
5. Payload options:
    
    - `javascript://` for XSS
        
    - Redirect to phishing site
        

---

## Summary of Goals

To detect and exploit unkeyed inputs:

- **Look for changes** in HTML or behavior triggered by non-standard headers or parameters.
    
- **Test systematically** with tools like Param Miner.
    
- **Focus on user-controlled inputs** that modify server responses but aren’t part of the caching logic.
    

> [!tip]  
> A common bug bounty technique is using cache poisoning to **elevate self-XSS** to **stored XSS**, dramatically increasing severity and payout.

---

## Tools Mentioned

| Tool / Component               | Purpose                                       |
| ------------------------------ | --------------------------------------------- |
| **Burp Suite**                 | Proxy, HTTP history, request inspection       |
| **Param Miner**                | Automated header fuzzing for unkeyed inputs   |
| **Browser Dev Tools**          | Inspect poisoned elements (e.g., links, HTML) |
| _(Implied)_ **Custom Scripts** | Manual header injection for testing           |
| _(Implied)_ **Fuzzers**        | To test multiple headers/values efficiently   |


>[!note] What `X-Cache: HIT` and `X-Cache: MISS` Mean
>- **`X-Cache: HIT`**  
>     Means the **response was served from the cache** — no request hit the backend server.
>     
> - **`X-Cache: MISS`**  
>     Means the **response came from the origin server** (backend), and **wasn't in the cache** at the time of the request.
>     
> - Why it matters:  
>     You can detect when the cache is **cold (MISS)** vs **hot (HIT)** to **time your attacks or payloads**.
>     
> - Real-world use:  
>     Repeatedly request `/product` and log the `X-Cache` value.  
>     When it turns from `HIT → MISS`, the cache just expired — that's your moment to send your **poisoned request**.
>     



---

> [!note] How Param Miner Helps with Web Cache Poisoning
> 
> - **Discovers unkeyed inputs**  
>     Param Miner sends a variety of unexpected parameters and headers (e.g. `?cb=123`, `X-Forwarded-Host`) to find inputs that **affect the backend but aren’t part of the cache key**.
>     
> - **Identifies poisoning opportunities**  
>     If a response is **different when a parameter is present**, but the cache still treats the URL as the same (`/news`, `/product`), this might let you **inject malicious content into the cache**.
>     
> - **Automates the boring part**  
>     Rather than manually guessing headers or params, Param Miner **systematically tests** a wide set, looking for inconsistencies or cache behavior.
>     
> - Real-world use:  
>     Run Param Miner on a cacheable endpoint like `/news`.  
>     If you find a param like `?debug=true` or a header like `X-Host: attacker.com` that changes the response but is ignored by the cache, that’s a lead.  
>     You can then try poisoning the cache with malicious content using that input.
>     
