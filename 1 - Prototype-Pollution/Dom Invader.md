
## **DOM Invader: Detecting Client-Side Prototype Pollution and DOM-Based XSS**

> [!info]  
> **Overview**  
> DOM Invader is a Burp Suite extension used to identify client-side security vulnerabilities such as DOM-based XSS and prototype pollution. It monitors the browser's DOM for untrusted sources and dangerous sinks, allowing for dynamic testing and payload injection.

---

### **Setting Up DOM Invader**

> [!tip]  
> Before using DOM Invader, ensure the extension is enabled and the correct attack types are activated.

#### Steps:

1. Open **Burp Suite** and go to the **Extensions** tab.
    
2. Locate and **enable the DOM Invader** extension.
    
3. In DOM Invader settings, scroll to **Attack Types**.
    
4. Enable **Prototype Pollution** detection.
    
5. Click **Reload** to activate the setting.
    

---

### **Analyzing the `/v2` Dashboard**

> [!info]  
> The `/v2` version of the dashboard removes the `dparam` library but remains vulnerable to prototype pollution through native JavaScript.

#### How to explore:

- Open your project in **Visual Studio Code**.
    
- Navigate to the `Views` directory and open `dashboard-v2`.
    
- This is an alternate implementation that is still exploitable but written using pure JavaScript.
    

---

### **Using DOM Invader to Detect Prototype Pollution**

1. Launch the target app in a **Chromium browser** proxied through Burp.
    
2. Press `F12` (or `Ctrl + Shift + C`) to open Developer Tools.
    
3. Navigate to the **DOM Invader** tab.
    

#### DOM Invader will automatically detect potential sources, such as:

- `__proto__.property=value`
    
- `constructor.prototype.property=value`
    

> [!success]  
> **Example payload:**
> 
> ```
> http://target.site/?__proto__.injected=true
> ```

To verify if the payload was successful:

```javascript
console.log({}.injected); // Output: true
```

---

### **Scanning for Sinks (Gadgets)**

> [!info]  
> A source by itself is not dangerous unless it reaches a sink like `innerHTML`, `document.write`, or `eval`.

1. In DOM Invader, click **Scan for Gadgets**.
    
2. A new browser tab will open and perform a gadget scan (this may take 30–60 seconds).
    
3. Return to the DOM Invader tab after the scan.
    
4. If a sink is found (e.g., `element.innerHTML`), it will be listed under **Sinks**.
    

> [!tip]  
> Clicking on a sink shows its **stack trace**. This can help trace the input flow and identify the vulnerable function or line of code.

---

### **Exploiting the Sink**

Once a sink is found:

1. Click the **Exploit** button next to the sink in DOM Invader.
    
2. Observe the injected payload in the page or in the console.
    
3. If successful, you will observe the execution of JavaScript (e.g., via a script injection or alert box).
    

---

### **When DOM Invader Fails to Identify Sinks**

> [!warning]  
> Sometimes DOM Invader will find a source but no sinks, or its default payloads won’t work.

In such cases:

- Manually inject payloads into form inputs, URL parameters, or headers:
    
    ```
    ?__proto__.testProperty=example
    ```
    
- Validate by checking:
    
    ```javascript
    console.log({}.testProperty); // example
    ```
    
- Perform manual code review using DevTools:
    
    - Use the **Sources** tab to set breakpoints.
        
    - Trace variable values and control flow from input to function call.
        
    - Confirm if polluted properties affect DOM manipulation or logic flow.
        

> [!tip]  
> Look for conditional statements that rely on polluted properties:
> 
> ```javascript
> if (config.debug) {
>   // Execution flow can be altered by pollution
> }
> ```

---

### **Additional Use Cases in DOM Invader**

- **Inject payloads into forms or input fields**
    
- **Inject payloads into URL parameters**
    
- Useful for testing reflected or stored input points in the DOM
    

---

## **Key Takeaways & Exam Reminders**

> [!important]  
> Review and understand these points carefully—they are critical for the exam.

- **DOM Invader detects client-side vulnerabilities**, including prototype pollution and XSS.
    
- A valid prototype pollution payload:
    
    ```
    ?__proto__.polluted=true
    ```
    
- Verify success with:
    
    ```javascript
    console.log({}.polluted); // true
    ```
    
- Use **Scan for Gadgets** to find potential sinks.
    
- Common sinks include:
    
    - `element.innerHTML`
        
    - `document.write`
        
    - `eval()`
        
- If no sink is found:
    
    - Perform manual analysis using breakpoints and console inspection.
        
    - Trace input values through the application.
        
- **DOM Invader is semi-automated**—some payloads and cases require manual testing.
    

---

Let me know if you'd like a companion page covering **Server-Side Prototype Pollution**, **DOM XSS Remediation**, or **Burp Suite Automation Scripts**.