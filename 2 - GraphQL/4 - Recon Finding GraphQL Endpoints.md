# GraphQL Recon & Attack Surface Discovery

## High-Level Summary

This section focuses on discovering GraphQL endpoints during penetration tests or bug bounty engagements. It covers methods to find both known and hidden GraphQL endpoints using tools like Burp Suite, Intruder, custom wordlists, and GraphQL-specific tools like Graphw00f. The module includes practical demonstrations of endpoint fuzzing, error-based enumeration, and server fingerprinting. It also emphasizes the importance of customizing wordlists and checking default framework behaviors.

---

## Table of Contents

- [Overview of Recon Approaches](#overview-of-recon-approaches)
    
- [Manual Discovery Using Burp Suite](#manual-discovery-using-burp-suite)
    
- [Identifying GraphQL Endpoints via Fuzzing](#identifying-graphql-endpoints-via-fuzzing)
    
- [Common Endpoint Naming Patterns](#common-endpoint-naming-patterns)
    
- [Using Burp Suite Intruder](#using-burp-suite-intruder)
    
- [Troubleshooting: URL Encoding Errors](#troubleshooting-url-encoding-errors)
    
- [Tool: Graphw00f](#tool-graphw00f)
    
- [Graphw00f Fingerprinting Output](#graphw00f-fingerprinting-output)
    
- [Apollo Server Default Settings](#apollo-server-default-settings)
    
- [Tips for Custom Wordlists](#tips-for-custom-wordlists)
    

---

## Overview of Recon Approaches

Two perspectives:

1. **Wide scope** – multiple hosts/domains (e.g., bug bounties)
    
    - Use tools like `nmap` and scripting engine for initial detection
        
2. **Narrow scope** – single app focus (e.g., web app pen test)
    
    - Focus on frontend traffic, HTTP history, and passive analysis
        

> [!note]  
> This course focuses on deep-diving into single applications or targets with tighter scopes.

---

## Manual Discovery Using Burp Suite

Steps:

1. Run the app:
    
    ```bash
    npm install
    node app.js
    ```
    
2. Open Burp Suite → Proxy → Open Browser → Navigate to `http://localhost:3000`
    
3. Interact with the app:
    
    - Create a user
        
    - Create a server
        
    - Send messages
        
    - Create/join rooms
        

### Burp Suite Behavior

- Go to **HTTP History**
    
- Look for POST requests to `/graphql`
    
- Burp detects GraphQL and allows you to format/view queries directly
    

> [!info]  
> GraphQL requests appear frequently because the frontend is polling every few seconds.

---

## Identifying GraphQL Endpoints via Fuzzing

Even if the standard `/graphql` endpoint is present, others may exist:

- `/dev/graphql`
    
- `/v1/graphql`, `/v2/graphql`, etc.
    

### Strategy:

1. Create a **wordlist** of common GraphQL endpoints.
    
2. Use an invalid GraphQL query to test if the endpoint exists.
    

Example query:

```graphql
query {
  test
}
```

Valid endpoint → returns:

```json
{
  "errors": [
    {
      "message": "Cannot query field \"test\" on type \"Query\"."
    }
  ]
}
```

Invalid endpoint → returns:

```json
404 Not Found
```

> [!info]  
> Filter responses based on status code, length, or presence of GraphQL-specific errors.

---

## Common Endpoint Naming Patterns

Create a wordlist like:

```
GraphQL Endpoints

`/graphql`

`/graphql/`

`/graphiql`

`/graphiql/`

`/v1/graphql`

`/v2/graphql`

`/v3/graphql`

`/v1/graphiql`

`/v2/graphiql`

`/v3/graphiql`

`/dev/graphql`

`/dev/graphiql`

`/playground`

`/playground/`

`/v1/playground`

`/v2/playground`

`/v3/playground`

`/api/v1/playground`

`/api/v2/playground`

`/api/v3/playground`

`/api/graphql`

`/api/graphql/`

`/api/v1/graphql`

`/api/v2/graphql`

`/api/v3/graphql`

`/api/graphiql`

`/api/graphiql/`

`/api/v1/graphiql`

`/api/v2/graphiql`

`/api/v3/graphiql`

`/api/v1/graphql`

`/api/v2/graphql`

`/api/v3/graphql`

`/dev/v1/graphql`

`/dev/v2/graphql`

`/dev/v3/graphql`

`/dev/v1/graphiql`

`/dev/v2/graphiql`

`/dev/v3/graphiql`

`/graphql/v1`

`/graphql/v2`

`/graphql/v3`

`/console`

`/graphql-console`

`/graphiql-console`

`/graphql-explorer`

`/graphiql-explorer`

`/graphql/playground`

`/graphql/api`

`/graphql-dev`

`/graphql-test`

`/api/graphql/playground`

`/api/graphql/explorer`

`/graphql/admin`

`/graphql/api`

`/graphql/console`

`/graphql/explorer`

`/graphiql/admin`

`/graphiql/console`

`/graphiql/explorer`

`/api/v1/graphql/console`

`/api/v2/graphql/console`

`/api/v3/graphql/console`

`/api/v1/graphql/explorer`

`/api/v2/graphql/explorer`

`/api/v3/graphql/explorer`

`/queries`

`/queries/`

`/graphql-queries`

`/api/graphql-queries`

`/graphql/v1/queries`

`/graphql/v2/queries`

`/graphql/v3/queries`

`/graphql/debug`

`/debug/graphql`

`/debug/graphiql`

`/graphql-introspection`

`/introspection`

`/api/graphql/introspection`

`/graphql/docs`

`/api/graphql/docs`
```

> [!tip]  
> Include trailing slashes as some servers interpret them differently based on configuration.

---

## Using Burp Suite Intruder

1. Send a known GraphQL request to Intruder
    
2. Set the **endpoint path** as the attack position
    
3. Load your GraphQL endpoint wordlist
    
4. Launch attack
    

> [!warning]  
> Make sure **"URL-encode these characters"** is unchecked if you don’t want the payload to be altered.

---

## Troubleshooting: URL Encoding Errors

> [!warning]  
> If you're getting incorrect or no results, check if Burp Intruder is URL-encoding your payload. Disable encoding when testing raw paths.

Example of incorrect behavior:

```
%2Fgraphql → Encoded version of /graphql → Server may reject
```

---

## Tool: Graphw00f

GraphQL server detection and fingerprinting tool.

### Install & Run:

```bash
git clone https://github.com/dolevf/graphw00f.git
cd graphw00f
pip install -r requirements.txt  # optional
python3 main.py --target http://localhost:3000 --detect --fingerprint
```

---

## Graphw00f Fingerprinting Output

- Found endpoints:
    
    - `/graphql` ✅
        
    - Did **not** detect `/dev/graphql` (requires re-scan)
        
- Identified backend:
    
    - `Apollo` (GraphQL engine)
        
- Technologies:
    
    - JavaScript
        
    - Node.js
        
    - TypeScript
        

> [!info]  
> Graphw00f also provides links to CVEs, default behaviors, and documentation for the detected engine.

---

## Apollo Server Default Settings

> [!note]  
> Based on Graphw00f fingerprinting, these are the **default security settings** for Apollo:

|Feature|Default|
|---|---|
|Field suggestions|Enabled|
|Query depth limit|Disabled|
|Query cost analysis|Disabled|
|Introspection|Enabled if not in production|
|Debug mode|Enabled|
|Batch requests|Supported|

> [!warning]  
> These defaults may change. Always verify on the latest Apollo documentation if targeting Apollo-based GraphQL servers.

---

## Tips for Custom Wordlists

> [!tip]  
> Combine known sources like:
> 
> - `SecLists`
>     
> - `Assetnote`
>     
> - `PayloadsAllTheThings`
>     

But always:

- Add custom paths based on application behavior
    
- Expand on legacy patterns (`/v1`, `/admin`, `/internal`, etc.)
    

> [!info]  
> Custom paths can uncover endpoints that others miss—especially helpful in competitive bug bounty environments.
