

---

# Server-Side Prototype Pollution Scanner – Simplified Guide

> [!summary]  
> This Burp Suite extension **automatically checks** for server-side prototype pollution vulnerabilities by injecting crafted JSON payloads and observing how the server responds.

---

## What It Is

The **Server-Side Prototype Pollution Scanner** is a Burp Suite extension created by **Gareth Hayes** that:

- Tests for **server-side prototype pollution** automatically.
    
- Sends different **JSON payloads** with `__proto__` to the server.
    
- Monitors changes in **response codes** or behavior to detect if pollution worked.
    

> [!note]  
> Prototype pollution can let an attacker change global behavior on the server (e.g., change status codes, roles, or other sensitive values).

---

## When To Use It

Use this tool when:

- You're testing a **JavaScript-based backend** (like Node.js/Express).
    
- You notice the server **accepts JSON payloads** (e.g., `Content-Type: application/json`).
    
- You're **not sure** if prototype pollution is exploitable and want to **automate the test**.
    

> [!tip]  
> It's especially helpful when manually testing is too slow or repetitive.

---

## Limitations

> [!important]  
> You **need Burp Suite Pro** to see the results.

|Feature|Burp Suite Pro|Burp Suite Community|
|---|---|---|
|Install extension|✅ Yes|✅ Yes|
|View scan results (Issues)|✅ Yes|❌ Not visible|
|Full scan integration|✅ Yes|⚠️ Limited (manual only)|

---

## How It Works – Step by Step

### 🧪 Scan Workflow:

1. **Capture a request** in Burp (e.g., `POST /create-task`).
    
2. **Send it to Repeater**.
    
3. **Right-click** in Repeater → `Extensions` → `Server Side Prototype Pollution` → `Full Scan`.
    

> [!caution]  
> Only do this in **safe environments**. In production, you could accidentally disrupt the app.

---

## What It Sends (Behind the Scenes)

The scanner sends **payloads like**:

```json
{
  "__proto__": {
    "status": 599
  }
}
```

And then watches how the server reacts:

- ✅ Normal request returns `302 Found`.
    
- ❌ Malformed payload causes a `510` or other error.
    
- ✅ Reset request returns server to `302`.
    
- 🧪 Final test checks if pollution sticks.
    

> [!note]  
> These response code patterns help confirm that the pollution was **successful and temporary**, without crashing the app.

---

## Key Payload Types Used

|Notation Type|Example|Use Case|
|---|---|---|
|Dot notation|`"__proto__.polluted": true`|Traditional JS pollution|
|Bracket notation|`"__proto__['polluted']": true`|Bypass basic filters|
|Safe testing|`"__proto__": {"status": 599}`|Low-impact detection payload|

---

## Where to Find the Extension

1. Open Burp → **Extensions → BApp Store**.
    
2. Search: `"prototype"` or `"pollution"`.
    
3. Install: **Server Side Prototype Pollution Scanner** by Gareth Hayes.
    

> [!tip]  
> After installing, scan options appear in the **Repeater right-click menu**.

---

## Interpreting Results (Pro Users Only)

- Go to the **Issues** tab to view findings.
    
- Look for:  
    `Server-side prototype pollution via JSON status`
    

It means the tool likely succeeded in injecting a pollution payload and altering server behavior temporarily.

---

## Best Practices

> [!success]  
> **Exam + Real-World Tips:**

- Always **review the full request and response** in Repeater to understand what changed.
    
- Use this tool as a **first pass**, then manually confirm.
    
- Focus on endpoints that:
    
    - Accept `application/json`
        
    - Involve object merging or data manipulation
        
- Document any issues with the payload used and observed response changes.
    

> [!warning]  
> **Don’t use aggressive scans on production** systems. It may fill logs, break functionality, or trigger rate limits.