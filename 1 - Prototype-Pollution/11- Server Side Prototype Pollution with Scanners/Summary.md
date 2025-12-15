
# Server-Side Prototype Pollution Scanner (Burp Suite Extension)

> [!summary]  
> Detailed explanation of the **Server-Side Prototype Pollution Scanner**, how it operates, its setup, practical usage, interpreting results, and crucial considerations for effective use.

---

## Overview

The **Server-Side Prototype Pollution Scanner**, developed by Gareth Hayes, is a Burp Suite extension designed to automate the detection of server-side prototype pollution vulnerabilities. It specifically tests JSON-based payloads to identify exploitable conditions where user-controlled input may manipulate server-side JavaScript objects.

---

## Tool Access and Limitations

- **Available** in the Burp Suite App Store under: `Server Side Prototype Pollution Scanner`
    
- **Compatible** with both Burp Suite Pro and Community editions.
    
- **Visibility of results** is limited to **Burp Suite Pro** only (Community users cannot see issues).
    

> [!important]  
> While Community users can install the extension, results are not visible. Watching demonstrations or upgrading to Pro is required for full usage.

---

## Installing the Extension

1. Open **Burp Suite → Extensions → BApp Store**.
    
2. Search for **"Prototype"**.
    
3. Install **"Server Side Prototype Pollution Scanner"**.
    

### Scan Types Provided:

- **Dot notation** payload (`__proto__.key`)
    
- **Square bracket notation** payload (`__proto__["key"]`)
    

---

## Step-by-Step Scanner Usage

1. **Login and Prepare the Target Application:**
    
    - Log into the target application.
        
    - Perform basic actions (e.g., create a task).
        
2. **Capture Request:**
    
    - Go to **HTTP History** in Burp Suite.
        
    - Identify the relevant request (e.g., `POST /tasks/create`).
        
3. **Send to Repeater and Scan:**
    
    - Right-click the request → **Send to Repeater**.
        
    - In Repeater, right-click → **Extensions → Server Side Prototype Pollution → Full Scan**.
        

> [!caution]  
> On production systems, be cautious and run scans selectively to prevent unintended disruptions or denial of service.

---

## Why Modify Content Type from URL-encoded to JSON?

- JSON payloads map directly to JavaScript objects, providing a straightforward mechanism to exploit prototype pollution.
    
- URL-encoded data typically doesn't support nested structures clearly, making JSON a better candidate for pollution attacks.
    

---

## Interpreting Scan Results

- Results appear in the **Issues tab** (Burp Suite Pro).
    
- Example issue detected: **"Server-side prototype pollution via JSON status"**.
    
- The scanner identifies pollution by manipulating the HTTP status codes and monitoring server responses.
    

### Typical Scanning Workflow Observed:

1. **Initial Valid Payload:** Sends crafted payload, expecting normal server behavior (`302 Found`).
    
2. **Malformed JSON Trigger:** Intentionally malformed payload (`{`) triggers server error (`510`).
    
3. **Reset Payload:** Sends payload (`status=0`) aiming to revert server state back to normal (`302 Found`).
    
4. **Verification:** Sends another malformed payload, confirming if the server returns standard error (`400 Bad Request`).
    

> [!note]  
> Recognizing these response patterns (302 → 510 → 302 → 400) helps confirm successful prototype pollution detection and its temporary effects on server stability.

---

## Impact and Considerations

- **High Traffic Generation:** Extensive scans can flood the server with numerous requests.
    
- **Production Environment:** Scans may clutter logs, temporarily disrupt functionality, or cause performance degradation.
    
- **Local Testing Environment:** Extensive scans are safe and recommended for thorough testing.
    

> [!warning]  
> Always consider the environment’s stability and privacy when conducting aggressive scans to avoid unintended consequences or disruptions.

---

## Best Practices & Exam Reminders

- Familiarize yourself with each scan setting and payload type.
    
- Use **Burp Suite Pro** to fully benefit from scanner capabilities.
    
- Always manually validate automated findings to confirm genuine vulnerabilities and avoid false positives.
    
- Include detailed issue reports with payloads and remediation suggestions.
    

> [!success]  
> **Effective Workflow Tips:**
> 
> - Combine automated scanning with manual inspection.
>     
> - Pay attention to server responses and application behavior during scanning.
>     
> - Regularly review tool documentation and settings for optimal results.
>     

---