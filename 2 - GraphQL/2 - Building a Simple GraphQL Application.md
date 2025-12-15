# GraphQL Hands-On Lab: Schema, Resolvers, and Server Setup

### Technical Summary for Open-Book Penetration Testing Exam

---

## Project Setup

A minimal GraphQL backend was created to simulate a chat application.

### Directory Structure

```
/chat-app
  ├── app.js               # Main entry point
  └── /graphql
      ├── schema.js        # Type definitions
      └── resolvers.js     # Query & mutation logic
```

### Initialization

```bash
npm init -y
npm install apollo-server graphql
```

> [!note]  
> This installs `Apollo Server` (for creating the GraphQL server) and `graphql` (for defining types and executing operations).

---

## GraphQL Schema (`schema.js`)

The schema defines:

- **Types**: Objects like `Message`
    
- **Scalars**: Basic types (e.g., `ID`, `String`)
    
- **Non-nullable Fields**: Marked with `!`
    
- **Queries**: Read-only operations
    
- **Mutations**: Write operations
    

### Example Type Definition

```graphql
type Message {
  id: ID!
  content: String!
  author: String!
}
```

> [!info]  
> Fields with `!` are non-nullable—queries or mutations will fail if values are missing.

### Query Definition

```graphql
type Query {
  messages: [Message!]!
}
```

- Returns a list of `Message` objects.
    
- The list itself and each message are non-nullable.
    

### Mutation Definition

```graphql
type Mutation {
  sendMessage(content: String!, author: String!): Message!
}
```

- Accepts `content` and `author` as input.
    
- Returns a new `Message` object.
    

### Module Export

```js
module.exports = typeDefs;
```

> [!summary]  
> The schema defines how clients interact with the API. This contract is enforced at runtime via Apollo Server.

---

## Resolvers (`resolvers.js`)

Resolvers implement the behavior defined in the schema.

### Setup

```js
const { v4: uuidv4 } = require('uuid');
let messages = [];
```

> [!note]  
> Messages are stored in-memory (no database). `uuidv4` is used to generate unique message IDs.

### Query Resolver

```js
Query: {
  messages: () => messages
}
```

- Returns all messages from memory.
    

### Mutation Resolver

```js
Mutation: {
  sendMessage: (_, { content, author }) => {
    const newMessage = {
      id: uuidv4(),
      content,
      author
    };
    messages.push(newMessage);
    return newMessage;
  }
}
```

- Accepts mutation arguments
    
- Creates new message object
    
- Stores it in `messages` array
    
- Returns the new message
    

### Module Export

```js
module.exports = resolvers;
```

> [!summary]  
> Resolvers map schema operations to logic. They can fetch from memory, databases, or external APIs.

---

## Apollo Server (`app.js`)

### Imports and Initialization

```js
const { ApolloServer } = require('apollo-server');
const typeDefs = require('./graphql/schema');
const resolvers = require('./graphql/resolvers');
```

### Create Server and Start

```js
const server = new ApolloServer({ typeDefs, resolvers });

server.listen().then(({ url }) => {
  console.log(`Server ready at ${url}`);
});
```

> [!info]  
> The default port is `4000`. You can control-click on the terminal output to open the browser sandbox.

---

## Testing Queries and Mutations

Once the server is running at `http://localhost:4000`, test GraphQL operations using the Apollo sandbox.

### Example Mutation

```graphql
mutation {
  sendMessage(content: "Hello there", author: "Jeremy") {
    id
    content
    author
  }
}
```

Returns:

```json
{
  "data": {
    "sendMessage": {
      "id": "UUID",
      "content": "Hello there",
      "author": "Jeremy"
    }
  }
}
```

### Example Query

```graphql
query {
  messages {
    id
    content
    author
  }
}
```

Returns a list of all message objects.

> [!note]  
> You can query only the needed fields to demonstrate GraphQL's ability to **reduce over-fetching**.

```graphql
query {
  messages {
    author
  }
}
```

---

## Key GraphQL Concepts Demonstrated

|Concept|Description|
|---|---|
|**Schema**|Declares structure and available operations|
|**Resolvers**|Define how queries and mutations execute|
|**Scalars**|Basic types (`ID`, `String`)|
|**Non-nullable Fields**|Required inputs/outputs (`!`)|
|**In-memory Storage**|Used in place of a database for testing|
|**Apollo Server**|Manages GraphQL API lifecycle and validation|

> [!warning]  
> Without validation or authorization, in-memory storage and schema openness can be exploited. For real-world apps, enforce auth and input sanitization.

---

## Final Notes

- GraphQL server runs with **Apollo Server**, listening by default on `http://localhost:4000`.
    
- Operations are executed via the Apollo sandbox (no frontend required).
    
- This setup is useful for **observing query flexibility**, resolver function logic, and message flow.
    

> [!summary]  
> The lab covers the foundational structure of a GraphQL API: schema creation, resolver implementation, and server deployment. Knowing this structure helps during enumeration and testing of GraphQL-based web applications.
