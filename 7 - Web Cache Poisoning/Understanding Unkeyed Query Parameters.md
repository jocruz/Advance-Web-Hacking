
## Understanding Unkeyed Query Parameters

An unkeyed query parameter is a parameter that is used by the backend to generate the response but is ignored by the caching layer when deciding whether to store or serve a cached version of the page. This creates a mismatch: different inputs (like `ref=abc` vs `ref=<script>`) result in different responses, but the cache treats them as the same. As a result, a malicious response can be stored in the cache and later served to users who make harmless requests. This is the basis of cache poisoning using unkeyed parameters.

### Example of Unkeyed Behavior:

1. Attacker sends:  
    `GET /?ref=<script>alert(1)</script>`  
    The backend reflects this, and the cache stores the response.
    
2. A normal user visits:  
    `GET /?ref=hello`  
    But the cache serves the attacker’s version, since it didn’t consider `ref` when caching.
    

This means `ref` is unkeyed — the backend uses it, but the cache doesn’t.

---

## Cache Busters

A cache buster (commonly named `cb`) is a dummy query parameter used to force the cache to treat each request as unique. The backend usually ignores this parameter, but the caching system includes it in the cache key.

This makes it useful for testing, because it creates isolated cache entries and avoids polluting shared responses.

### Example of Cache Buster Behavior:

1. Tester sends:  
    `GET /?ref=abc&cb=1`  
    The cache sees this as a unique key and stores the response.
    
2. Tester sends:  
    `GET /?ref=abc&cb=2`  
    The cache stores this separately.
    
3. Each time `cb` changes, the cache treats it as a new version — even if the response is the same.
    

Cache busters work only if the caching system is configured to include query parameters in the cache key. If the cache drops query parameters entirely, then both `cb=1` and `cb=2` are treated the same (as `/`), and the cache buster has no effect.

Understanding which parameters are unkeyed, keyed, or dropped is essential when testing for cache poisoning vulnerabilities.