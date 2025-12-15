# Web Cache Poisoning – Lab: Unkeyed Header (Nginx + Docker)

## High-Level Summary

This lab demonstrates how to exploit an **unkeyed header** in a real-world-like setup using **Docker**, **Nginx**, and a demo application. The focus is on identifying an input (e.g., `Accept-Language`) that affects the application output but is not included in the **cache key**, making it exploitable via **web cache poisoning**. The lab involves confirming cache configuration, testing reflection, using **Param Miner**, manually poisoning the cache, and verifying with an incognito browser. The lab ends with a review of mitigation strategies to prevent this class of vulnerabilities.

---

## Lab Setup

### Start the Lab Environment

1. Navigate to the lab directory:
    
    ```bash
    cd ~/labs/unkeyed-header-lab
    ```
    
2. Extract files:
    
    ```bash
    tar -xf lab.tar
    cd lab/
    ```
    
3. Start the container:
    
    ```bash
    sudo docker-compose up
    ```
    
    > [!note]  
    > The first run will pull required Docker images. Future launches will be faster.
    

---

## Review the Nginx Cache Configuration

File: `default.conf`

### Key Configuration Elements:

```nginx
proxy_cache_key "$scheme$proxy_host$request_uri";
```

- Cache key includes:
    
    - **Scheme** (HTTP/HTTPS)
        
    - **Host**
        
    - **URI**
        
- Excludes: headers such as `Accept-Language`, cookies, etc.
    

```nginx
proxy_cache_valid 200 60s;
```

- Only **200 OK** responses are cached
    
- Cached for **60 seconds**
    

```nginx
proxy_ignore_headers Set-Cookie;
```

- Ignores the `Set-Cookie` header when caching
    

```nginx
add_header X-Edge-Cache $upstream_cache_status;
```

- Adds a custom header to indicate cache status (`HIT` / `MISS`)
    

> [!note]  
> Understanding web server cache configuration (e.g., Nginx, Apache) can help you quickly spot which headers are unkeyed during a pentest.

---

## Application Access & Discovery

- Access the app through the **edge cache** (Nginx) at:
    
    ```
    http://localhost:8081
    ```
    
- **Do NOT** use port `3000`, which connects directly to the app and bypasses the cache.
    

### Inspecting in Burp Suite

- First load: both **application cache** and **edge cache** miss
    
- Resend request in **Repeater**:
    
    - Edge cache may HIT
        
    - Application cache may MISS
        

> [!note]  
> Dual caching (application + edge) can cause unusual behavior. Be cautious when diagnosing cache hits/misses.

---

## Identifying the Unkeyed Input

### Step-by-Step Process:

1. Observe the `Accept-Language` header in request:
    
    ```http
    Accept-Language: en;q=0.9
```
1. Note its **reflection** in the response HTML.
    
2. Replace value with a unique string:
    
    ```http
    Accept-Language: qwerty
```
    
3. Search response for `qwerty`:
    
    - Avoid common words (like `test`) to prevent false positives.
        
    - Suggested value: `aardvark` (rare in real apps).
        
4. Enable **auto-scroll** in Burp to easily spot reflections.
    
5. Wait 60 seconds for previous cache to expire, then resend.

> [!tip]  
> Using rare strings helps eliminate noise in HTML responses during manual testing.

---

## Cache Poisoning Exploit

### Inject XSS Payload:

```http
Accept-Language: <svg onload=prompt(1)>
```

- Wait for cache expiry, then send the request.
    
- Response reflects the payload.
    
- Open a **new incognito browser**:
    
    - Navigate to: `http://localhost:8081`
        
    - Confirm that the XSS executes.


> [!note]  
> This is a **stored XSS via cache poisoning**, even though the site only reflects the input. Without caching, this would be classified as **self-XSS** and generally out-of-scope for bug bounties.

---

## Using Param Miner (Optional)

### Setup:

1. Install from **Burp Suite → Extensions → BApp Store → Param Miner**
    
2. In **Repeater**, run:
    
    ```
    Extensions → Param Miner → Guess Headers
    ```

- Param Miner will test many headers and watch for reflected or behavioral changes.


### Limitations:

- In this lab, Param Miner **did not detect** the unkeyed header.
    
- Reason: it doesn't have a test case tailored for this specific `Accept-Language` reflection.


> [!warning]  
> Automated tools like Param Miner are helpful, but **do not catch everything**. Manual verification is always necessary.

---

## Fixing the Vulnerability

### Mitigation Strategies:

1. **Include the header in the cache key**  
    Example (Nginx):
    
    ```nginx
    proxy_cache_key "$scheme$proxy_host$request_uri$http_accept_language";
    ```
    
2. **Add a `Vary` header**
    
    ```http
    Vary: Accept-Language
```
    
    - Tells caches to store separate versions for different values of the header
        
3. **Sanitize reflected input**
    
    - Update templating engine to **escape output**:
        
        - Replace unsafe double-curly mustaches `{{ input }}` with:
            
            ```handlebars
            {{= input }}
            ```
            

---

## Tools Mentioned / Used

|Tool / Component|Description|
|---|---|
|**Docker / Docker Compose**|Lab containerization environment|
|**Nginx**|Edge cache handling for the application|
|**Burp Suite**|Manual testing and HTTP inspection|
|**Param Miner**|Header fuzzing for unkeyed input discovery|
|_(Implied)_ Templating Engines|Application rendering logic (escaped vs unescaped output)|
|_(Implied)_ Chromium / Incognito Mode|To verify poisoned cache as a new user|
