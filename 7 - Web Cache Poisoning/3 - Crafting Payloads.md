# Web Cache Poisoning – Crafting Payloads

## High-Level Summary

Once an **unkeyed input** is identified, the next step is to **craft a payload** that modifies the server's response in a way that, when cached, causes **harmful effects** for other users. This includes injecting malicious scripts (stored XSS), redirecting users to phishing sites, causing denial of service, or even exposing internal server data. This section outlines the most common payload strategies used in cache poisoning attacks and highlights specific techniques and headers used to trigger these behaviors.

---

## Purpose of Crafting Payloads

The goal is to manipulate an application’s response via an **unkeyed input** so that the **cached version** of the response performs one of the following:

- Executes **malicious scripts** (XSS)
    
- Redirects users to **attacker-controlled sites**
    
- Returns **error pages** (Denial of Service)
    
- Exposes **internal or sensitive data** (SSRF + Cache Leak)

---

## Key Attack Types Enabled by Cache Poisoning

### 1. Stored Cross-Site Scripting (XSS)

If the unkeyed input is reflected in the **HTML output**, you can inject JavaScript payloads.

#### Example:

```http
Accept-Language: <script>alert(1)</script>
```

- If this is reflected in the page:
    
    ```html
    <p>Language: <script>alert(1)</script></p>
    ```
    
- And the page is cached, all users will see the alert.
    

> [!tip]  
> This technique **upgrades self-XSS to stored XSS**, increasing both impact and bug bounty reward potential.

---

### 2. Redirects and Phishing

If a redirect endpoint reflects user-controlled input from an **unkeyed parameter**, you can poison the redirect destination.

#### Example Endpoint:

```http
GET /redirect?url=http://phishing-site.com
```

#### Server Response:

```http
HTTP/1.1 302 Found
Location: http://phishing-site.com
```

If the cache **ignores the `url` parameter**, this redirect gets stored under `/redirect`, affecting all future users.

> [!warning]  
> Any user visiting `/redirect` may be sent to the **attacker-controlled domain**, even if they use a different or no `url` parameter.

---

### 3. Denial of Service (DoS)

Triggering an error response (e.g., `500 Internal Server Error`) via an unkeyed input can cause the cache to store that error page.

#### Example Technique:

```http
Host: victim.com:81
```

- If the backend doesn’t serve anything on port 81, it may return `404` or `400`.
    
- If the cache ignores port numbers in the key, this error page is cached.
    

> [!note]  
> This results in a **Cache Poisoned Denial of Service (CPDoS)**: all users receive the error page until the cache is purged.

---

### 4. Exposing Internal Resources (SSRF + Caching)

Sometimes, the backend **uses an unkeyed input to perform server-side fetches** (SSRF). If the response is cached, sensitive data can leak.

#### Attack Chain:

1. Identify an endpoint like:
    
    ```
    GET /avatar?user=123
    ```
    
    ...which fetches internal avatars from a service.
    
2. Inject unkeyed header:
    
    ```http
    X-Forwarded-Host: 169.254.169.254
```
    
    (This is the AWS metadata IP.)
    
3. Backend fetches the metadata and returns it in the response.
    
4. If this response is cached under `/avatar?user=123`, anyone can access it later to retrieve the internal data.
    

> [!warning]  
> This is a severe **data exposure vector** that combines SSRF and cache poisoning.

---

## Summary of Payload Goals

|Payload Type|Goal|
|---|---|
|**Stored XSS**|Execute attacker-supplied JavaScript for every user|
|**Redirects / Phishing**|Redirect users to malicious websites|
|**Denial of Service**|Serve error or maintenance pages to all users|
|**Internal Resource Access (SSRF)**|Cache and expose sensitive internal data|

> [!tip]  
> Focus on how your **unkeyed input changes the response**, and then check if that response can be **stored in cache**.

---

## Tools Mentioned / Implied

|Tool / Component|Description|
|---|---|
|_(Implied)_ Burp Suite|For crafting, sending, and observing requests/responses|
|_(Implied)_ Dev Tools|For inspecting HTML and cached JavaScript or redirect behaviors|
|_(Implied)_ AWS Metadata IP|Used in SSRF to retrieve cloud instance data|
|_(Implied)_ Custom Scripts|To automate header injection or manipulate redirect parameters|

---

> [!note] How `X-Forwarded-Host` Can Be Exploited in SSRF & Web Cache Poisoning
> 
> - You request:  
>     `GET /avatar?user=123`  
>     With the header:  
>     `X-Forwarded-Host: 169.254.169.254`
>     
> - 🔁 What it does:  
>     `X-Forwarded-Host` tells the backend to **pretend** the original host was `169.254.169.254`.
>     
> - 🧠 Why it matters:  
>     If the backend **uses that header** to build internal URLs (e.g., `http://<host>/internal/avatar/123`), it may make a **server-side request** — that’s **SSRF**.
>     
> - 🕵️‍♂️ No user redirection:  
>     The user is **not redirected** — the backend makes an **internal request** using that header.
>     
> - 🧪 Example misuse in backend:
>     
>     ```python
>     url = "http://" + request.headers["X-Forwarded-Host"] + "/avatar/" + user_id
>     data = requests.get(url)
>     ```
>     
> - ☣️ Result:  
>     The backend fetches `http://169.254.169.254/avatar/123`, returns internal data, and caches it.
>     
> - 💥 Cache Poisoning Risk:  
>     If `/avatar?user=123` is cacheable, that **internal response is now served to all users**.
>     
> - ✅ Summary:  
>     This technique chains **SSRF** with **Web Cache Poisoning** using an unkeyed header (`X-Forwarded-Host`) that the backend trusts.
>     

---
