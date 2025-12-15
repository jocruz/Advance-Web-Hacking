
# Understanding `__proto__` and Prototype Pollution in JavaScript

> [!info]  
> **Purpose**  
> This section explains the relationship between `__proto__`, `Object.prototype`, and prototype pollution vulnerabilities. It also walks through how pollution works, edge cases in modern JavaScript engines, and the real-world impact on client- and server-side applications.

---

## Key Concepts: Prototypes in JavaScript

- All JavaScript objects have an internal prototype.
    
- Access/modification happens via:
    
    - `__proto__` (deprecated but still widely supported)
        
    - `Object.setPrototypeOf()` and `Object.getPrototypeOf()`
        
- Most objects inherit from `Object.prototype`, the base prototype.
    

---

### Modifying Shared Prototypes

```javascript
let obj1 = {};
let obj2 = {};

obj1.sharedProp = "test";

console.log(obj2.sharedProp); // undefined
console.log(obj1.sharedProp); // "test"
```

Now modify the prototype:

```javascript
obj1.__proto__.sharedProp = "polluted";
console.log(obj2.sharedProp); // "polluted"
```

> [!tip]  
> Setting `obj1.__proto__` actually modifies `Object.prototype`, so **all objects** share the polluted property.

---

## Overwriting Built-in Methods via Prototype Pollution

```javascript
obj1.__proto__.toString = function() { alert("XSS"); };

console.log(obj2.toString()); // triggers alert
```

> [!danger]  
> Overwriting `Object.prototype.toString` impacts every object that inherits from it. This can lead to XSS if string conversions are used in the DOM.

---

## MDN Documentation & Legacy Behavior

- `__proto__` is deprecated but still widely supported.
    
- According to [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/proto):
    
    - "It exists for web compatibility."
        
    - "Its behavior is standardized as a legacy feature."
        
- Even if discouraged, it's retained to **avoid breaking old websites**.
    

> [!note]  
> As attackers, this "legacy" behavior is often an advantage.

---

## Example: When Pollution _Doesn’t_ Work

```javascript
let user = {};
let userInput = "__proto__";
user[userInput] = { isAdmin: true };

console.log(user.isAdmin); // true
console.log(Object.prototype.isAdmin); // undefined
```

- Why?
    
    - Modern engines like Chrome's V8 sometimes **insert an intermediate prototype** between user and `Object.prototype`.
        
    - This prevents accidental or malicious pollution.
        

Verify the prototype chain:

```javascript
console.log(user.__proto__); 
console.log(user.__proto__.__proto__);
```

---

## Working Example of Prototype Pollution

```javascript
let user2 = {};
let userInput = "__proto__";

user2[userInput].isAdmin = true;

console.log(user2.isAdmin); // true
console.log(Object.prototype.isAdmin); // true
```

> [!tip]  
> When `user2.__proto__` points directly to `Object.prototype`, pollution works. No intermediate prototype is inserted.

---

## Key Takeaways

> [!important]  
> Understand these concepts for identifying or exploiting prototype pollution:

- `__proto__` lets you modify the prototype chain.
    
- Modifying `Object.prototype` affects all objects.
    
- Behavior **varies across JavaScript engines** and **depends on the object structure**.
    
- Legacy compatibility means many web apps still support dangerous features.
    

---

## Impact of Prototype Pollution

- Prototype pollution lets attackers:
    
    - Inject properties into all objects.
        
    - Overwrite built-in methods (`toString`, `hasOwnProperty`, etc.).
        
    - Change application behavior globally.
        
    - Achieve **XSS on the client-side** or **RCE/DoS on the server-side**.
        

---

### Client-Side Impact

- Injected values into DOM-manipulating functions → **XSS**
    
- Changing defaults in frameworks (e.g., Vue, Angular) → **unexpected UI behavior**
    

### Server-Side Impact

- Modify config objects → **unauthorized access**
    
- Overwrite logic functions → **code execution**
    
- Denial of service if polluted objects crash logic
    

---

## Prototype Pollution Challenges

- **Engine-specific behavior** (e.g., V8 in Chrome) sometimes blocks pollution.
    
- Creating payloads requires understanding the prototype chain.
    
- Impact depends on how polluted objects are used.
    

> [!quote]  
> “Prototype pollution is application-specific. In some apps it leads to nothing, in others it’s game over.”

---

## Widespread Attack Surface via NPM

- NPM modules like `lodash`, `deep`, `merge`, `dot-prop` may be vulnerable.
    
- Developers often use object merging, which can be abused to pollute prototypes.
    
- Many vulnerable modules have **no CVE** but are still dangerous.
    

---

## Summary

|Concept|Explanation|
|---|---|
|`__proto__`|A legacy accessor for the prototype of objects|
|`Object.prototype`|The base object all others inherit from|
|Prototype Pollution|Injecting properties into object prototypes to affect multiple objects|
|Impact|Can lead to XSS, RCE, or DoS depending on how polluted data is used|
|Exploitation|Can occur on both **client-side** and **server-side**|
|Prevention|Avoid dynamic `merge()` or unchecked object assignments using user input|
|Real-World Risk|Many libraries/modules unknowingly expose pollution vectors|

---

## Further Reading

- [MDN: Object.prototype](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/prototype)
    
- [PayloadAllTheThings – Prototype Pollution](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Injections/Prototype%20Pollution)
    
- [HackerOne Report Examples](https://www.hackerone.com/blog/prototype-pollution)
    

---

