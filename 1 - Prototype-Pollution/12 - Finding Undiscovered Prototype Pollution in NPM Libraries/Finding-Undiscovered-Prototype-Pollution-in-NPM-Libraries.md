
# Automating Prototype Pollution Discovery in NPM Packages

> [!info]  
> **Overview**  
> This section explains how to take open-source research and tools like `findvolme.js` and a custom Bash script to detect potential prototype pollution vulnerabilities in NPM libraries. The process includes verifying findings, crafting proof of concepts, and practicing responsible disclosure.

---

## Initial Research and Setup

### Key Resource

- A research paper by **Oliver Ato** (linked in course material) outlines:
    
    - The nature of prototype pollution.
        
    - Common vulnerable functions like `merge()`, `clone()`, and `setByPath()`.
        
    - Affected libraries such as:
        
        - `lodash`, `merge`, `deep`, `defaults`, `hoax`, `dot-prop`, `deep-clone`, `pathval`.
            

### Associated Tool

- **findvolme.js** (JavaScript script to scan for potential pollution vulnerabilities).
    
- `findvolme.sh` is just a wrapper to run `findvolme.js`.
    

---

## Step-by-Step: Setting Up the Detection Tool

### 1. Save the Detection Script

```bash
nano findvolme.js
```

Paste the raw contents of the script and save (`Ctrl + X`, then `Y`).

### 2. Initialize the Node Project

```bash
npm init -y
```

### 3. Install a Known Vulnerable Package

```bash
npm install lodash@4.17.4
```

> [!tip]  
> This version of `lodash` is known to be affected by prototype pollution.

---

## Using the Detection Tool

### Run the Script

```bash
node findvolme.js lodash
```

Expected output:

```
Detected: Prototype pollution in lodash
```

To refine the output:

```bash
node findvolme.js lodash | grep detected
```

> [!success]  
> This confirms that the script has flagged lodash as potentially vulnerable.

---

## Building a Scanning Pipeline (Bash Script)

### Objective

Create a script that:

- Accepts keywords.
    
- Queries NPM for related libraries.
    
- Filters by popularity.
    
- Downloads, installs, and scans each package.
    
- Logs and deduplicates results.
    

---

### Structure of the Bash Script

1. **User Input**
    
    - Keywords (e.g., `clone`, `deep`, `merge`)
        
    - Popularity score range
        
    - Number of modules to scan
        
    - Timeout per module
        
2. **NPM Query**
    
    - Example API call:
        
        ```bash
        https://registry.npmjs.org/-/v1/search?text=deep&size=100
        ```
        
3. **Filter & Batch Install**
    
    - Packages filtered by popularity score.
        
    - Install in **batches of 10** to avoid partial failures.
        
4. **Scan Using `pollutionchecker.js`**
    
    - Script renamed from `findvolme.js`.
        
    - Renaming:
        
        ```bash
        mv findvolme.js pollutionchecker.js
        ```
        
5. **Log Results**
    
    - `log.txt`: List of scanned packages.
        
    - `found.txt`: Vulnerable libraries.
        
6. **Cleanup**
    
    - Uninstalls packages after each batch.
        

---

### Example Run

```bash
# Run the script (bash script is assumed to be already prepared)
bash scan-packages.sh
```

When prompted:

```
Keyword: deep
Popularity range: 0 - 0.2
Number of modules: 10
Timeout per module: 300
```

> [!tip]  
> Lower popularity ensures you're not scanning widely used production libraries unintentionally.

---

## Results and Manual Verification

### After the scan

View findings:

```bash
cat found.txt
```

If `deep` was found:

```bash
npm install deep
node pollutionchecker.js deep
```

Manual check:

```bash
npm audit
```

If no reported prototype pollution appears, build a minimal proof-of-concept to confirm.

---

## Confirming Prototype Pollution in `deep`

1. Install the library:
    

```bash
npm install deep
```

2. Write a basic test script:
    

```javascript
const deep = require('deep');
let obj = {};
deep(obj, '__proto__.polluted', true);
console.log({}.polluted); // true if vulnerable
```

3. Run it:
    

```bash
node test.js
```

If output shows `true`, the library is prototype pollution vulnerable.

> [!warning]  
> Even if `npm audit` doesn’t flag it, a manual PoC confirms the issue.

---

## Responsible Disclosure Process

- Email the maintainers (use email in `package.json` or GitHub repo).
    
- Wait 30 days.
    
- If no response, consider publishing a notice in a responsible way (e.g., GitHub issue).
    
- Avoid full public disclosure without remediation unless the risk is negligible (e.g., package is 11 years old with no maintenance).
    

---

## Key Takeaways and Exam Reminders

> [!important]  
> These are critical concepts and actions to recall during the exam.

- Use automation to scan hundreds of libraries quickly.
    
- `findvolme.js` flags potential prototype pollution but always confirm manually.
    
- Use the keywords:
    
    - `merge`, `clone`, `path`, `defaults`, `set`
        
- Common vulnerable functions:
    
    - `Object.assign`, custom merge implementations, path-based setters.
        
- Use `__proto__.polluted=value` to test vulnerability.
    
- Use `npm audit` for general CVE detection, but do not rely on it alone.
    
- Responsible disclosure is a required best practice.
    

---

## References

- Oliver Ato's paper on prototype pollution (linked in course materials)
    
- [PayloadAllTheThings: Prototype Pollution](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Injections/Prototype%20Pollution)
    
- [NPM Registry Search API](https://registry.npmjs.org/-/v1/search?text=clone)
    
- [Node.js `npm audit` documentation](https://docs.npmjs.com/cli/v9/commands/npm-audit)
    

---

Let me know if you'd like me to document the next lab: **server-side prototype pollution** or **automated CVE discovery through custom PoCs**.