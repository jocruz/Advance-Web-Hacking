
# GraphQL Web Application Security

### Technical Summary for Open-Book Penetration Testing Exam

---

## Overview of GraphQL

GraphQL is a **query language and runtime** for APIs developed by Facebook in 2012 and open-sourced in 2015. It addresses inefficiencies in REST APIs by enabling clients to request **only the exact data they need**, reducing over-fetching and under-fetching.

> [!summary]  
> **GraphQL enables fine-grained data requests via a single endpoint.**  
> It’s commonly used over HTTP and widely adopted by companies like GitHub, Twitter, and Shopify.

---

## GraphQL vs REST

|Feature|REST|GraphQL|
|---|---|---|
|**Endpoints**|Multiple (e.g., `/users`, `/posts`)|Single (e.g., `/graphql`)|
|**Data Fetching**|Fixed; entire resource|Flexible; field-specific and nested|
|**Request Count**|Often multiple requests|One request for all needed data|

> [!info]  
> With GraphQL, one request can fetch a user’s profile, posts, and followers by specifying only the required fields.

---

## Key Components of GraphQL

### 1. Schema

Defines the **types, fields, and relationships** available in the API.

- Think of `type` as a class or object (e.g., `User`).
    
- Fields are properties of that type (e.g., `id`, `name`, `email`).
    
- Relationships can link types (e.g., `User` → `Posts`).
    

```graphql
type User {
  id: ID
  name: String
  email: String
  posts: [Post]
}
```

---

### 2. Queries

Used by clients to **request data**. Flexible and declarative.

- Specify exact fields
    
- Include arguments or filters
    
- Allow nesting for related types
    

Example:

```graphql
{
  user(id: 1) {
    name
    email
    posts {
      title
    }
  }
}
```

> [!note]  
> GraphQL queries return data in the exact shape requested. You only get what you ask for—no more, no less.

---

### 3. Parser (Validator)

Validates queries **before** execution:

- Ensures fields and types exist in the schema
    
- Validates types and structure
    
- Verifies access/authorization (if implemented)
    

> [!warning]  
> **Missing or misconfigured validation may allow over-permissive access** or malformed query attacks.

---

### 4. Resolvers

Backend functions that fetch data for each field.

- Each field in a query maps to a resolver
    
- Can source data from databases, APIs, or services
    
- Executed only after validation
    

> [!info]  
> Nested queries require multiple resolver calls—can impact performance or be exploited in **DoS attacks** via complex/deep queries.

---

## Security Considerations

### Denial of Service (DoS)

Nested or recursive queries can lead to performance issues.

- Mitigate with **query depth limits** and **complexity analysis**
    
- Use **timeout limits** or **rate limiting**
    

### Authorization

GraphQL's flexibility requires **field-level authorization**.

- Ensure each field or type enforces access controls
    
- Avoid assuming top-level query access implies access to nested data
    

### Input Validation

Validate query inputs thoroughly.

- Prevent injection or schema manipulation
    
- Sanitize user-supplied arguments
    

> [!warning]  
> Failure to properly secure GraphQL may expose sensitive relationships, bypass access control, or enable resource exhaustion.

---

## Typical Attack Surface in GraphQL

|Attack Vector|Description|
|---|---|
|Introspection|Reveals full schema; disable or restrict in production|
|Overly Permissive Resolvers|Allow access to unintended fields or objects|
|Deeply Nested Queries|Exploit lack of complexity control for DoS|
|Lack of Rate Limiting|Enables brute force or enumeration|
|Field-Level Access Bypass|Unauthorized access to sensitive fields or types|

> [!summary]  
> **For exams**, check for:
> 
> - Enabled introspection
>     
> - Absence of query depth limits
>     
> - Lack of field-level auth
>     
> - Sensitive fields exposed in schema
>     
> - Deep nesting support
>     

---

## Lab Preparation Reminder

Before testing GraphQL apps:

1. Inspect available endpoints (`/graphql` or similar).
    
2. Use introspection (if enabled) to dump schema:
    
    - Tools: `GraphQL Voyager`, `InQL`, or browser plugins like `Altair`
        
3. Craft custom queries to enumerate users, roles, posts, etc.
    
4. Analyze server response structure for injection opportunities.
    
5. Test nested queries for performance or DoS issues.
    

> [!note]  
> Always check schema exposure via introspection to map potential attack paths.
