# Understanding and Attacking OAuth 2.0

### High-Level Summary

OAuth 2.0 is a secure authorization framework that allows third-party applications to access user data without exposing user credentials. It operates by issuing tokens with defined scopes and roles to ensure safe, delegated access. This note covers the theory behind OAuth, its components, token lifecycle, scopes, and common misconfigurations that can introduce vulnerabilities. Understanding this framework is essential for both building secure applications and identifying flaws during security assessments.

---

## What Is OAuth 2.0?

OAuth 2.0, short for **Open Authorization**, is an open standard designed to provide **secure, delegated access** to user resources. Instead of sharing a username and password, users grant limited access to applications through **tokens**.

This allows:

- Secure interaction between applications and services
    
- Fine-grained access control
    
- Passwords to remain confidential
    

It’s widely implemented by platforms such as **Google**, **Facebook**, and other major providers.

---

## The Four Roles in OAuth

OAuth's model revolves around four distinct roles:

### 1. Resource Owner

The individual or entity that owns the protected resources — typically, **the user**.

### 2. Client

The application that requests access to the user's data. This is the **third-party app**.

### 3. Resource Server

The server that holds the protected data. It responds to requests that include a **valid access token**.

### 4. Authorization Server

Responsible for **authenticating the user** and issuing **access and refresh tokens** to the client.

> [!note]  
> A solid grasp of these roles is critical for identifying where OAuth vulnerabilities can arise, especially in real-world attacks like token leakage or improper validation.

---

## OAuth Authorization Code Flow

The most common flow is the **Authorization Code Flow**, which securely exchanges credentials and tokens. Here's a step-by-step breakdown:

### 1. Authorization Request

The client redirects the user to the authorization server to begin the consent process.

### 2. User Consent

The user is presented with a consent screen, detailing requested **scopes** such as email access or personal profile info.

### 3. Authorization Code Issuance

If consent is granted, the authorization server returns a short-lived **authorization code** by redirecting the user back to the client.

### 4. Token Exchange

The client sends the authorization code to the authorization server in exchange for an **access token** (and possibly a refresh token).

### 5. Access Resource

The client uses the access token to retrieve protected data from the resource server.

> [!tip]  
> This flow ensures that user credentials are never exposed to the client directly. Instead, the system relies on short-lived tokens to maintain security.

---

## Tokens in OAuth

Tokens are central to OAuth’s security design.

### Access Token

- Grants the client permission to access specific resources
    
- Typically short-lived
    
- Helps limit the damage if a token is leaked
    

### Refresh Token

- Used to obtain new access tokens without re-authenticating the user
    
- Long-lived and should be protected securely
    

---

## Understanding Scopes

**Scopes** define what the client can do with the access token. They enforce **least privilege** by limiting token capabilities.

Examples:

- `read:user` — read-only access
    
- `write:data` — permission to modify user data
    

> [!warning]  
> Misconfigured or overly broad scopes can grant unnecessary access, potentially leading to privilege escalation or data leaks.

---

## Why OAuth Is Secure (In Theory)

OAuth’s design introduces several **security advantages**:

- **Role separation** helps isolate responsibilities
    
- **Token-based access** avoids password sharing
    
- **Scopes** limit what can be done with a token
    
- **Expiration and revocation** reduce token misuse risk
    

If an access token is compromised:

- It can be revoked
    
- The user’s credentials remain safe
    

---

## Common Implementation Pitfalls

While OAuth is secure by design, poor implementation introduces vulnerabilities:

### 1. Misconfigured Redirect URIs

Attackers may intercept tokens if the redirect URI is not strictly validated.

### 2. Insecure Token Storage

Tokens stored insecurely (e.g., local storage in the browser) may be accessed by malicious scripts.

### 3. Overly Broad Scopes

Excessive permissions allow more access than needed — violating the principle of least privilege.

### 4. Long Token Lifetimes

Tokens valid for too long increase the window of opportunity for misuse if stolen.

### 5. Lack of HTTPS

Unencrypted traffic can lead to **man-in-the-middle (MitM) attacks** where tokens or credentials are intercepted.

> [!warning]  
> Even a theoretically secure protocol like OAuth becomes dangerous if redirect URIs, scopes, or token handling are misconfigured.

---

## Conclusion

OAuth 2.0 is a widely used authorization framework that, when properly implemented, allows secure delegated access to user data. It eliminates the need for password sharing and ensures access is limited through tokens and scopes. However, real-world security depends heavily on correct implementation and configuration.

---

## Tools Mentioned

|Tool / Concept|Description|
|---|---|
|**Burp Suite**|Used to inspect and test each step of the OAuth flow|
|**OAuth 2.0 Framework**|Core protocol being explained and tested|
|**HTTPS** (implied)|Necessary for secure communication and protection against MitM attacks|
|**Authorization Server** (conceptual)|Issues tokens and handles user authentication|
|**Resource Server** (conceptual)|Stores and serves protected data based on token access|
|**Browser/Client Application** (implied)|Where redirect URIs and token storage issues often occur|
