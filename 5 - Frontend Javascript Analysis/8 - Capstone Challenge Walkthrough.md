# Capstone Challenge Walkthrough: Decrypting Frontend Logic

Let’s walk through the final capstone challenge. This exercise focuses on reverse engineering obfuscated frontend JavaScript code to gain unauthorized access to a protected article.

---

### Step 1: Open Developer Tools

1. Load the application and press `F12` to open Developer Tools.
    
2. Navigate to the **Sources** tab to begin inspecting the JavaScript code.
    

Inside the source code, you’ll find an obfuscated script block. To analyze it effectively:

- Collapse unnecessary elements in the DOM.
    
- Locate the obfuscated script—look for meaningful variable assignments or functions.
    

---

### Step 2: Identify the Secret Key

A promising line appears early:

```javascript
var secretKey = String.fromCharCode.apply(null, someArray);
```

To decode this:

1. Identify the array (e.g., `someArray`) that contains character codes.
    
2. Pass the values into a decoder that converts character codes to strings.
    

#### In the Console or an External Tool:

```javascript
String.fromCharCode(115, 117, 98, 115, 99, 114, 105, 98, 101, 114)
```

This decodes to:

```plaintext
subscriber
```

> [!Caution]
> The secret key used for article decryption is `"subscriber"`. This will be central to bypassing the access restriction.

---

### Step 3: Understand the Logic Check

The script checks local storage to verify if the user is a subscriber:

```javascript
if (localStorage.getItem("someKey") === secretKey) {
    // Decrypt and display the article
}
```

To test this in real time:

1. Place a breakpoint inside the conditional block.
    
2. Reload the page and monitor whether execution enters the decryption branch.
    

---

### Step 4: Bypass via Code Manipulation

To verify the logic:

- Manually force the condition to return `true` by altering the script temporarily.
    
- Observe that the article decrypts successfully.
    

But for a persistent solution, you'll want to simulate a subscriber without modifying the code.

---

### Step 5: Local Storage Injection

The goal is to populate local storage with a key-value pair that satisfies the conditional:

```javascript
localStorage.setItem("subscriber", "subscriber");
```

You can set this via the browser console or the Application tab in Developer Tools.

> [!Note]  
> You can fuzz or brute-force local storage keys to find what the code expects. In this case, both the key and value were `"subscriber"`.

Once this is stored, reload the page and the article will be automatically decrypted based on the frontend logic.

---

### Summary

This capstone challenge demonstrated how to:

- Use Developer Tools to deobfuscate and inspect JavaScript.
    
- Decode character codes with `String.fromCharCode`.
    
- Understand how localStorage conditions gate content access.
    
- Bypass content restrictions by mimicking expected storage states.
    

> ✅ **Final Takeaway**  
> Never rely on frontend-only checks for authentication or access control. Always validate on the server side, as client-side logic can be reverse-engineered and bypassed.
