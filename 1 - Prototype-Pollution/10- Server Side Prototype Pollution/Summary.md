

# Server-Side Prototype Pollution Challenge (Detailed Exam Notes)

> [!summary]  
> Comprehensive notes explaining why and how server-side prototype pollution payloads are delivered and verified, including debugging and payload considerations for open-book exams.

---

## Understanding Payload Delivery and Content Types

- Applications often accept multiple content types (e.g., URL-encoded forms, JSON, XML).
    
- Changing the content type can reveal additional vulnerabilities.
    

### Why Change from URL-encoded to JSON?

- **URL-encoded forms** typically pass flat key-value pairs. JSON allows complex, nested objects, making it easier to inject dangerous properties like `__proto__`.
    
- JSON payloads directly translate to JavaScript objects, facilitating direct manipulation of server-side objects and increasing the attack surface for prototype pollution.
    

> [!important]  
> JSON payloads simplify prototype pollution because they naturally map to JavaScript objects.

---

## Safe Verification of Prototype Pollution

- Objective: Identify prototype pollution without crashing or permanently disrupting the application.
    
- Gareth Hayes' method (recommended):
    

**Payload Example:**

```json
{"__proto__":{"status":599}}
```

- Send malformed JSON to trigger an error.
    
- Normal server response: **400 Bad Request**.
    
- Polluted response (if prototype pollution succeeds): **599 Custom Status**.
    

### Why use a low-impact payload?

- High-impact payloads (like polluting critical properties) can permanently crash applications.
    
- Low-impact payloads (changing a status code) provide proof of pollution without causing severe disruption.
    

> [!caution]  
> Always revert changes after confirming pollution to avoid temporary denial of service.

---

## Detailed Testing Workflow

### Step 1: Recon and Payload Conversion

- Capture and analyze HTTP requests using Burp Suite Proxy.
    
- Convert captured URL-encoded form payload to JSON format using Burp Suite's Content Type Converter.
    

### Step 2: Confirming Prototype Pollution

- Inject payload:
    

```json
{"__proto__":{"polluted":true}}
```

- Use debugger or server-side console to verify:
    

```javascript
console.log({}.polluted); // true confirms pollution
```

> [!important]  
> Server-side prototype pollution may not produce immediate client-side changes. Debugging is essential for verification.

---

## Debugging with Breakpoints

### Why Debugging is Important

- Allows runtime inspection of application states.
    
- Confirms prototype pollution that doesn't visibly affect the application.
    

### How to Use Debugger Effectively

- Set breakpoints before and after suspect merging functions (e.g., `$.extend`, object merging).
    
- Examine object prototypes directly within the debugger:
    

```javascript
object.prototype
```

- Temporarily disable authentication and security checks to streamline testing.
    

---

## Exploiting Sensitive Properties

- Prototype pollution becomes critical when sensitive properties (e.g., user roles) are polluted.
    

### Example: Admin Role Escalation

```javascript
if (request.session.role !== 'admin') redirect();
```

Inject via JSON payload:

```json
{"__proto__":{"role":"admin"}}
```

- Result: All users inherit `role="admin"` due to polluted global object prototype.
    

> [!danger]  
> Always carefully consider the impact of polluting sensitive properties.

---

## Bypassing Filters and Alternative Payloads

- Filters might remove specific keywords like `__proto__`.
    
- Test for filter presence by injecting reflected parameters and observing server responses.
    

### Example Filter Bypass

If `__proto__` is filtered, try recursive or nested variations:

```json
{"__pro__to__":{"polluted":true}}
```

- Use different notations (bracket, dot, constructor notation).
    
- Reference payload lists and documentation regularly:
    

[Prototype Pollution Payloads - ACSECexplained](https://acsecexplained.gitbook.io)

---

## Essential Exam Takeaways

> [!success]  
> Exam essentials:

- Change content types to increase attack surface.
    
- Use low-impact payloads to safely confirm prototype pollution.
    
- Debugging and breakpoints are critical for inspecting and verifying payload effects.
    
- Always consider sensitive properties that could escalate the attack severity.
    
- Keep alternative payload strategies ready to bypass filters effectively.
    

---