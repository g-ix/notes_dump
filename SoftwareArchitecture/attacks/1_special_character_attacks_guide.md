# Special-character attacks — from basic to advanced (and how to stop them)

Special characters are one of the most common yet underestimated attack surfaces in modern applications.  
This guide explains—from **basic to advanced**—how attackers exploit them and how to secure your codebase.

---

## 1. Basic / Common Attacks

### 1.1 Path Traversal
**What:** Using `../` or encoded equivalents to escape directories and access restricted files.  
**Example:** `../../../etc/passwd` or `%2e%2e%2fetc%2fpasswd`  
**Impact:** Data theft, file overwrite, privilege escalation.  
**Mitigation:** Canonicalize paths, reject inputs containing `..` or slashes, store files by UUID instead of user-supplied names.

### 1.2 Null Byte Injection
**What:** `%00` or `\0` to terminate strings early.  
**Example:** `file.png%00.jpg` → treated as `file.png`.  
**Impact:** Bypass validation, execute arbitrary content.  
**Mitigation:** Strip control chars, reject `\u0000`, validate decoded strings.

### 1.3 Directory Separator Injection
**What:** Inserting `/` or `\` to alter file paths.  
**Impact:** File overwrite, unintended directory creation.  
**Mitigation:** Disallow `/`, `\`, and `:` in user-supplied path parts.

### 1.4 Command Injection
**What:** Using `;`, `&&`, `|`, backticks, `$()` to inject shell commands.  
**Example:** `rm -rf /; echo`  
**Impact:** Remote Code Execution (RCE).  
**Mitigation:** Never call shell with user input. Use APIs like `spawn()` or prepared argument arrays.

---

## 2. Web and Data-Layer Attacks

### 2.1 XSS (Cross-Site Scripting)
**What:** Injecting `<`, `>`, `'`, `"`, `(`, `)` to run scripts.  
**Example:** `"><script>fetch('/exfil?cookie='+document.cookie)</script>`  
**Impact:** Cookie theft, session hijacking.  
**Mitigation:** Contextual output encoding (HTML, JS, URL), CSP headers.

### 2.2 SQL / NoSQL Injection
**What:** Abuse of `'`, `"`, and operators (`$ne`, `$where`).  
**Example:** `'; DROP TABLE users; --`  
**Impact:** Data corruption, leakage, admin compromise.  
**Mitigation:** Always use parameterized queries or ORM bindings.

### 2.3 CSV / Formula Injection
**What:** Injecting `=`, `+`, `-`, `@` into CSV cells that Excel executes as formulas.  
**Example:** `=cmd|' /C calc'!A0`  
**Impact:** RCE or data exfiltration when CSV opened.  
**Mitigation:** Prefix with `'` before export, sanitize formulas.

### 2.4 LDAP / XPath / JSON Injection
**What:** Manipulate queries with special chars in filters.  
**Mitigation:** Escape values properly or use parameterized queries.

---

## 3. Input Processing / Parser-Level Attacks

### 3.1 CRLF Injection
**What:** Injecting `\r\n` to create fake HTTP headers.  
**Example:** `Set-Cookie: session=abc\r\nLocation: http://evil.com`  
**Impact:** Cache poisoning, header injection, redirect exploits.  
**Mitigation:** Strip control characters and validate headers.

### 3.2 Email Header Injection
**What:** Inject `\r\n` to add custom headers in emails.  
**Mitigation:** Validate email fields and remove CR/LF sequences.

### 3.3 Format String Attack
**What:** Supplying `%s`, `%x`, `%n` patterns to gain memory access.  
**Impact:** Memory leak or code execution (C/C++).  
**Mitigation:** Never use untrusted input as format strings.

### 3.4 Regex Denial of Service (ReDoS)
**What:** Crafted regex inputs causing exponential backtracking.  
**Example:** `aaaaaaaaaaaaaaaa!` with `(.+)+`  
**Mitigation:** Avoid catastrophic regexes, use safe libraries or timeouts.

---

## 4. Advanced Unicode / Encoding Attacks

### 4.1 Homoglyph (Confusable) Characters
**What:** Using look-alike characters (`а` Cyrillic vs `a` Latin).  
**Impact:** Phishing, credential spoofing, UI deception.  
**Mitigation:** Normalize to NFKC, detect cross-script mixing, warn users.

> NFKC: Normalization Form KD (Compatibility Decomposition - exactly! Why there's 'K' in NFKD?) - NFKD performs a compatibility decomposition - replacing characters that have similar but not identical forms with their basic equivalents. 
> NFKD first applies a "compatibility decomposition," which breaks down characters into their simplest forms. This includes replacing compatibility characters with their basic equivalents. 
> For NFKD, visual equivalence is more important than exact character representation. 


### 4.2 BiDi / Right-to-Left Override (U+202E)
**What:** Reverses filename display order to hide true extensions.  
**Example:** `evilgpj‮exe` → shows as `evil.jpg`.  
**Mitigation:** Strip format controls (`\p{Cf}`), disallow U+202E, log anomalies.

### 4.3 Unicode Normalization & Mojibake
**What:** Incorrect UTF decoding leading to weird characters like `â¯`.  
**Impact:** Filter bypass, data corruption.  
**Mitigation:** Normalize (NFKC), ensure UTF-8 consistency, reject invalid sequences.

### 4.4 Zero-Width / Invisible Characters
**What:** Hidden chars (U+200B) create visually identical strings.  
**Impact:** Filter evasion, duplicate account names.  
**Mitigation:** Remove `\p{Cf}`, normalize input.

### 4.5 Double Encoding / Percent Encoding
**What:** Use `%252e%252e%252f` to double-encode traversal sequences.  
**Mitigation:** Canonicalize safely once and revalidate.

---

## 5. Advanced Protocol / System-Level Attacks

### 5.1 Surrogate Pair Corruption
**What:** Broken UTF-16 pairs cause memory corruption in native parsers.  
**Mitigation:** Validate UTF input, use safe decoding libraries.

### 5.2 SQL Collation & Case-Insensitive Tricks
**What:** Exploit database collation differences for duplicates.  
**Mitigation:** Store canonical keys in normalized collation.

### 5.3 OS / Filesystem Quirks
**What:** Windows ignores dots/spaces; mixed path normalization.  
**Mitigation:** Normalize per platform, validate at every layer.

---

## 6. Defensive Checklist

1. **Normalize** all strings (`NFKC`) and reject `\p{Cc}` / `\p{Cf}`.  
2. **Use allowlists** over blocklists.  
   ```ts
   const ALLOWED = /^[\p{L}\p{N}\p{M}\p{Sk} '"@#$%*&.,\-|~()]*$/u;
   ```
3. **Never interpolate input into shell or queries.**  
4. **Store files by UUID,** not original filename.  
5. **Contextually encode** on output (HTML, URL, CSV, JS).  
6. **Strip invisible / control characters.**  
7. **Log disallowed chars** and monitor for abuse.  
8. **Limit lengths** to mitigate ReDoS/fuzzing.  
9. **Sandbox file paths** and validate MIME types.  
10. **Warn admins** if inputs contain homoglyphs or RTL overrides.

---

## 7. Hardened Filename Sanitizer (Express Example)

```ts
export function sanitizeFilename(input: string, maxLen = 150) {
  // Normalize
  let s = input.normalize('NFKC');
  // Strip control and format characters
  s = s.replace(/[\p{Cc}\p{Cf}]+/gu, '');
  // Remove dangerous path chars
  s = s.replace(/[\/\\:]+/g, '');
  // Limit and trim
  s = s.trim().slice(0, maxLen);
  // Whitelist validation
  const ALLOWED = /^[\p{L}\p{N}\p{M}\p{Sk} '"@#$%*&.,\-|~()]*$/u;
  const ok = ALLOWED.test(s);
  return { ok, value: s };
}
```

---

## 8. TL;DR

1. Normalize inputs (NFKC).  
2. Disallow `/`, `\`, `:` and null bytes.  
3. Strip invisible/control characters.  
4. Allow Unicode letters safely.  
5. Always store & serve sanitized versions.  
6. Log anomalies.

---

Author: **Oracle Security Notes — 2025 Edition**
