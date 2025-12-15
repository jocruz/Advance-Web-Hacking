# GraphQL Introspection – Obsidian Notes

## High-Level Summary

This session explains GraphQL introspection—what it is, why it's a feature, and how attackers can abuse it. It explores how introspection reveals schema details such as types, queries, and mutations, making it a powerful tool for API reconnaissance and exploitation. The transcript walks through hands-on usage via Burp Suite, InQL Scanner, and GraphQL Voyager, emphasizing how to detect if introspection is enabled, how to work around restrictions, and how to manually extract schema details when introspection is disabled.

---

## Table of Contents

- [Understanding GraphQL Introspection](#understanding-graphql-introspection)
    
- [Risks of Introspection in Production](#risks-of-introspection-in-production)
    
- [Testing Introspection](#testing-introspection)
    
- [Using Burp Suite for Introspection Queries](#using-burp-suite-for-introspection-queries)
    
- [Analyzing and Using Schema Data](#analyzing-and-using-schema-data)
    
- [GraphQL Voyager](#graphql-voyager)
    
- [Bypassing Keyword Filtering](#bypassing-keyword-filtering)
    
- [InQL GraphQL Scanner](#inql-graphql-scanner)
    
- [Disabling/Enabling Introspection in Code](#disablingenabling-introspection-in-code)
    
- [Tools Reference](#tools-reference)
    

---

## Understanding GraphQL Introspection

> [!info]  
> **Introspection is a feature, not a vulnerability.**  
> However, if left enabled in production, it can provide attackers with the full API map and expose critical attack surfaces.

- GraphQL introspection allows querying the schema itself.
    
- Useful in development for debugging and tooling (e.g. GraphQL Playground).
    
- Allows viewing types, objects, queries, and mutations.
    

---

## Risks of Introspection in Production

- Reveals internal schema, hidden endpoints, legacy queries, or overly permissive fields.
    
- Can enable targeted attacks like:
    
    - Authorization bypass
        
    - Sensitive data exposure
        
    - Denial of Service (via circular queries)
        

> [!warning]  
> Introspection should be disabled in production unless there is a strong reason to keep it enabled.

---

## Testing Introspection

### Manual Test Strategy

- Send a basic introspection query.
    
- Evaluate the server's response:
    
    - If successful → Introspection is enabled.
        
    - If blocked or error → May be filtered or disabled entirely.
        

### Default Behavior

- Many frameworks (e.g. Apollo Server, Hasura) have introspection enabled by default.
    
- May be disabled based on environment variables (e.g., production mode).
    

---

## Using Burp Suite for Introspection Queries

Steps:

1. Run app (`node app.js`)
    
2. Log in through the web interface (localhost:3000)
    
3. In Burp Suite:
    
    - Go to `HTTP History`
        
    - Find a GraphQL request
        
    - Right-click → `GraphQL` → `Set Introspection Query`
        
    - Send the request
        

### Outcome:

- If enabled → JSON schema returned
    
- If disabled → Error message or blocked keywords
    

> [!tip]  
> Use `Save GraphQL queries to sitemap` in Burp to organize discovered queries and mutations.

---

## Analyzing and Using Schema Data

- Review the sitemap in Burp to find query/mutation names.
    
- Send useful entries to Repeater (`Ctrl+R`) for further testing.
    

Example mutation:

```graphql
mutation {
  createServer(name: "Alex's second server") {
    id
  }
}
```

Example query:

```graphql
query {
  me {
    id
    username
    email
  }
}
```

Use these to:

- Check for injection vulnerabilities
    
- Test access controls (e.g., delete another user’s server)
    
- Discover hidden or legacy fields
    

---

## GraphQL Voyager

Steps:

1. Go to [https://graphql-kit.com](https://graphql-kit.com)
    
2. Open **GraphQL Voyager**
    
3. Paste the **introspection JSON output**
    
4. Visualize top-level queries and nested types
    

> [!info]  
> Voyager is useful for identifying nested query chains and potential information disclosure paths.

Example use:

- Start at `query { me }`
    
- Navigate to nested `servers → messages → rooms`
    
---

## Bypassing Keyword Filtering

Some GraphQL implementations attempt to block introspection by filtering out certain keywords like `schema`.

### Standard Introspection Query Using `__schema`:

```graphql
{
  __schema {
    types {
      name
    }
  }
}
```

This query requests all the types (objects) defined in the schema.

> [!warning]  
> If the application blocks queries containing `__schema`, introspection may still be possible through alternative methods like the `__type` query below.

### Alternative Query Using `__type`:

```graphql
{
  __type(name: "User") {
    name
    fields {
      name
    }
  }
}
```

This retrieves all the fields for a specific object type (e.g., `User`).

> [!info]  
> Even if `__schema` is filtered, the server may still allow queries using `__type`, which can be fuzzed with common object names.

---
## InQL GraphQL Scanner

Burp Suite Extension: `InQL GraphQL Scanner`

### How to Use:

1. Install from Burp App Store (`InQL`)
    
2. Restart Burp if necessary
    
3. Go to `InQL` tab in Burp
    
4. Enter target URL (e.g., `http://localhost:3000/graphql`)
    
5. Click **Analyze**
    

> [!info]  
> InQL will parse all queries and mutations, format them, and allow you to send them to Repeater or Intruder.

### Features:

- Works with live endpoints or JSON schema files
    
- Helps when introspection is disabled but a schema file is available
    
- Can generate batch attacks (covered in later sections)
    

---

## Disabling/Enabling Introspection in Code

In **Apollo Server**:

```js
const server = new ApolloServer({
  typeDefs,
  resolvers,
  introspection: false, // or true
});
```

To enable introspection:

- Set `introspection: true`
    
- Restart the server
    

> [!tip]  
> Disabling introspection is not enough. Ensure sensitive queries are also properly access-controlled.

---

## Injection Possibilities Revealed Through Introspection

> [!danger]  
> **SQL Injection or similar attacks may be possible** when introspection reveals input-based mutations like `joinServerByInviteCode`.

When introspection is enabled, it exposes all available queries and mutations, including ones not directly linked to frontend functionality. This visibility allows attackers to test inputs for injection vulnerabilities.

### Example Scenario:

- The `joinServerByInviteCode(inviteCode: String)` mutation is discovered.
    
- Attacker tests:
    

- `mutation {   joinServerByInviteCode(inviteCode: "1234' OR '1'='1") {     id   } }`
    
- Even if SQL injection is not possible, logic flaws or improper validation could allow unauthorized access to resources.
    

This highlights the importance of **carefully testing input-bearing fields** for:

- SQL Injection
    
- NoSQL Injection
    
- GraphQL Injection
    
- Broken Access Control
    

> [!tip]  
> Mutations like `deleteServer(id: ID)` or `joinServerByInviteCode(code: String)` are especially interesting targets for fuzzing, injection testing, and permission validation.


---
## Tools Reference

### 1. **Burp Suite**

- Used to intercept and manipulate HTTP GraphQL requests.
    
- Offers GraphQL support via extensions (e.g., `Set Introspection Query`, `Save to Sitemap`).
    
- Useful for replaying queries and performing injections.
    

### 2. **GraphQL Voyager**

- Visualizer for GraphQL schema.
    
- Input: Introspection schema JSON
    
- Output: Graph-style view of queries, types, and relations
    
- Website: [https://graphql-kit.com](https://graphql-kit.com)
    

### 3. **InQL GraphQL Scanner**

- Burp Suite extension
    
- Analyzes GraphQL endpoints and generates formatted queries/mutations
    
- Useful for schema exploration, even without built-in syntax highlighting
    
- Can perform:
    
    - Query generation
        
    - Batch attacks (future section)
        
- Download: Burp App Store → `InQL`
    

> [!note]  
> When introspection is disabled, InQL will prompt for a schema file instead.
