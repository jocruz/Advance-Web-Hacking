# Using AI to Review Frontend Code Securely and Efficiently

In this video, we explore how AI—specifically tools like ChatGPT—can assist in reviewing frontend JavaScript code for both understanding and security assessments.

## Example 1: Understanding Debounce Functions

We begin with a common frontend function: a debounce handler. Debouncing is used to limit how frequently a function (like an event handler) can be called within a time frame.

The tricky part here is understanding the use of closures and maintaining the correct execution context.

If you come across unfamiliar code like this and you're unsure about how it works—especially when functions become longer or more complex—you can use AI tools like ChatGPT to help break it down.

> **Note:** Never copy and paste sensitive backend code into AI tools. Only use public-facing frontend code that’s already visible to anyone.

You can paste the code into ChatGPT with a prompt like:

```plaintext
Can you explain this code to me?
```

Or, for more specific insights:

```plaintext
Can you review this code for any security issues and highlight them?
```

Being specific with your question improves the quality of the response.

## 📌 High-Value Tip

> **🟦 Callout: Use Context Wisely**  
> When asking AI for help:
> 
> - Avoid sharing too much code at once—it may overwhelm the model.
>     
> - Avoid sharing too little—it may lack critical context.
>     
> - Focus on just enough relevant code (e.g., one function or a small group of dependent functions).
>     

## Example 2: Reviewing for XSS

Here's another function from the same code review session. This one dynamically injects user input into the DOM using `innerHTML`.

```javascript
element.innerHTML = userInput;
```

This should raise an immediate red flag: unfiltered user input being written directly to the DOM can lead to Cross-Site Scripting (XSS).

Instead, use safer alternatives like:

```javascript
element.textContent = userInput;
```

Or sanitize the HTML if necessary using libraries like DOMPurify:

```javascript
element.innerHTML = DOMPurify.sanitize(userInput);
```

We submitted this snippet to ChatGPT with the following prompt:

```plaintext
Can you review this code from a security perspective?
```

It correctly flagged the XSS issue and recommended using `textContent` or sanitizing the input. These suggestions matched our expectations.

> **🟦 Callout: Watch for InnerHTML Usage**  
> Any time you see `.innerHTML = userInput`, investigate further. Make sure the input is properly sanitized or consider using `.textContent` instead.

## Example 3: Dynamically Loading Scripts

The final example involves loading external scripts dynamically:

```javascript
const script = document.createElement("script");
script.src = userProvidedURL;
document.head.appendChild(script);
```

This can be dangerous if the `userProvidedURL` is not validated. Malicious users could inject scripts from untrusted sources.

Mitigation steps:

- Implement a strict whitelist of allowed script domains.
    
- Apply a robust Content Security Policy (CSP).
    

> **🟦 Callout: Enforce a Content Security Policy (CSP)**  
> A CSP limits the damage that can be done by injected or untrusted scripts. It's essential for dynamic script loading scenarios.

## Final Advice

When using AI tools to help you with code analysis:

- Be **specific** in your questions.
    
- **Limit** the code snippet to what’s relevant.
    
- **Add** context if the snippet relies on external variables or logic.
    
- **Avoid** feeding AI large scripts or full-page JavaScript files. Break it down function by function.
    
- Always **test and validate** the AI's suggestions—don't rely blindly on its recommendations.
    

By integrating AI into your code review process responsibly, you can significantly streamline your workflow and catch issues earlier.

---

Let me know if you'd like this formatted into a Markdown, Notion page, or Obsidian note next.