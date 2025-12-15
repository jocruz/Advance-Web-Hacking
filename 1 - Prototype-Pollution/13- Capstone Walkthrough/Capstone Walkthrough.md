
## **Analyzing Prototype Pollution in the Capstone Challenge**

> [!info]  
> **Overview**  
> This walkthrough simulates a real-world security engagement focused on identifying and exploiting **Prototype Pollution** vulnerabilities. While built-in tools like Burp Suite’s DOM Invader and server-side scanners aid in detection, many vulnerabilities must still be found via **manual code review** and **payload experimentation**.
> 
> This guide explores both **client-side** and **server-side** prototype pollution, highlights bypass techniques, and includes a breakdown of why filters may fail.

---

### **Client-Side Prototype Pollution Analysis**

React-based applications often use third-party libraries that contain prototype pollution risks—especially when merging objects without safeguards.

#### **1. Vulnerable Library in Use**

A known vulnerable library was used, but **no CVE or npm audit warning** appeared.

- This library was intentionally included as a challenge.
    
- Demonstrates how **not all vulnerabilities show up in automated scanners**.
    

---

#### **2. DOM Invader vs Burp Suite Community Edition**

> [!note]  
> **DOM Invader Behavior Varies by Edition**

- DOM Invader did **not detect prototype pollution** in the Community Edition.
    
- DOM Invader **successfully flagged `__proto__` pollution** in Burp Suite Pro:
    
    - Detected: `proto property = value` in the `message` parameter
        
    - Found `elements.innerHTML` as the sink
        

> [!tip]  
> **Payload Example**
> 
> ```json
> { "message": "<script>alert('XSS')</script>" }
> ```

---

#### **3. Weak Filtering in `merge.js`**

The script tried to block `__proto__`, `constructor`, and `prototype`—but failed.

##### **Why the Filter Failed**

- **Only top-level keys were filtered**.
    
- Filter was **not recursive**, meaning nested payloads could bypass it.
    

##### **Bypass Technique**

> [!example]  
> **Bypassed Key Example**
> 
> ```
> "pro" + "to" + "type"
> ```

This bypassed the naive filter and **successfully polluted the prototype**.

---

#### **4. Using Breakpoints to Debug**

> [!tip]  
> **Debugging Filters**  
> Use browser dev tools to:
> 
> - Set breakpoints in the filtering function
>     
> - Inspect how keys are parsed and altered
>     
> - Use fuzzing to explore filter logic
>     

This **helps build working payloads** even when filters or logic are obfuscated.

---

### **Server-Side Prototype Pollution**

The server-side logic included several endpoints. One of them was vulnerable due to improper object merging.

#### **1. Vulnerable Endpoint: `tasks/edit/:id`**

In the code:

```js
defaultsDeep(taskDefaults, req.body)
```

This pattern merges the user input into defaults and is **vulnerable to prototype pollution**.

##### **Payload Example**

```json
{
  "__proto__": { "polluted": true }
}
```

> [!success]  
> **Confirmed**  
> Polluted value appears globally via `Object.prototype`.

---

#### **2. Secure Code Example (Mitigated Version)**

In a previously fixed endpoint (`tasks/create`):

```js
Object.assign({}, obj1, obj2)
```

This usage of `Object.assign` is **not vulnerable** to prototype pollution.

> [!note]  
> Always test to confirm secure behavior, even when using safer methods.

---

#### **3. Modifying User Roles via Pollution**

By targeting a shared object like `role`, we can **elevate privileges** across users.

> [!example]  
> **Payload**
> 
> ```json
> { "__proto__": { "role": "admin" } }
> ```

> [!warning]  
> If successful, **all users become admins** due to prototype inheritance.

---

### **Scanner vs Manual Testing**

> [!note]  
> **Burp Suite Scanner Insights**

- The scanner **misses prototype pollution** with `application/x-www-form-urlencoded`.
    
- Works properly with `application/json`.
    

##### **Scanner Request Behavior**

|Request #|Description|Result|
|---|---|---|
|1|Valid payload|200/302 OK|
|2|Altered value|Confirmed change|
|3|Broken payload|5xx Error Triggered|
|4|Revert changes|200 OK|
|5|Broken payload again|4xx Bad Request|

---

### **Key Takeaways**

> [!warning]
> 
> - Prototype pollution can occur **client-side** and **server-side**.
>     
> - **Naive filtering** (e.g., `key !== '__proto__'`) is insufficient unless **recursive**.
>     
> - **Breakpoints and fuzzing** help understand and bypass complex filters.
>     
> - Always test **multiple content types**, like `application/json` vs `form-urlencoded`.
>     
> - **Dangerous merges** like `defaultsDeep(obj1, obj2)` should be reviewed and tested.
>     


![[Pasted image 20250720013944.png]]