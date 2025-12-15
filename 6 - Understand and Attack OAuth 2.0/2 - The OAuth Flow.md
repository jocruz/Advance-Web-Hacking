# OAuth 2.0: Hands-On Flow Walkthrough

### High-Level Summary

This walkthrough provides a detailed, practical demonstration of the OAuth 2.0 **Authorization Code Flow**, highlighting the interaction between the client and provider applications, HTTP requests, and key security elements like the `state` parameter. The flow is traced using Burp Suite, and attention is given to identifying CSRF protections, token exchanges, redirect URIs, and consent prompts. This is essential knowledge for understanding OAuth internals, debugging issues, and preparing for security assessments.

---

## Lab Setup

Two applications are used in this walkthrough:

- **Client App** (running on `localhost:3001`)
    
- **OAuth Provider** (running on `localhost:3000`)
    

Start each with:

```bash
cd oauth-client
node app.js

cd oauth-provider
node app.js
```

Both apps should now be accessible in the browser.

---

## Observing the OAuth Flow with Burp Suite

Start **Burp Suite** and open the embedded browser. Navigate to:

- `http://localhost:3000` (Provider)
    
- `http://localhost:3001` (Client)
    

### Initial Registration

1. Create a new user (e.g., **username: Jeremy**, **password: password**) on the provider.
    
2. Register a new OAuth client (e.g., name it `J1`).
    
3. Proceed to login via **"Log in with OAuth Provider"** from the client application.
    

---

## Step-by-Step Flow Analysis

### Step 1: Initiating the OAuth Login

When you click “Log in with OAuth Provider,” the client initiates the login request by redirecting the user to the provider’s `/auth/authorize` endpoint. This includes a **randomly generated `state` parameter**, used to prevent **Cross-Site Request Forgery (CSRF)**.

> [!tip]  
> Always verify that the `state` parameter is present and validated on both ends of the flow to mitigate CSRF risks.

Sample request:

```
GET /auth/authorize?state=ZNWW6...
```

### Step 2: Consent and Authorization

Since the user is already authenticated in the provider, they are not prompted for credentials. Instead, they are taken directly to a **consent screen** showing the scopes requested:

- Read profile info
    
- Username
    
- Email
    
- Profile picture
    

Clicking **Allow** sends a POST request to `/authorize`.

Sample POST body:

```http
POST /authorize
Content-Type: application/x-www-form-urlencoded

client_id=test-client
redirect_uri=http://localhost:3001/oauth/callback
scope=profile email
state=ZNWW6
consent=allow
```

### Step 3: Authorization Code Issuance

If the request is valid, the provider issues a **short-lived authorization code** and performs a redirect back to the client’s redirect URI:

```
302 Found
Location: http://localhost:3001/oauth/callback?code=xyz123&state=ZNWW6
```

> [!note]  
> The `code` is the short-lived authorization code that will be exchanged for tokens. The `state` value must match the original to ensure integrity.

---

## Step 4: Token Exchange (Backchannel)

This next step occurs **server-to-server** (not visible in the browser or Burp Suite):

The client exchanges the authorization code for tokens by sending a POST request to the provider:

```http
POST /oauth/token
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code
client_id=test-client
client_secret=supersecret
code=xyz123
redirect_uri=http://localhost:3001/oauth/callback
```

In response, the provider returns:

```json
{
  "access_token": "abc123",
  "id_token": "jwt...",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

These tokens are now used by the client to authenticate and authorize API calls.

---

## Final Step: Authenticated Access

After successful token acquisition, the client logs the user into the application. In this case, the user **Jeremy** is now authenticated and can use the to-do list app.

---

## Flow Summary

```plaintext
Client App ➝ Authorization Request ➝ OAuth Provider
           ⬑ Consent Prompt + State Validation
           ➝ Authorization Code Issued ➝ Redirect to Client
Client App ➝ Token Exchange (Server-to-Server)
           ⬑ Access + ID Tokens Received
Client App ➝ Authenticated User Session Established
```

> [!note]  
> Throughout this flow, proper handling of parameters like `state`, `client_id`, `redirect_uri`, and `scope` is critical to ensure OAuth is secure.

---

## Key Parameters to Observe

|Parameter|Description|
|---|---|
|`client_id`|Identifies the client application making the request|
|`redirect_uri`|Where the authorization server will redirect after the user consents|
|`response_type`|Usually set to `code`, requesting an authorization code|
|`state`|A CSRF protection value that must be validated throughout the flow|
|`scope`|Defines the level of access requested by the client|

---

## Tools Mentioned

|Tool / Concept|Description|
|---|---|
|**Burp Suite**|Used to intercept and analyze the OAuth flow|
|**Node.js**|Runtime environment used to launch the OAuth apps|
|**Browser Developer Tools** (implied)|Used for interacting with apps and consent screens|
|**OAuth Client App**|Runs on `localhost:3001`, initiates the flow|
|**OAuth Provider App**|Runs on `localhost:3000`, handles authentication and token issuance|
|**Authorization Server** (conceptual)|Manages consent and issues tokens|
|**Resource Server** (implied)|Would normally handle API requests authenticated via tokens|
