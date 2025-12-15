## Capstone Challenge: Prototype Pollution Walkthrough

> [!summary]  
> **Overview**  
> This walkthrough simulates a real-world security engagement focused on identifying and exploiting **Prototype Pollution** vulnerabilities. While built-in tools like Burp Suite’s DOM Invader and server-side scanners assist in detection, manual code review and targeted payload testing remain crucial for comprehensive vulnerability identification.
> 
> This guide covers both **client-side** and **server-side** prototype pollution, explains effective filter bypass techniques, and emphasizes debugging methods.

---

## 1. Initial Observations

> [!note]  
> **Key Insights**
> 
> - A deliberately vulnerable library was included in the challenge, which did **not** trigger warnings from automated scanners (`npm audit`).
>     
> - This underscores the importance of manual analysis alongside automated tools.
>     

---

## 2. Client-Side Prototype Pollution Analysis

### DOM Invader Analysis (Burp Suite)

- **Community Edition:** DOM Invader **failed** to detect the vulnerability.
    
- **Professional Edition:** DOM Invader successfully detected Prototype Pollution:
    
    - Source: Identified `__proto__` pollution via the `message` parameter.
        
    - Sink: Detected as `elements.innerHTML`.
        

> [!example]  
> **Payload Example:**
> 
> ```json
> { "message": "<script>alert('XSS')</script>" }
> ```

> [!note]  
> Always manually verify vulnerabilities detected by automated tools to confirm actual impact.

### Manual Code Review (`merge.js`)

A manual review found a weak filter designed to block dangerous keys:

```javascript
const blockedKeywords = ["__proto__", "constructor", "prototype"];
```

#### Reasons for Filter Failure

- The filter checked **only top-level keys**, neglecting nested objects (non-recursive filtering).
    
- Easily bypassed with concatenation or nested payload structures.
    

> [!example]  
> **Confirmed Bypass Example:**
> 
> ```json
> {
>  "pro" + "to" + "type": { "polluted": true }
> }
> ```

### Debugging Techniques (Using Breakpoints)

> [!note]  
> **Debugging Approach:**
> 
> - Set breakpoints in the browser's developer tools to inspect and debug filter logic.
>     
> - Fuzz payloads and monitor their processing to understand filter behavior and develop effective bypass strategies.
>     

---

## 3. Server-Side Prototype Pollution

### Vulnerable Endpoint: `/tasks/edit/:id`

Code vulnerable to prototype pollution:

```javascript
defaultsDeep(taskDefaults, req.body);
```

#### Exploitation Example

> [!example]  
> **Confirmed Payload:**
> 
> ```json
> {
>  "__proto__": { "polluted": true }
> }
> ```

- Confirmed global prototype pollution by checking: `Object.prototype.polluted === true`
    

### Example of Secure Code (Non-vulnerable endpoint: `/tasks/create`)

```javascript
Object.assign({}, obj1, obj2);
```

> [!note]  
> Always verify assumptions through testing, even when using methods presumed secure.

### Privilege Escalation via Prototype Pollution

Prototype pollution allowed global privilege escalation by modifying user roles.

> [!example]  
> **Privilege Escalation Payload:**
> 
> ```json
> {
>  "__proto__": { "role": "admin" }
> }
> ```

- Result: All application users inherited admin privileges, showcasing critical vulnerability impact.
    

---

## 4. Automated Scanner Effectiveness (Burp Suite Scanner)

### Scanner Detection Capabilities

- Scanner **failed** to detect prototype pollution with content type `application/x-www-form-urlencoded`.
    
- Scanner **successfully** detected vulnerabilities using content type `application/json`.
    

### Scanner Workflow (Step-by-step Analysis)

|Request|Action|Response|Purpose|
|---|---|---|---|
|1|Send valid JSON payload|`200 OK` / `302`|Establish a baseline|
|2|Send prototype pollution payload|Confirmed change|Verify if pollution vulnerability exists|
|3|Send malformed payload|`510` Error|Confirm server behavior is affected|
|4|Send revert payload (`status: 0`)|`200 OK`|Restore server state|
|5|Resend malformed payload|`400 Bad Request`|Confirm server restoration|

---

## 5. Additional Testing Recommendations

> [!note]  
> **Practical Recommendations:**
> 
> - **Test multiple content types** (JSON, form data, XML).
>     
> - Evaluate endpoints handling user input via different parsers.
>     
> - Analyze sensitive properties such as JWT claims, API responses, and user roles to identify impactful prototype pollution payloads.
>     
> - Watch for dangerous functions (e.g., `eval()`) that, combined with prototype pollution, could result in severe exploits like Remote Code Execution (RCE).
>     

---

## Key Takeaways

> [!summary]
> 
> - Prototype pollution affects both **client-side** and **server-side**.
>     
> - Simple filters (`key !== '__proto__'`) are inadequate without **recursive checks**.
>     
> - Effective debugging through **breakpoints and fuzzing** provides deeper insights and aids filter bypasses.
>     
> - Always validate assumptions through testing across multiple content types and request formats.
>     
> - Conduct thorough code reviews, focusing especially on object merging functions that may introduce vulnerabilities.
>