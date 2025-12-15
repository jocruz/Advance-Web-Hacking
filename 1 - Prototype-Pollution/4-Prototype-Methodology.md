

## Understanding Sources, Sinks, and Gadgets in Prototype Pollution

> [!info]  
> **Overview**  
> This section explains the three critical components required to successfully exploit prototype pollution vulnerabilities: sources, sinks, and gadgets. Understanding how these elements work together helps you recognize exploitation paths such as authentication bypass, cross-site scripting (XSS), or even remote code execution (RCE).

---

### 1. **What Are Sources?**

- A **source** is any user-controlled input that can inject arbitrary properties into the prototype of an object.
    
- Common sources include:
    
    - URL query parameters
        
    - JSON payloads
        
    - Messages or data passed between web app components
        
- These are often merged into objects via functions like `Object.assign()` or lodash’s `merge()`.
    

> [!tip]  
> Look for parameters like `__proto__`, `constructor.prototype`, or `prototype` in inputs. These are signs of potential pollution sources.

---

### 2. **What Are Sinks?**

- A **sink** is a place in the code where polluted properties are used in a dangerous or exploitable way.
    
- Examples include:
    
    - Using a polluted property in a conditional (`if (user.isAdmin)`)
        
    - DOM manipulation based on polluted values
        
    - Executing values via `eval()` or similar
        

> [!warning]  
> A sink is dangerous when it allows prototype-polluted properties to affect application logic without validation.

---

### 3. **What Are Gadgets?**

- A **gadget** is a key property or function that enables exploitation — often because it is:
    
    - Trusted blindly in the application logic
        
    - Used without validation
        
    - Inherited from the polluted prototype chain
        
- Examples:
    
    - `isAdmin`
        
    - Configuration flags (`debug`, `logLevel`)
        
    - Dynamically assigned permissions or roles
        

> [!tip]  
> A gadget doesn’t cause harm alone — it must be used by the app (via a sink) **after** it has been introduced (via a source).

---

### Example Walkthrough

#### **Scenario:**

- The query string includes `__proto__` with a property like `isAdmin=true`.
    
- This value is merged into a user object using `Object.assign()` or similar.
    
- The app checks `user.isAdmin` to authorize admin access.
    

#### **Result:**

- Since `isAdmin` is not explicitly set on the user object, the app looks up the prototype chain.
    
- It finds `isAdmin=true` on `Object.prototype`, allowing unauthorized access.
    

#### **Key Components:**

- **Source:** The query string input using `__proto__`
    
- **Sink:** The conditional `if (user.isAdmin)`
    
- **Gadget:** The `isAdmin` property, used without checking whether it exists on the object directly
    

---

## Step-by-Step Checklist: How to Exploit Prototype Pollution

> [!note]  
> This is your go-to methodology when analyzing applications for prototype pollution in both client-side and server-side contexts.

---

### Step 1: Identify Potential Sources

- Look for user inputs that could be merged into objects:
    
    - Query strings
        
    - JSON payloads
        
    - Web messages or local storage values
        
- Check if these inputs are passed to functions like:
    
    - `Object.assign()`
        
    - `$.extend()`
        
    - `lodash.merge()` or similar
        

---

### Step 2: Test for Prototype Pollution

- Try injecting via:
    
    - URL: `?__proto__[isAdmin]=true`
        
    - JSON: `{"__proto__": {"isAdmin": true}}`
        
- Use the browser console or debugger to inspect new object behavior:
    
    ```js
    const obj = {};  
    console.log(obj.isAdmin); // Should be true if pollution worked
    ```
    

---

### Step 3: Locate Sinks

- Review how the app uses the potentially polluted property.
    
- Common sinks include:
    
    - `eval()`, `document.createElement()`, `innerHTML`
        
    - Conditional logic like `if (user.isAdmin)`
        
- Use DevTools to log or break on usage of the polluted property.
    

---

### Step 4: Find Gadgets

- Look for application properties or settings used without validation.
    
- Good candidates:
    
    - Access control flags (e.g., `role`, `isAdmin`)
        
    - Feature toggles (`debug=true`, `enableBeta`)
        
    - Configuration objects merged dynamically
        

---

### Step 5: Craft and Deploy Payload

- Combine source + gadget + sink into an exploit:
    
    - Modify configuration
        
    - Escalate privileges
        
    - Inject scripts
        
- Example:
    
    ```json
    {
      "__proto__": {
        "isAdmin": true
      }
    }
    ```
    

---

## Key Takeaways

- **Sources** are entry points for injecting malicious properties.
    
- **Sinks** are where these properties get used unsafely.
    
- **Gadgets** are the exploited properties that interact with sinks to trigger unexpected behavior.
    
- The full exploitation path requires all three elements working together.
    

> [!success]  
> **Methodology Tip:**  
> Don’t just test for prototype pollution — trace how polluted properties are used. Even if pollution is successful, without a sink and a gadget, it may not be exploitable.

