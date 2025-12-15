## GraphQL Cheat Sheet – Open Book Reference

---

##  Key Definitions

> [!info]  
> These are the basic building blocks of a GraphQL API. Understand these to make sense of introspection queries and schema exploration.

---

### Summary Table

|Concept|Role|Equivalent in REST or SQL|
|---|---|---|
|Schema|API structure definition|None (conceptually closest to OpenAPI spec)|
|Type|Custom object definition|Table schema / class|
|Query|Read operation|GET / SELECT|
|Mutation|Write/update/delete operation|POST/PUT/DELETE / INSERT/UPDATE|
|Field|Specific data returned|Column in SQL / JSON field|

### Schema

The overall structure of the GraphQL API. It defines what data is available, how it's organized, and how clients can interact with it.  
Includes types, queries, mutations, and their relationships.

### Query

A read-only operation. Used to **fetch data** from the server.  
Example: `getUser`, `me`, `accessKeys`.

### Mutation

A write operation. Used to **change data** on the server.  
Example: `createUser`, `updateServer`, `deleteRoom`.

### Type

Defines the shape of an object in the schema.  
Example: a `User` type might have fields like `id`, `username`, and `email`.

### Field

An individual piece of data on a type.  
Example: `username` is a field on the `User` type.

---

## Introspection Queries

> [!info]  
> Introspection allows you to ask a GraphQL API about its structure: types, queries, mutations, and fields. Useful for schema enumeration and discovering attack surface.

---

### 1. Test if introspection is enabled

```graphql
{
  __schema {
    types {
      name
    }
  }
}
```

> [!note]  
> This fetches all defined types (like `User`, `Server`, `Query`, etc.). If the server blocks this, introspection may be disabled or filtered.

---

### 2. List all fields of a specific type (like an object)

```graphql
{
  __type(name: "User") {
    fields {
      name
    }
  }
}
```

> [!info]  
> Returns all fields for the `User` type (e.g., `id`, `username`, `email`). Use this when you want to explore what attributes an object has.

---

### 3. List all top-level queries

```graphql
{
  __type(name: "Query") {
    fields {
      name
    }
  }
}
```

> [!tip]  
> Useful to discover readable data endpoints like `users`, `me`, `accessKeys`. If this is blocked or returns null, you may need to guess query names.

---

### 4. List all mutations

```graphql
{
  __type(name: "Mutation") {
    fields {
      name
    }
  }
}
```

> [!note]  
> Reveals operations that **change** data, such as `createUser`, `deleteServer`, `updateAccessKey`.

---

## Manual Querying

### ## Example: Querying current user

```graphql
query {
  me {
    id
    username
    email
  }
}
```

> [!info]  
> This is a typical query to retrieve data for the currently authenticated user.

---

### Example: Creating an object (mutation)

```graphql
mutation {
  createServer(name: "test-server") {
    id
    name
  }
}
```

> [!info]  
> Use this format when testing mutation functionality (create, update, delete actions).

---

## Hidden Field Discovery

> [!tip]  
> If `createX`, `updateX`, `deleteX` mutations exist, test for a query like `X` or `Xplural`. For example:

```graphql
query {
  accessKeys {
    id
    key
  }
}
```

> [!note]  
> Even if not visible in introspection, queries may still work if you guess the correct name.

---

## Bypassing Filters

### ## If `__schema` is blocked, try `__type`

```graphql
{
  __type(name: "Server") {
    name
    fields {
      name
    }
  }
}
```

> [!warning]  
> This is a bypass for partial introspection blocks. Use common type names (User, Server, Room) to fuzz manually.

---

## Tools Reference (for live testing)

- **Burp Suite**  
    Use “GraphQL → Set Introspection Query” to send schema queries.
    
- **InQL Scanner (Burp Extension)**  
    Automatically parses schema, generates query/mutation templates, supports batch testing.
    
- **GraphQL Voyager**  
    Visualizes schema structure using introspection JSON. Useful for mapping nested objects.
