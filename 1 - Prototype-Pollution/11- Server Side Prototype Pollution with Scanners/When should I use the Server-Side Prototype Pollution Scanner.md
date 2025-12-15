

> [!question]  
> **"When should I use the Server-Side Prototype Pollution Scanner?"**

---

### ✅ Use the scanner when **all of these are true**:

1. **The application accepts `application/json` requests**
    
    - You're seeing POST, PUT, or PATCH requests with a body that looks like:
        
        ```json
        {
          "user": "john",
          "preferences": {
            "darkMode": true
          }
        }
        ```
        
    - This tells you the backend is likely using **JavaScript-based object merging**.
        

---

2. **User input is being sent in nested structures**
    
    - Example: You can send something like this:
        
        ```json
        {
          "settings": {
            "notifications": true
          }
        }
        ```
        
    - That’s a green flag because **nested JSON** is exactly what lets us hide payloads like `__proto__`.
        

---

3. **You suspect the backend uses JavaScript or Node.js**
    
    - Prototype pollution mainly affects **JavaScript environments**, especially when object merging libraries (like `lodash`, `jQuery`, or `Object.assign`) are used.
        

---

4. **You’ve found an endpoint where your input is echoed back or affects behavior**
    
    - For example:
        
        - A request where you send JSON and it returns a `status` or role you didn’t expect
            
        - A configuration or settings endpoint
            
        - An object like:
            
            ```json
            {
              "user": {
                "name": "John",
                "role": "user"
              }
            }
            ```
            
            could be an **entry point** for injecting `"__proto__": {"role": "admin"}`
            

---

> [!note]  
> **You don’t just run the scanner on random endpoints.**  
> You first identify a promising request (e.g., JSON-based with merge behavior), **then** run the scanner on that request.

---

### 🔍 Example: What a “good” target request might look like

```
POST /api/preferences HTTP/1.1
Content-Type: application/json

{
  "theme": "dark",
  "notifications": true
}
```

That’s a good candidate because:

- It’s JSON.
    
- You control the structure.
    
- The server might be merging it into a config object.
    

→ Send this to **Repeater**, then run the **Full Scan** from the extension.

---

### ❌ Don't use it when:

- The request is `application/x-www-form-urlencoded` with no option to switch to JSON.
    
- You’re testing non-JavaScript backends (e.g., PHP, Python) unless they use JS-like object merging.
    
- You haven’t found any signs of object behavior or settings being changed.
    

---

> [!success]  
> **TL;DR: When to use the extension**
> 
> - When the backend accepts **JSON**
>     
> - When you can send **nested objects**
>     
> - When you suspect JS object merging is happening
>     
> - When you want to check for pollution **without manually crafting every payload**
>     

---

Let me know if you want a checklist template or visual flowchart to help decide when to use it!