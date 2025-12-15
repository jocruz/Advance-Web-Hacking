
## **Prototype Pollution Scanner (Burp Suite Extension)**

> [!info]  
> **Overview**  
> This section introduces the **Server-Side Prototype Pollution Scanner** Burp Suite extension, explaining its usage, limitations, and behavior during active scans. The scanner automates the detection of prototype pollution via JSON payloads and helps validate findings with proper context and remediation notes.

---

### **Tool Access and Requirements**

- The **Prototype Pollution Scanner** was developed by **Gareth Hayes**.
    
- Accessible via the **Burp Suite App Store** under the name:  
    `Server Side Prototype Pollution Scanner`
    
- ✅ Can be installed on both **Burp Suite Pro** and **Community** editions.  
    ❌ However, **results are only visible in Burp Suite Pro**.
    

> [!tip]  
> If you're using **Burp Suite Community**, you can still install the extension, but you won't see issue details in the **Issues Panel**. You can still follow along with video demonstrations or upgrade to Pro for full visibility.

---

### **Installing the Extension**

1. Navigate to `Extensions` → `BApp Store`
    
2. Search for: `Prototype`
    
3. Click **Install** on the **Server Side Prototype Pollution Scanner**
    

The extension offers multiple scanning modes:

- `body.scan` – Injects JSON using dot notation (e.g. `__proto__.key`)
    
- `bodySquare.scan` – Injects using square brackets (e.g. `__proto__["key"]`)
    

> [!info]  
> These techniques are based on previously discussed payload structures that exploit how objects are merged or interpreted in backend code.

---

### **Exploration and Use**

#### **Step-by-Step Usage**

1. Log into the target application and create a new task (or equivalent data).
    
2. Go to **HTTP History**, find the relevant POST request (e.g., `POST /tasks/create`)
    
3. Send the request to **Repeater**
    
4. Right-click → `Extensions` → `Server Side Prototype Pollution` → **Full Scan**
    

> [!important]  
> **Warning for Production Use:**  
> Always assess the environment before running full scans. On **production systems**, scan selectively to prevent unintended side effects.

---

### **Scan Settings and Options**

- Clicking the plugin's toolbar icon opens detailed configuration settings.
    
- Hover over any setting for a quick explanation.
    
- Scan types include:
    
    - **Full Scan**
        
    - **Custom payload testing**
        
    - **Payload type toggles** for dot notation and bracket-based payloads.
        

---

### **Reading and Validating Results**

- After running a scan, results appear under the **Issues** tab (Pro only).
    
- Example finding:  
    **Server-side prototype pollution via JSON `status`**
    
- Useful details provided include:
    
    - Affected parameter
        
    - Sample payload
        
    - Remediation guidance
        

---

### **Understanding Scan Behavior**

#### **Request Behavior**

The scanner sends multiple crafted payloads and observes response codes:

- `302 Found` → Redirects
    
- `510` → Server error from malformed JSON
    
- `400 Bad Request` → Rejected payload
    
- `status=0` → Resets status to default
    

These behaviors help infer the backend logic and whether the payload was successfully interpreted.

> [!note]  
> In this example, status `0` seemed to revert the server to a functional state after prior errors like `510`. It's a useful pattern to recognize when analyzing scan impact.

---

### **Considerations and Impact**

- The scan **can generate high traffic**, especially on large or shared systems.
    
- If targeting a live application:
    
    - Expect noise in logs and visible data clutter
        
    - Some users or admins may detect this activity
        
- If targeting local VMs:  
    ✅ You have full control, so run full scans freely.
    

> [!warning]  
> **Be cautious with aggressive scans on production** environments.  
> Scan failures (e.g., temporary app crashes) may disrupt service and raise suspicion.

---

### **Final Thoughts**

- The **Server-Side Prototype Pollution Scanner** is a **powerful tool** for quickly testing endpoints across large applications.
    
- Results include **payloads, request/response pairs, and remediation tips**, aiding in effective reporting.
    
- Scanning should be a regular part of your **workflow**, especially when time or resources limit manual testing.
    

> [!success]  
> **Best Practices:**
> 
> - Understand each scan mode and payload type before use.
>     
> - Use Burp Suite Pro to fully leverage the plugin’s issue output.
>     
> - Analyze results critically; not every flagged issue is exploitable.
>     
> - Combine with manual validation to avoid false positives or missed logic paths.
>     
