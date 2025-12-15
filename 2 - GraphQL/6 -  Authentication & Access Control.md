# GraphQL Authentication & Access Control

## Summary

GraphQL does **not** have built-in authentication or access controls. This means that developers must implement these controls manually, resulting in a wide variety of implementations. Existing attacks like JWT manipulation, brute forcing, and access control testing still apply. Specific GraphQL features such as query batching introduce new attack vectors that can bypass rate limiting and web application firewalls (WAFs).

> [!warning]  
> **Authentication ≠ Authorization**  
> Authentication = who you are. Authorization = what you’re allowed to do.

## Authentication & Access Control in GraphQL

- GraphQL does **not** include native authn/authz; these are implemented by developers per app.
    
- Common controls (e.g., JWT-based login systems) still apply.
    
- Key risks:
    
    - Broken access control (BOLA, IDOR, BAFLA)
        
    - Rate limiting bypass
        
    - Brute force vulnerabilities via query batching
        

> [!info]  
> GraphQL is vulnerable to **the same categories of attacks** as REST APIs, with some unique attack surfaces (e.g., batching, introspection, nested queries).

---

## Query Batching & Rate Limit Bypass

**Batching** allows multiple GraphQL queries or mutations to be sent in a single HTTP request by wrapping them in an array of query objects.

- Originally designed to improve performance by reducing the number of HTTP requests.
    
- Attackers can abuse this feature to:
    
    - Bypass rate-limiting protections.
        
    - Circumvent naive Web Application Firewall (WAF) rules.
        
    - Perform credential stuffing or brute-force attacks more efficiently.
        

> [!note]  
> Batched queries are not part of the GraphQL specification but are commonly supported by client libraries and servers for performance optimization.

### Why This Matters for Authentication and Access Control

Applications often apply rate limits on a per-request basis. When attackers send multiple login attempts in a **single request**, those rate limits are not triggered.

> [!note]  
> This technique is especially useful in brute-force scenarios, where sending multiple login mutations in a single batch allows attackers to avoid detection while increasing speed and efficiency.

### Example Batched Login Request

```json
[
  {
    "query": "mutation { login(email: \"user1@example.com\", password: \"password1\") }"
  },
  {
    "query": "mutation { login(email: \"user2@example.com\", password: \"password2\") }"
  },
  {
    "query": "mutation { login(email: \"user3@example.com\", password: \"password3\") }"
  }
]
```

Each login attempt is included in one HTTP request, making it harder for simple protections to detect brute-force activity.

> [!note]  
> Security controls must account for the **number of operations per request**, not just the number of requests, when defending GraphQL endpoints.

---
### Practical Walkthrough

1. Create a user and login via browser.
    
2. Observe token returned in signup response.
    
3. Capture login request:
    
    - Contains email, password
        
    - Login query is visible
        
4. Copy login mutation and variables into a text editor.
    
5. Format into a batched query (array of objects):
    

```json
[
  { "query": "mutation { login(email: \"test@example.com\", password: \"123456\") }", "variables": { ... } },
  { "query": "mutation { login(email: \"test2@example.com\", password: \"123456\") }", "variables": { ... } }
]
```

6. Send via Burp Repeater or Intruder.
    

### Rate Limiting Bypass with Burp

- Intruder limitations:
    
    - Can't dynamically map different payloads to separate queries easily
        
- Workaround:
    
    1. Split wordlist: `split -l 3 passwords.txt part-`
        
    2. Use Pitchfork mode with multiple payload sets:
        
        - Payload set 1 → part-aa
            
        - Payload set 2 → part-ab
            
        - Payload set 3 → part-ac
            
    3. Disable URL encoding
        
    4. Grep for `token` in Burp response to detect successful logins
        

> [!tip]  
> This sends **1 HTTP request with 3 login attempts**, evading naive rate-limiting mechanisms.

## Broken Access Control

### IDOR (Insecure Direct Object Reference)

- Exploiting access to data tied to another user by modifying object IDs in requests.
    

### BAFLA (Broken Function Level Authorization)

- Accessing functionality meant for another role (e.g., creating/deleting objects without permission).
    

> [!example]  
> In testing, sending `query { getMessages(roomId: 3) }` returned data from a room not owned by the user. This is a classic IDOR.

### Vertical vs Horizontal Privilege Escalation

- **Vertical**: Gaining higher-level permissions (e.g., normal user → admin)
    
- **Horizontal**: Accessing peer data (e.g., another user's inbox)
    

> [!info]  
> GraphQL’s flexibility **increases the risk** of access control flaws because developers must enforce logic at resolver level per field or mutation.

## Methodology

1. Run introspection query to identify schema
    
2. Review all queries/mutations for fields like `id`, `roomId`, `userId`, etc.
    
3. Test queries with different values to identify:
    
    - IDOR/BOLA: Reading other user data
        
    - BAFLA: Performing unauthorized actions
        
4. Check JWT to confirm current user’s role/identity before testing
    
5. Use front-end to verify if the backend response aligns with allowed UI actions
    

## Tools

### Burp Suite

**Purpose**: Intercept and manipulate GraphQL requests

- Use Repeater to manually test queries
    
- Use Intruder with Pitchfork mode for batching attacks
    
- Grep Match: Add `token` as filter to find successful logins
    

### CrackQL

**Purpose**: Automate brute force/password spraying for GraphQL APIs

- Clone: `git clone https://github.com/nicholasaleks/CrackQL`
    
- Install: `pip install -r requirements.txt`
    
- Supports:
    
    - Password spraying
        
    - 2FA OTP bypass
        
    - User account enumeration
        

```bash
python crackql.py --url http://localhost:3000/graphql --query login.graphql --input users.txt --passwords passwords.txt
```

> [!note]  
> You must modify queries to match the application schema.

---