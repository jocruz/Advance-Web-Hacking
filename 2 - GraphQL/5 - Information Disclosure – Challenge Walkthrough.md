# GraphQL Information Disclosure – Challenge Walkthrough

## High-Level Summary

This walkthrough demonstrates how to fuzz a GraphQL endpoint to discover sensitive fields like `email` and `password`, even when introspection is disabled. It leverages **field suggestions**, **field stuffing**, and introduces **GraphQL aliases** to bypass WAF filtering. The session wraps up by emphasizing the value of checking **mutations** for misconfigurations and broken access control vulnerabilities.

---

## Table of Contents

- [Fuzzing Types with Burp Intruder](#fuzzing-types-with-burp-intruder)
    
- [Generating Wordlists with ChatGPT](#generating-wordlists-with-chatgpt)
    
- [Analyzing Field Suggestion Responses](#analyzing-field-suggestion-responses)
    
- [Validating Subfield Requirements](#validating-subfield-requirements)
    
- [Field Stuffing for Sensitive Data](#field-stuffing-for-sensitive-data)
    
- [Advanced Data Traversal via Nested Queries](#advanced-data-traversal-via-nested-queries)
    
- [Bypassing WAF with Aliases](#bypassing-waf-with-aliases)
    
- [Don't Forget to Check Mutations](#dont-forget-to-check-mutations)
    

---

## Fuzzing Types with Burp Intruder

> [!info]  
> Use Burp Intruder to automate guessing of type names when introspection is disabled.

1. In Burp, send a GraphQL request to **Intruder**.
    
2. Load a wordlist like `types.txt`:
    
    - Uppercase: `User`, `Post`
        
    - Lowercase: `user`, `post`
        
    - Singular & plural: `user`, `users`
        
3. Uncheck **"URL-encode these characters"** to avoid corrupted payloads.
    
4. Start the attack (slow on Community Edition).
    

Example request payload:

```graphql
query {
  user
}
```

---

## Generating Wordlists with ChatGPT

> [!tip]  
> Use ChatGPT to expand object/type name guesses based on app context.

Prompt example:

> “Here are types I've already discovered: User, Post. The app is a social media platform. Can you suggest more types (singular and plural, uppercase and lowercase)?”

Incorporate:

- Domain-specific terms (`match`, `team`, `profile`, `follower`)
    
- Patterns observed in existing fields
    
- Common GraphQL naming conventions
    

---

## Analyzing Field Suggestion Responses

Field suggestions help identify valid field names even if you submit incorrect ones.

Example response:

```json
{
  "errors": [
    {
      "message": "Cannot query field 'user' on type 'Query'. Did you mean 'users'?"
    }
  ]
}
```

> [!info]  
> Burp Intruder lets you detect these messages quickly by comparing content length or grepping for `"Did you mean"` in the response body.

---

## Validating Subfield Requirements

When querying plural types (e.g., `users`), GraphQL may require subfield selection.

### Example:

```graphql
query {
  users
}
```

Response:

```
Field 'users' of type '[User]' must have a selection of subfields.
Did you mean 'users { ... }'?
```

### Fix:

```graphql
query {
  users {
    id
  }
}
```

---

## Field Stuffing for Sensitive Data

> [!warning]  
> Field stuffing is especially useful when introspection is disabled or field names are unknown.

### Example:

```graphql
query {
  users {
    name
    username
    email
    password
  }
}
```

Observed response:

- Some fields throw errors
    
- `password` field **returns data**!
    

> [!danger]  
> Password hashes and email addresses were leaked from the `users` object—**a critical finding**.

---

## Advanced Data Traversal via Nested Queries

Sometimes sensitive fields are not directly exposed under the root type.

Example strategy:

```graphql
query {
  posts {
    author {
      email
      password
    }
  }
}
```

> [!info]  
> Leverage nested objects like `posts → author → email/password` to find deeper leaks.

---

## Bypassing WAF with Aliases

If your queries are being blocked or filtered based on sensitive field names like `password`, use **GraphQL aliases** to mask them.

### Example:

```graphql
query {
  u: username
}
```

Response:

```json
{
  "u": "cheese"
}
```

> [!tip]  
> Use aliases to:
> 
> - Obfuscate queries from WAF rules
>     
> - Remove sensitive words from traffic logs
>     
> - Reformat responses for analysis
>     

---

## Don't Forget to Check Mutations

> [!note]  
> Most people focus on queries—but **mutations** are often a goldmine for exploitation.

### Mutation Discovery Example:

```graphql
mutation {
  createUser {
    id
  }
}
```

This can reveal:

- Account creation functions
    
- Potentially unprotected logic
    
- Entry points for broken access control
    

> [!warning]  
> Failing to test mutations may mean missing major vulnerabilities like privilege escalation or unauthorized account creation.

---

