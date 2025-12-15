# OAuth 2.0: Missing PKCE Protection in Authorization Code Flow

### High-Level Summary

This section explains the **PKCE (Proof Key for Code Exchange)** extension to OAuth 2.0 and how its absence exposes public clients (like SPAs and mobile apps) to authorization code interception attacks. The Burp Suite scanner flagged a missing `code_challenge` parameter, confirming that the implementation lacks PKCE. This module breaks down how PKCE works, when it's needed, and why it's a required security control for public clients that cannot securely store secrets.


>[!summary]
> **OAuth 2.0 Authorization Code Flow (with PKCE)**  
> The client app redirects the user to the OAuth provider to log in and approve access. After consent, the provider returns an authorization code. The client app then exchanges this code for tokens in a secure backend request.
>
> **PKCE (Proof Key for Code Exchange)** is an added security layer for public clients. The app:
> 1. Generates a `code_verifier` (a random secret)
> 2. Hashes it to create a `code_challenge`
> 3. Sends the `code_challenge` to the provider during the login request
> 4. Later sends the original `code_verifier` to exchange the code for tokens
> 	1. Server to Server, it will never go through the browser, it goes from the backend to the token server or whatever Oauth Server
> 
>
> The server checks if `SHA256(code_verifier)` matches the earlier `code_challenge`. If yes, tokens are returned.
>
> **Why it's secure:** Even if an attacker steals the authorization code, they can't use it without the original `code_verifier`, which was never exposed. Only the hash (`code_challenge`) was public.

>[!note]
> **Server-to-Server Requests in OAuth**  
> The **authorization code exchange** (Step 5 of the flow) happens **server-to-server**, meaning it does **not go through the browser**.  
> The client’s **backend** sends the code and the `code_verifier` directly to the **token endpoint** of the OAuth provider.  
> This keeps the `code_verifier` (the secret) **hidden from attackers**, even if the browser traffic is intercepted.

---

## What Is PKCE?

**PKCE** stands for **Proof Key for Code Exchange**. It is an extension to the OAuth 2.0 Authorization Code Flow designed to:

- Protect **public clients** (e.g., mobile apps, single-page applications, desktop apps)
    
- Prevent **authorization code interception attacks**
    

> [!note]  
> PKCE is not optional in modern secure OAuth flows — it is **required** for public clients that cannot securely store a `client_secret`.

---

## Burp Suite Finding

The Burp Suite scanner reported the following issue:

> **OAuth 2.0 Authorization Code Flow Without PKCE**

Details:

- The login request does **not** include the `code_challenge` parameter.
    
- This indicates that **PKCE is not implemented**.
    
- As a result, the flow is vulnerable to **authorization code interception attacks**.
    

This vulnerability was demonstrated in the **previous lab**, where a stolen code allowed the attacker to impersonate the victim.

---

## How PKCE Works

PKCE adds an **extra verification step** to the standard OAuth flow:

### Step 1: Client Generates a Code Verifier

Before sending the user to the authorization server, the client generates a **random code verifier**:

```plaintext
code_verifier = random(43–128 characters)
```

### Step 2: Client Derives a Code Challenge

The client creates a **code challenge** from the verifier. Usually:

```plaintext
code_challenge = base64url(SHA256(code_verifier))
```

> [!tip]  
> The client can also use the `plain` method, where `code_challenge = code_verifier`, though this is less secure than SHA256 (called the `S256` method).

### Step 3: Client Sends Authorization Request

```http
GET /authorize?
  response_type=code
  client_id=public-client
  redirect_uri=https://client.example.com/callback
  code_challenge=xyz123
  code_challenge_method=S256
```

### Step 4: Server Returns Authorization Code

The server issues the authorization code and records the `code_challenge`.

### Step 5: Token Request with Code Verifier

The client exchanges the code for a token, sending the original `code_verifier`:

```http
POST /token
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code
code=abc123
code_verifier=random_original_value
client_id=public-client
redirect_uri=https://client.example.com/callback
```

### Step 6: Server Verifies Code Verifier

The server:

- Hashes the `code_verifier`
    
- Compares it to the original `code_challenge`
    
- If it matches, returns an access token
    

> [!warning]  
> Without PKCE, **anyone with the authorization code** can exchange it for a token. PKCE prevents this by requiring the original verifier.

---

## Why PKCE Matters

PKCE is **critical for security** when the client:

- Cannot safely store a `client_secret`
    
- Is a **mobile app**, **desktop app**, or **SPA**
    
- Operates in an **environment where interception is possible**
    

### Without PKCE:

- Authorization codes can be **intercepted** (e.g., via MitM, XSS, proxy manipulation)
    
- Attackers can exchange them for access tokens
    
- Users can be impersonated
    

### With PKCE:

- Only the original client with the correct `code_verifier` can successfully complete the token exchange
    

> [!note]  
> PKCE **does not require any server-side secrets**, making it ideal for public clients. Its adoption is now standard in modern secure OAuth implementations.

---

## Conclusion

Burp Suite identified a missing `code_challenge`, confirming the **absence of PKCE**. This makes the application vulnerable to **authorization code reuse** and **account takeover**. Implementing PKCE is essential for any public client using the OAuth 2.0 Authorization Code Flow.

---

## Tools Mentioned

|Tool / Concept|Description|
|---|---|
|**Burp Suite**|Flags missing PKCE in OAuth flows|
|**PKCE (Proof Key for Code Exchange)**|OAuth extension for public client protection|
|**code_verifier**|Random string generated by client|
|**code_challenge**|Hash or plain version of `code_verifier`, sent during auth request|
|**code_challenge_method**|Typically `S256`, indicating SHA256 hash is used|
|**SPA / Mobile / Desktop Clients**|Public clients that cannot securely store secrets|
