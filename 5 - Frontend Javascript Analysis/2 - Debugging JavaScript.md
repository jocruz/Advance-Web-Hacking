# Debugging JavaScript (Browser-Based)

## Summary

This session introduces techniques for browser-based JavaScript debugging in a frontend-focused Node.js application. It demonstrates:

- Setting up the debugging environment using Chrome Developer Tools
    
- Working with breakpoints: manual and event listener-based
    
- Inspecting how JavaScript transforms inputs (e.g., through `escapeHTML`)
    
- Using the **Watch Panel** and **Console** for deeper variable inspection
    
- Exploring real-time analysis for dynamic/minified/obfuscated JS
    
- Installing and using the **Chrome Resource Saver Plugin** to extract and beautify frontend assets for further local analysis

---

## Environment Setup

```bash
# Unzip and navigate into the AWH frontend lab
unzip awh-frontend-lab-1.zip
cd awh-frontend-lab-1

# Open in Visual Studio Code
code .

# Install dependencies and run the app
npm i
node app.js
```

Visit the app in the browser and open **Developer Tools** (`F12`), then configure your workspace:

- **Snap DevTools to the left**
    
- **Run the app on the right**  
    This layout supports efficient side-by-side debugging and analysis.
    

---

## File Structure Overview

- **index.ejs** — Main view file.
    
- **/css/** — Styles (sometimes JS may be hidden here during CTFs).
    
- **/js/** — JavaScript files, e.g., `main.js`.
    

---

## Dynamic JavaScript Debugging

Modern JavaScript apps often use:

- Minified or obfuscated JS
    
- Dynamically executed logic that's hard to reverse statically
    

> [!tip]  
> **Dynamic analysis** (breakpoints, runtime inspection) is often faster than reading entire JS files in CTFs or assessments.

---

## Breakpoints: Manual vs. Event Listener

### 1. **Event Listener Breakpoints**

Access these via the **Developer Tools → Sources → Event Listener Breakpoints** panel.

Useful for catching logic triggered by:

- **Keyboard events** (keydown, keyup, keypress)
    
- **Inputs**
    
- **Clicks, timers, etc.**
    

#### Example:

Search for `timer`, then set a breakpoint on:

```js
setInterval(...)
```

When clicking **Start**, DevTools pauses at this function — useful for identifying logic related to the Pomodoro timer.

---

### 2. **Manual Breakpoints**

Used for direct inspection of logic in user actions like form submission.

#### Example:

Break on `addTodo()` logic:

```js
const text = escapeHTML(userInput);
```

Inspect the transformed payload:

```html
< becomes &lt;
> becomes &gt;
' becomes &#39;
```

> [!note]  
> Hovering over variables after a breakpoint **reveals runtime-transformed payloads**, enabling precise understanding of front-end filters.

---

## Console Inspection

While paused at a breakpoint, switch to the **Console** panel:

```js
text
// View value of the 'text' variable
```

You can interact directly with the JavaScript scope and evaluate expressions.

---

## Watch Panel

### Overview:

Tracks the values of specific expressions or variables over time.

### Usage:

```js
// Add an expression to track
this.todos

// View localStorage todos
JSON.parse(localStorage.getItem("todos"))
```

Break at the delete action to inspect `todos` before and after deletion.

> [!tip]  
> **Use Watch Panel** to monitor variables across multiple breakpoints without re-hovering or console typing.

---

## Saving and Beautifying Code with Chrome Resource Saver

### Why:

Helps retrieve and beautify:

- Minified HTML, CSS, JS
    
- Inline or embedded assets
    
- For full local analysis in VS Code
    

### Steps:

1. Install **Chrome Resource Saver** plugin.
    
2. With the app running, open the plugin.
    
3. Click **Save** to extract all frontend assets.
    

```bash
# Example of unpacked assets
localhost/
└── localhost_3000/
    ├── index.html
    ├── main.js (beautified)
    ├── style.css
```

Re-open these in VS Code for better offline/static analysis.

> [!note]  
> Great for CTFs or source-based bug hunting — lets you inspect the full un-minified logic in your preferred editor.

---

## Tools Mentioned

|Tool / Plugin|Purpose|
|---|---|
|**Chrome DevTools**|Breakpoints, variable inspection, dynamic runtime analysis|
|**Visual Studio Code**|Full source inspection, static analysis|
|**Chrome Resource Saver**|Save & beautify frontend assets for offline exploration|
|**Node.js**|Serves the simple frontend application|
|**npm**|Installs project dependencies|
|**Breakpoints (manual/event)**|Debugging logic through real-time triggers|
|**Watch Panel**|Monitors specific JS expressions across breakpoints|
|**Console Panel**|On-the-fly variable inspection during runtime debugging|

---

## Key Takeaways

> [!tip]  
> When working with obfuscated or minified JS:
> 
> - Focus on **runtime behavior** using breakpoints
>     
> - Use **event listener breakpoints** to avoid digging through logic manually
>     
> - Use the **Watch Panel** to monitor how specific values change
>     
> - Save and **beautify frontend assets** for offline review when available
>     

> [!warning]  
> Always check **non-obvious folders** like `/css/` or `/images/` for JS in CTFs or hidden source files.
