## Prototype Pollution Scanner (Burp Suite Pro)

> [!info]  
> **Overview**  
> This section provides a detailed walkthrough of using the **Server-Side Prototype Pollution Scanner** plugin in **Burp Suite Pro**. It explains installation, configuration, usage, and important caveats to keep in mind while testing applications.

---

### Installing the Scanner Plugin

- Open Burp Suite Pro and go to **Extensions > BApp Store**.
    
- Search for `Prototype`.
    
- Install the plugin called **Server-Side Prototype Pollution Scanner**.
    

> [!note]  
> The scanner is **only fully functional in Burp Suite Pro**. While you can install it in Community Edition, you won’t see scan results since the **Issues** panel is not accessible.

---

### Understanding How the Scanner Works

- The scanner checks various payload types:
    
    - `body.scan`: JSON body scans
        
    - `body.dot.scan`: Payloads like `__proto__.x`
        
    - `body.square.scan`: Payloads like `__proto__[x]`
        

> [!tip]  
> Always skim through the plugin documentation to understand the techniques it uses and its limitations.

Once installed, a **"Server Side Prototype Pollution"** button appears in the top bar of Burp. Clicking it opens a settings panel.

- Each scan type can be toggled.
    
- Hovering over a setting reveals a tooltip with additional info.
    

---

### Running a Scan

1. Navigate to **HTTP History** in Burp Suite.
    
2. Find a POST request (e.g., `POST /tasks/create`) and send it to **Repeater**.
    
3. Right-click in the request panel > **Extensions > Server Side Prototype Pollution > Full Scan**.
    

> [!caution]  
> If you’re testing on a production system, avoid full scans. Use individual scans to isolate bugs more safely.

#### JSON Body Requirement

- If the request isn’t in JSON format, you may need to **manually convert it to JSON** for the plugin to function.
    

```json
{
  "type": "song",
  "status": "test"
}
```

Once submitted, Burp will begin testing various payloads and monitor the responses.

---

### Viewing Scanner Results

- Go to the **Issues** tab (only in Burp Suite Pro).
    
- Look for findings labeled like `Server-side prototype pollution via JSON status`.
    
- Clicking into the issue shows:
    
    - The payload that triggered the response
        
    - Response codes
        
    - Error and validation behavior
        
    - Remediation details
        

#### Example Workflow

1. Plugin sends an original payload.
    
2. Then it sends malformed JSON to trigger errors (e.g., `{` to get 510).
    
3. Sends valid JSON with `status: 0` and observes if it resets logic.
    

---

### Observations During Testing

- A successful payload triggers a 302 or 510 response.
    
- Malformed payloads are used to test app stability and error handling.
    
- JSON with `status: 0` may revert to original behavior.
    

> [!warning]  
> Some payloads may **temporarily crash** or break parts of the app. In testing environments, this is acceptable. In production, this could be disruptive.

---

### Final Notes

- The scanner generates a lot of traffic.
    
- Useful for **broad endpoint coverage**.
    
- Scanner found a working payload: `Server-side prototype pollution via JSON status`.
    
- Good for triaging and automating testing, but manual confirmation is still important.
    

> [!important]  
> **Workflow Reminder:**
> 
> - Use scanner for wide coverage when time is limited.
>     
> - Understand tool behavior to avoid missing issues.
>     
> - Always validate findings manually when possible.
>     
