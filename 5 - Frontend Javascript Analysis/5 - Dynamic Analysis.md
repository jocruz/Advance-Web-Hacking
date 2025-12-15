# Dynamic Analysis of Obfuscated JavaScript

## Summary

This session demonstrates how to **dynamically analyze obfuscated JavaScript** code by:

- Using browser dev tools and breakpoints
    
- Tracing variable values during execution
    
- Reconstructing the logic and flag checks step by step
    
- Leveraging tools like **CyberChef** to decode binary and Base64
    
- Comparing obfuscated and original versions of the same logic
    
- Understanding how to identify and work with **flag validation** mechanisms
    

It concludes with a walkthrough of a similar custom challenge (`challenge.js`) and encourages practicing the same dynamic methodology.

---

## Setup: Launching the Lab

1. Open `index.html` from **frontend-lab-2**
    
2. Press `F12` to open DevTools and dock it for easy inspection
    
3. Initially, the JavaScript may point to `obfuscated.js`.  
    Update it temporarily to `main.js` by editing **line 42 in `index.html`**:
    
    ```html
    <script src="main.js"></script>
    ```
    
4. Refresh the page and input a test flag:
    
    ```text
    test
    ```
    
    It will fail, prompting further inspection.
    

---

## Tracing the Flag Validation

### Step-by-Step Breakdown

1. **Open `main.js`** via the DevTools Sources panel.
    
2. Locate the following key variables:
    
    ```js
    const flagInput, checkButton, correctFlag, wrongFlag
    ```
    
3. Trigger the `checkFlag()` function:
    
    ```js
    const input = flagInput.value;
    ```
    

Use breakpoints and hover to inspect variable values during execution.

---

## Check 1: Flag Length Validation

```js
if (flag.length !== 12) {
  showWrongFlag();
  return false;
}
```

> [!note]  
> The required flag must be exactly **12 characters** long.

Input:

```text
1234567890ab  // 12 characters
```

---

## Check 2: Character Code at Index

```js
if (flag.charCodeAt(4).toString(2) !== "1100100") {
  showWrongFlag();
  return false;
}
```

Analysis:

```js
String.fromCharCode(parseInt("1100100", 2)); // Output: "d"
```

So the **5th character** (index 4) must be `'d'`.

> [!tip]  
> Use **CyberChef** with "From Binary" to quickly decode character checks.

---

## Check 3–n: More Binary Character Checks

Additional checks follow the same pattern:

```js
flag.charCodeAt(index).toString(2) === binary_value
```

Manually or via script/CyberChef, decode each binary string to ASCII and reconstruct the flag.

Example:

```js
"1111001" => "y"
"1100101" => "e"
"1101111" => "o"
```

---

## Check 4: Base64 Comparison

```js
const base64Segment = btoa(flag.substring(0, 3));
if (base64Segment !== "YXdo") {
  showWrongFlag();
  return false;
}
```

Decoded:

```bash
echo "YXdo" | base64 --decode
# Output: "awh"
```

Or via CyberChef: use **"From Base64"**  
→ The first three characters must be `"awh"`.

---

## Check 5: Character Position Validation

```js
if (flag[3] !== "{") {
  showWrongFlag();
  return false;
}
```

Character at **index 3** must be `{`

Final flag:

```text
awh{dynamic}
```

Add remaining characters:

```text
awh{dynamic}
```

When entered and `Check Flag` is pressed, the flag is accepted.

---

## Switching to Obfuscated Version

Now update `index.html` line 42 to:

```html
<script src="obfuscated.js"></script>
```

Refresh the app. The logic is the same as the non-obfuscated version, but:

- Variable names are changed
    
- Expressions are less readable
    
- Some values appear in **hex** or **binary**
    

---

## Dynamic Analysis of `obfuscated.js`

### Finding a Good Breakpoint

Use **Sources** → `obfuscated.js`, and set a breakpoint near:

```js
if (flag.length !== 12)
```

Once passed, step through the code:

- Look for binary/hex checks
    
- Use hover or console to inspect runtime values
    
- Look for Base64 string comparisons
    
- Break down comparisons using CyberChef or inline JS
    

> [!note]  
> Even if the code is obfuscated, the **runtime behavior reveals values** when hovered or executed.

---

### Pattern Recognition

Look for patterns such as:

```js
flag.charCodeAt(x).toString(2) === binary_string
```

You can reconstruct these checks just as in the readable version.

---

## Packed Value Discovery

If you encounter this:

```js
btoa(flag.substring(0, 3)) === "YXdo"
```

It still maps to `awh`, and the logic holds.

---

## Flag Construction

Flag is the same:

```text
awh{dynamic}
```

And validated using a chain of:

- Length check
    
- Character codes
    
- Base64 conversion
    
- Bracket character
    
- Final length enforcement
    

---

## Bonus Challenge: `challenge.js`

- Another obfuscated challenge was added as `challenge.js`
    
- Contains slightly modified logic
    
- Use same dynamic analysis workflow:
    
    - Set breakpoints
        
    - Step through code
        
    - Hover to inspect variables
        
    - Decode values using CyberChef or console
        

To use:

1. Edit `index.html`:
    
    ```html
    <script src="challenge.js"></script>
    ```
    
2. Refresh and inspect using DevTools
    

> [!tip]  
> Work line-by-line and **comment each operation** in your text editor to build understanding.

---

## Key Tools & Resources

|Tool / Resource|Purpose|
|---|---|
|**DevTools (F12)**|Breakpoints, inspection, stepping through code|
|**CyberChef**|Decode binary/Base64, convert formats|
|**console.log / watch panel**|Test values at runtime|
|**Visual Studio Code**|View and annotate JS files|
|**challenge.js**|Practice obfuscated code walkthrough|

---

## Key Takeaways

> [!tip]  
> **Dynamic analysis** allows you to inspect obfuscated code efficiently by watching runtime behavior instead of reverse engineering statically.

> [!note]  
> Even with obfuscation, breakpoints and variable hovering can reveal decoded flag values step by step.

> [!warning]  
> Always analyze unknown JS in a **controlled browser environment** (never execute in prod or sensitive systems).



http://10.10.10.10:3001/api/verify?token=4e969030e67f1151009e3204bd342ad20bc8a31f7ffc274f9f72291a69915566