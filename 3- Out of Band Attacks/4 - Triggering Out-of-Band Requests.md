# Triggering Out-of-Band Requests & SSRF Context

## High-Level Summary

This note explores the concept of **triggering out-of-band (OOB) interactions**, particularly through **Server-Side Request Forgery (SSRF)**. It emphasizes how SSRF can exist as a **standalone vulnerability** or be a **secondary consequence** of other vulnerabilities like **XXE**, **blind SQLi**, or **server-side rendering**. The note concludes with a practical **4-step testing methodology** for discovering and leveraging OOB interactions.

---

## What is an Out-of-Band Request?

An **out-of-band (OOB) request** occurs when an application is tricked into making an **unsolicited HTTP/DNS/FTP request** to a server controlled by the attacker.

> [!info]
> Out-of-band interactions are often used to identify **blind vulnerabilities** or gain **insight into internal service behavior** when direct feedback is not provided by the target.

---

## SSRF: The Primary Vector

> [!info]
> **Server-Side Request Forgery (SSRF)** is one of the **simplest and most common ways** to trigger OOB requests.

### Typical SSRF Context:
- Application takes a **user-supplied URL** (e.g., image fetcher, webhook, API proxy).
- Fails to validate or sanitize the input.
- Server performs the request on behalf of the user.

**Result**: Server makes a request to a location it shouldn't (e.g., internal services, AWS metadata).

---

## Other Vulnerabilities That Can Trigger OOB Requests

While SSRF is the primary method, other vulnerabilities can also result in out-of-band behavior:

> [!tip]
> Use OOB payloads in **secondary vulnerabilities** with limited impact to potentially escalate severity.

- **XXE (XML External Entity Injection)**
- **File Upload** (e.g., uploading HTML or images that are parsed server-side)
- **Server-Side Rendering** (SSR)
  - HTML/JS rendered server-side can generate OOB triggers
- **Blind SQL Injection**
- **Deserialization**

> [!note]
> If you already have **high-impact bugs** like RCE or command injection, OOB requests may be irrelevant.  
> However, for **low/medium severity issues**, OOB interactions can uncover additional attack surface or pivot points.

---

## Testing Methodology: Triggering OOB Requests

Here is a **4-step practical checklist** for testing OOB behavior in an application:

---

### 1. **Set Up a Collector**
- Use a service like:
  - **Burp Collaborator**
  - **Interact.sh**
- Purpose: Receive HTTP/DNS/etc. callbacks from the target application.

---

### 2. **Map the Application for OOB Entry Points**
Look for areas where **user input might trigger server-side requests**:

- URL fields in parameters or JSON
- File upload functionality (e.g., PDF/image parsing)
- PDF or image generators
- Server-side rendering (HTML or JS input)
- Webhooks, third-party integrations
- Any other vulnerabilities (XXE, blind SQLi, etc.)

> [!tip]
> Even if a URL field seems harmless (e.g., avatar fetch), test it—some OOB triggers are hidden behind secondary processing.

---

### 3. **Craft and Inject Payloads**
Send payloads to identified entry points:

Examples:
```http
http://<your-collaborator-subdomain>
http://127.0.0.1:80
http://169.254.169.254/latest/meta-data
````

- Use **unique subdomains** per injection to track specific payload triggers.
    
- Try multiple schemes: `http`, `https`, `ftp`, `gopher` (if supported).
    

---

### 4. **Evaluate Impact and Pivot**

If a payload is triggered:

- Determine what was accessed (e.g., was internal metadata reached?).
    
- Use the OOB behavior to:
    
    - Discover **internal services**
        
    - Leak **sensitive data**
        
    - Access **unauthorized endpoints**
        

> [!warning]  
> While SSRF may not always provide a full exploit chain, it can expose **misconfigurations**, **cloud instance metadata**, or be used in chained attacks (e.g., SSRF to RCE).

---

## Summary & Key Considerations

- SSRF is the **most direct and repeatable way** to trigger out-of-band interactions.
    
- Don’t overlook **low-severity bugs**; use them to test for OOB behaviors.
    
- Always use a **reliable OOB collector** like Interact.sh or Burp Collaborator.
    
- Consider **out-of-band testing** part of your **default methodology**, especially in modern app architectures with SSR, APIs, and integrations.
    

> [!important]  
> **Out-of-band testing is critical for finding blind or indirect vulnerabilities** that won't show in immediate responses.
