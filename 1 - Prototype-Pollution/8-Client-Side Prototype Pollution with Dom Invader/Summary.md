
# Client-Side Prototype Pollution with DOM Invader (Concise Summary)

> [!summary]  
> Summarizes key concepts and workflows for identifying client-side prototype pollution using Burp Suite's DOM Invader extension.

---

## Overview

- **DOM Invader** (Burp Suite extension) automates detection of **prototype pollution** vulnerabilities and identifies potential sinks such as DOM-based XSS via `innerHTML`.
    
- Vulnerabilities exist even without external libraries (e.g., vanilla JavaScript).
    

---

## Setting Up DOM Invader

- Enable DOM Invader in Burp Suite’s Chromium browser.
    
- Go to **Attack Types** → enable **Prototype Pollution**.
    
- Refresh page, open Dev Tools (`F12`) and select the **DOM Invader** tab.
    

> [!important]  
> Ensure Prototype Pollution is explicitly enabled for DOM Invader to detect relevant vulnerabilities.

---

## Detecting Pollution Entry Points

DOM Invader commonly detects two entry points:

- `__proto__[property]=value` via URL parameters.
    
- `constructor.prototype[property]=value` via URL parameters.
    

### Confirming Pollution

Example payload:

```
?constructor.prototype.polluted=true
```

Confirm via console:

```javascript
console.log({}.polluted); // true indicates successful pollution
```

> [!important]  
> Remember alternate payloads (`constructor.prototype`) alongside `__proto__` for exams.

---

## Finding Pollution Sinks

- Click **"Scan for Gadgets"** in DOM Invader to identify potential sinks (e.g., `innerHTML`).
    
- DOM Invader provides automated payload injection via **"Exploit"** button.
    

> [!danger]  
> If `innerHTML` appears as a sink, DOM-based XSS is possible and exploitable.

---

## If DOM Invader Misses the Sink

- DOM Invader may fail to identify some exploitation paths.
    
- If no sink is found, manually inspect the source code and use Dev Tools breakpoints to trace data flows.
    

> [!caution]  
> Relying solely on automated tools can miss important sinks—manual verification is often necessary.

---

## Practicing on `/v2` Dashboard

- Version `/v2` does **not use `dparam`**; relies on vanilla JavaScript.
    
- Still vulnerable to the same prototype pollution attacks.
    

> [!note]  
> Use `/v2` for hands-on practice without external libraries like `dparam`.

---

## Key Exam Reminders

- Enable **Prototype Pollution** in DOM Invader.
    
- Test multiple payload types: `__proto__`, `constructor.prototype`.
    
- Verify pollution via browser console:
    
    ```js
    console.log({}.polluted);
    ```
    
- Use **Scan for Gadgets** to identify sinks.
    
- Manual inspection is crucial if automated detection fails.
    

> [!important]  
> Understanding manual prototype pollution workflows alongside DOM Invader ensures consistent success in exams and real-world scenarios.

---