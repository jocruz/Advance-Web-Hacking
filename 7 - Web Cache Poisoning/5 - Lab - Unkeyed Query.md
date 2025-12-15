
# Web Cache Poisoning – Lab: Unkeyed Query Parameter

## High-Level Summary

This lab explores how **unkeyed query parameters** can be exploited to poison cached content and trigger stored XSS. It demonstrates how to use **Param Miner** to identify vulnerable parameters, how to inject a payload, how to confirm cache poisoning, and how to use **cache busters** to manage testing. The lab concludes with an advanced technique involving **cloaked parameters**, which can manipulate backend behavior under specific conditions.

---

## Lab Setup & Initialization

### Start the Environment

1. Extract and enter the lab directory:
    
    ```bash
    tar -xf lab3.tar
    cd lab3/
    ```
    
2. Launch the container:
    
    ```bash
    sudo docker-compose up
    ```


> [!note]  
> This lab builds on the previous one and reuses the same Docker image, so it should spin up quickly.

---

## ## Access the Application

Navigate to:

```
http://localhost:8081
```

> [!tip]  
> Use **port 8081** to hit the Nginx edge cache. Port **3000** bypasses it and hits the app directly.

---

## Identify Unkeyed Query Parameters

### Step 1: Use Param Miner

1. Send the main request to **Repeater**.
    
2. Right-click → `Extensions → Param Miner → Guess Parameters`.
    
    > Choose this over “Guess Headers” for this lab.
    
3. Review results:
    
    - In **Community Edition**, you'll see only the finding count.
        
    - If installed: check `Targets → Issues` or `All Issues` tab.
        
    - Discovered parameter:
        
        ```
        ref
        ```
        

### Step 2: Test Parameter Behavior

Try:

```
GET /?ref=123
```

- First request: `X-Cache: MISS`
    
- Second request: `X-Cache: HIT`

> [!note]  
> This confirms the `ref` parameter **does not alter the cache key** and is therefore **unkeyed**.

---

## Use Cache Busters (CB)

A **cache buster** is a parameter used to force new cache entries. Commonly shortened to `cb`.

### Example:

```http
GET /?cb=1
```

- First time → cache miss
    
- Subsequent → cache hit

### Combine CB with Payloads

To test variations:

```http
GET /?cb=11&ref=qwerty
```

> [!tip]  
> Use cache busters to **control what gets cached** and to avoid interfering with other cached versions of the page.

---

## Crafting and Testing XSS Payloads

### Step-by-Step:

1. Target the reflected `ref` value in the HTML:
    
    - Reflected inside a `<script>` block.
        
2. Payload example:
    
    ```http
    GET /?ref=";prompt(1);//&cb=11
    ```
    
    - Break out of context (`"`)
        
    - Inject `prompt(1)`
        
    - Comment out trailing script (`//`)
        
3. Confirm it gets cached:
    
    - Wait ~60 seconds for the cache to expire.
        
    - Revisit with:
        
        ```
        GET /?ref=qwerty
        ```
        
    - Observe `X-Cache: HIT`
        
4. Validate with a new incognito browser:
    
    - Visit:
        
        ```
        http://localhost:8081
        ```
        
    - Confirm that the **XSS executes**, proving the payload was **successfully cached**.


> [!note]  
> Without cache poisoning, this would be **self-XSS**, which is typically out-of-scope for bug bounties. Caching elevates it to **stored XSS**.

---

## Advanced: Cloaked Parameters

A **cloaked parameter** is a trick to pass multiple parameters in a single key using a `;`.

### Example:

```http
GET /?ref=123;foo=bar
```

- Some backends may interpret `foo` as a second parameter
    
- Behavior depends on how the backend parses semicolons
    

> [!tip]  
> While rare in the wild, this trick is documented in sources like **PortSwigger**. It can be effective against specific stacks or misconfigurations.

---

## Key Takeaways

- **Param Miner** is effective at discovering unkeyed parameters (not just headers).
    
- **Query parameters** can be as dangerous as headers if they influence the response but aren't keyed.
    
- **Cache busters** (e.g., `cb=123`) are useful for:
    
    - Controlling cache entries
        
    - Bypassing existing cache during testing
        
- Always validate cache poisoning using an **incognito browser** to eliminate browser cache interference.
    

---

## Tools Mentioned / Used

|Tool / Component|Description|
|---|---|
|**Docker / Docker Compose**|Lab containerization environment|
|**Nginx**|Edge cache layer in the lab|
|**Burp Suite**|For HTTP history, Repeater, and running Param Miner|
|**Param Miner**|Used to discover unkeyed **query parameters**|
|**Logger++ (optional)**|Burp extension for tracing server-side errors|
|_(Implied)_ Browser Incognito Mode|To verify cache poisoning without browser interference|


> [!tip]  
> **Cache Buster (`cb`)**
> 
> - Appending something like `?cb=123` to the URL forces the server to treat it as a **new request**, bypassing existing cache.
>     
> - Helps ensure you're **testing a fresh response** (for example, when checking if your payload is cached).
>     
> - Use it to **avoid contaminating other cache entries** when experimenting with XSS or other cache-poisoning payloads.
>     


> [!note]  
> **X-Edge-Cache: Expired**
> 
> - This header means the cache entry **used to exist** but is now **stale or timed out**.
>     
> - The edge server re-fetches the resource from the backend and **updates the cache**.
>     
> - Great moment to inject your payload, since the next visitor might get your malicious cached version.


> [!note] Why `?cb=123` Works Even If the Server Doesn't Use It
> 
> - Web servers typically **ignore unknown parameters** like `cb`, unless they are specifically coded to reject them.
>     
> - So when you send `GET /?cb=123`, the backend **renders the same page** as `GET /`, because it doesn't care about `cb`.
>     
> - BUT caching layers (like CDNs or reverse proxies) **treat the entire URL (including query strings) as part of the cache key** — unless configured otherwise.
>     
> - This lets attackers create **distinct cache entries** (e.g., `/`, `/?cb=1`, `/?cb=2`, etc.) even if the backend response is the same.
>     
> 
> **Why it matters:**  
> You can use dummy parameters like `cb` to **manipulate cache behavior** for testing or poisoning without affecting backend functionality.
