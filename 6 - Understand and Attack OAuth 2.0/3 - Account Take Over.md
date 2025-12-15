# Attacking OAuth 2.0: Code Theft, Account Hijacking & Scope Abuse

### High-Level Summary

This walkthrough demonstrates **how to exploit weaknesses in OAuth 2.0 implementations**, focusing on the authorization code flow. Key attacks include stealing authorization codes to hijack accounts, analyzing and manipulating scopes for privilege escalation, and identifying missing protections like **PKCE**. The attacker leverages Burp Suite’s Collaborator to intercept and reuse codes meant for another user. This section reinforces how user interaction, redirect URI control, and weak configurations can lead to serious OAuth flaws.

---

## Attack Objective

The goal is to:

1. **Hijack a victim’s account** by stealing their OAuth authorization code
    
2. **Reuse the stolen code** to log in as the victim
    
3. **Manipulate the `scope`** parameter to escalate privileges and retrieve sensitive data
    

---

## Step 1: Set Up Users

Two accounts are registered:

- **jesemme** (attacker): `jesemme@jesemme.com / password`
    
- **jeremy** (victim): `jeremy@jeremy.com / password`
    

The attacker will log in as **Jeremy** by stealing his authorization code.

---

## Step 2: Intercept the Authorization Code

1. Go to the **client app** and click **Log in with OAuth Provider** as Jeremy.
    
2. In Burp Suite, enable **Intercept**.
    
3. Click **Allow** on the consent screen.
    

At this point, the intercepted request contains:

- `client_id`
    
- `redirect_uri`
    
- `code`
    
- `state`
    

The attacker **manually forwards** the request, simulating that the victim has clicked a malicious link or form.

> [!note]  
> In the real world, attackers typically trick users via malicious websites, phishing emails, or JavaScript injection (e.g., via XSS) to capture authorization codes. These attacks require **user interaction**.

---

## Step 3: Capture the Code Using Burp Collaborator

In **Burp Collaborator**, the attacker observes an incoming request containing:

```
code=abcd1234
```

This is the victim's **authorization code**, intercepted through the poisoned redirect URI.

---

## Step 4: Use the Stolen Code

Now acting as the attacker:

1. Log into the OAuth provider as **Jeremy** (simulating attacker possession of Jeremy's code).
    
2. Intercept the request during login.
    
3. Replace the attacker’s own code with **Jeremy’s stolen code**.
    
4. Forward the request and allow traffic to continue.
    

Result:

- The attacker is now **authenticated as Jeremy** on the client app.
    

> [!warning]  
> This demonstrates **account takeover via stolen authorization codes**. If PKCE (Proof Key for Code Exchange) is not used, code interception becomes a critical vulnerability.

---

## Step 5: Scope Manipulation

Beyond basic account access, an attacker may attempt to:

- Expand the `scope` parameter to gain unauthorized access to sensitive data
    
- Fuzz undocumented or poorly enforced scopes
    

Common attack strategies:

- Check the provider’s documentation (e.g., Google, Facebook) for supported scopes
    
- Try known or custom scopes such as:
    
    ```http
    scope=profile email address card_details ssn
    ```
    
- Fuzz for hidden scopes when dealing with:
    
    - Smaller OAuth providers
        
    - Poorly documented or custom APIs
        

> [!tip]  
> When `scope` is improperly validated or enforced, attackers can escalate privileges and access additional sensitive information like addresses, credit card numbers, or social security numbers.

---

## Key Security Observation

Navigate to **All Issues** in Burp Suite to see a flagged vulnerability:

```
OAuth 2.0 Authorization Code Flow Without PKCE

> [!warning]  
> **PKCE (Proof Key for Code Exchange)** is essential to protect public clients (e.g., mobile or JavaScript apps) from code interception attacks. Its absence significantly increases the risk of authorization code abuse.

---

## Attack Flow Summary

```plaintext
Attacker registers account
⬇
Victim logs in via OAuth
⬇
Attacker intercepts authorization code using poisoned redirect URI (via Collaborator)
⬇
Attacker logs in as victim by replacing their own code with the stolen one
⬇
Attacker explores scope manipulation to escalate access
```

---

## Tools Mentioned

|Tool / Concept|Description|
|---|---|
|**Burp Suite**|Used for request interception and analysis|
|**Burp Collaborator**|Captures outbound HTTP requests and payloads for code stealing|
|**OAuth 2.0 Framework**|The protocol being attacked|
|**PKCE (Proof Key for Code Exchange)**|Missing in this lab — meant to prevent code interception|
|**Browser Interaction** (implied)|Used to simulate real-world user logins|
|**Scope Enumeration & Fuzzing**|Used to find undocumented or weakly enforced scopes|
|**Custom Web Page or Form** (implied)|Required in real-world attacks to capture victim interaction|
