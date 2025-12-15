# Using AI for Frontend Code Review

## Summary

This session discusses how **AI tools (like ChatGPT)** can assist in reviewing and analyzing frontend JavaScript code, especially during penetration testing or when facing unfamiliar functions. You’ll learn:

- When and how to use AI tools like ChatGPT responsibly
    
- Why you should be specific in your prompts
    
- The risks of analyzing code out of context
    
- How to leverage AI to identify **security issues** (e.g., XSS, script injection)
    
- Best practices for breaking code into **small snippets**
    
- The importance of **testing AI suggestions**
    

---

## Example 1: Reviewing a Debounce Function

Debounce functions are often used to:

- Limit how frequently an event handler executes
    
- Prevent performance issues in UI elements
    

### Key Code Behavior:

- Uses **closures** to store a `timeout` variable
    
- Maintains correct `this` context
    
- Executes a delayed version of a function only if no new calls are made during the delay
    

> [!note]  
> ChatGPT explains such functions clearly when asked _"Can you explain this code?"_ — helpful for uncommon patterns or unfamiliar syntax.

---

## Prompting AI Effectively

> [!tip]  
> Be specific with prompts:
> 
> - Ask _"Can you explain this code?"_ for general understanding
>     
> - Ask _"Can you review this code for security issues?"_ for vulnerability detection
>     

---

## Security Review with AI

### Example Code: Using `innerHTML`

```js
element.innerHTML = userInput;
```

#### Risk:

- Directly injecting **user-supplied input** into the DOM using `innerHTML` risks **Cross-Site Scripting (XSS)**.
    

#### ChatGPT Suggests:

- Replace with:
    
    ```js
    element.textContent = userInput;
    ```
    
- Or use a sanitizer:
    
    ```js
    element.innerHTML = DOMPurify.sanitize(userInput);
    ```
    

> [!warning]  
> AI may not catch that `userInput` was previously sanitized — **context matters**.

---

## Avoiding AI Misuse

- Don’t paste sensitive **backend** code into public AI tools.
    
- Do use it for **public-facing frontend code**, like:
    
    - JS in browser dev tools
        
    - Client-side logic in HTML files
        

> [!note]  
> Providing **too little code** gives inaccurate feedback.  
> Providing **too much code** causes information overload and vague answers.

---

## Best Practice for AI-Aided Review

1. **Isolate small, functional units** (e.g., a single function or tightly connected pair)
    
2. Ask for both:
    
    - Code explanation
        
    - Security review
        
3. Consider edge cases: Was this input already sanitized upstream?
    
4. Validate the AI's recommendations manually or through testing.
    

---

## Example 2: Loading Scripts Dynamically

```js
let script = document.createElement('script');
script.src = userProvidedUrl;
document.body.appendChild(script);
```

### Risk:

- If `userProvidedUrl` is not validated, you may load **malicious scripts**
    

### Mitigations:

- **Whitelist allowed domains**
    
- Use **Content Security Policy (CSP)** to restrict script sources
    

> [!tip]  
> Even when AI suggests mitigation, **test it** — don’t blindly trust automated fixes.

---

## Final Guidelines

|Good Practice|Why It Matters|
|---|---|
|Use **specific prompts**|Avoids vague or misleading feedback|
|Provide **just enough context**|Too little = incorrect advice; too much = overload|
|Keep code snippets **small and scoped**|Easier for AI to analyze and explain|
|Always **test AI suggestions**|Ensure behavior and security improvements|
|Never upload **sensitive backend code**|Avoid data leakage risks|

---

## Key Takeaways

> [!tip]  
> Use AI to speed up your frontend security reviews — especially when exploring unfamiliar logic or validating edge cases.

> [!note]  
> AI won’t replace human judgment. Always **test**, **validate**, and **contextualize** any suggestions it makes.

> [!warning]  
> Avoid trusting AI with frontend security analysis **without understanding the context**. Misleading recommendations can introduce vulnerabilities.

---

## Tools Mentioned

|Tool|Use Case|
|---|---|
|**ChatGPT (Free)**|Explain, analyze, or review code snippets|
|**DOMPurify**|Sanitize HTML safely before using `innerHTML`|
|**CyberChef**|(Referenced previously) Decode base64, binary|
|**Content Security Policy (CSP)**|Limit script source domains|
