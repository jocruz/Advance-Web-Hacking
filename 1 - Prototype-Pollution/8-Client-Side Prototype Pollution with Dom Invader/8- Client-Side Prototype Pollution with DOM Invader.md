## **Client-Side Prototype Pollution with DOM Invader**

> [!info]  
> **Overview**  
> This walkthrough demonstrates how to use **Burp Suite's DOM Invader extension** to identify and exploit **client-side prototype pollution vulnerabilities**. DOM Invader automates detection of pollution sources and potential sinks (e.g., XSS via `innerHTML`).
> 
> A version of the vulnerable dashboard exists at `/v2` which **does not use `dparam`** but is still vulnerable via **vanilla JavaScript**. The same attack vectors apply.

---

### **Getting Started with DOM Invader in Burp Suite**

> [!tip]  
> Make sure DOM Invader is enabled in Burp Suite Chromium:
> 
> - Enable the extension manually.
>     
> - Go to **Attack Types** and enable **Prototype Pollution**.
>     
> - Refresh the page.
>     

Access the Dev Tools:

```plaintext
F12 or Ctrl+Shift+C to open the console
```

Then select the **DOM Invader** tab.

---

### **Detection of Prototype Pollution Entry Points**

DOM Invader automatically identified these entry points:

- `__proto__[property]=value` in `location.search`
    
- `constructor.prototype[property]=value` in `location.search`
    

> [!important]  
> **Exam Note:**  
> DOM Invader supports **alternate payloads** like `constructor.prototype`, not just `__proto__`.  
> Example payload:
> 
> ```plaintext
> ?constructor.prototype.Jeremy=true
> ```
> 
> Then confirm via console:
> 
> ```js
> console.log({}.Jeremy); // true
> ```

---

### **Manually Confirming the Pollution**

After injecting a payload:

```plaintext
?constructor.prototype.polluted=true
```

Check pollution in the browser console:

```js
console.log({}.polluted); // Should output: true
```

> [!success]  
> This confirms the entry point is effective for prototype pollution.

---

### **Finding the Sink with DOM Invader**

Click **"Scan for Gadgets"** inside DOM Invader:

- Opens a new tab.
    
- Scans the page for pollution sinks (e.g., dangerous `innerHTML` usage).
    

When scanning completes:

- DOM Invader shows: `elements.innerHTML` as the sink.
    
- Click **"Exploit"** to test payload delivery.
    

> [!danger]  
> **Exam Critical:**  
> If pollution flows to an `innerHTML` sink, **XSS is possible**.  
> This can happen automatically with DOM Invader's exploit button.

---

### **What If DOM Invader Misses the Sink?**

> [!warning]  
> DOM Invader **does not always find the full path** to exploitation.  
> If no sink is found:
> 
> - Manually inspect source code.
>     
> - Set breakpoints in Dev Tools.
>     
> - Trace how user input flows.
>     

Even if the tool fails:

- Pollution might still be present.
    
- You can manually chain the source to a vulnerable sink.
    

---

### **Extra Notes on Version 2 of the Dashboard**

> [!tip]  
> The `/v2` version of the dashboard:
> 
> - Does **not use `dparam`** library.
>     
> - Uses **pure JavaScript**.
>     
> - Is still vulnerable to the same prototype pollution attacks.
>     

Use this version for practice without relying on external libraries.

---

### **Key Takeaways & Exam Reminders**

> [!important]  
> **Exam Tips:**
> 
> - Enable Prototype Pollution in DOM Invader under **Attack Types**.
>     
> - Try both `__proto__` and `constructor.prototype` payloads.
>     
> - Confirm pollution via `console.log({})`.
>     
> - Click **Scan for Gadgets** to find sinks.
>     
> - DOM Invader may find XSS sinks like `innerHTML`.
>     
> - Manual code review is **often necessary** when the tool fails.
>     

> [!success]  
> DOM Invader is a powerful assistant—but understanding prototype pollution manually is essential for consistent success during exams and real-world testing.

---
