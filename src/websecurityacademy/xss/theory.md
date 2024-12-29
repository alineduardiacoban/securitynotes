# Cross-Site Scripting (XSS)

## Table Of Contents
- [What Are XSS Vulnerabilities](#what-are-xss-vulnerabilities)
- [How to Find And Exploit XSS Vulnerabilities](#how-to-find-and-exploit-xss-vulnerabilities)
- [How to Prevent XSS Vulnerabilities](#how-to-prevent-xss-vulnerabilities)

---

## What Are XSS Vulnerabilities?

Cross-Site Scripting (XSS) vulnerabilities allow an attacker to inject malicious client-side scripts into the application, which are then executed in the victim user's web browser.

---

### Types of XSS

1. **Reflected XSS**:
   - Arises when the application directly reflects client-supplied input in the response of the request in an unsafe way.

2. **Stored XSS**:
   - Arises when the application stores client-supplied input in an unsafe way and includes it in later responses.

3. **DOM-Based XSS**:
   - Arises when JavaScript takes data from client-supplied input and passes it to a sink that supports dynamic code execution in the DOM.

---

### What Can XSS Be Used For?

- **Impersonate** or masquerade as the victim user.
- Perform any action the user can perform.
- **Read data** that the user can access.
- **Inject trojan functionality** into the website to capture login credentials.
- Perform **virtual defacement** of the website.

---

### Impact of XSS Vulnerabilities

- **Confidentiality**: None / Partial (Low) / High
- **Integrity**: None / Partial (Low) / High
- **Availability**: None / Partial (Low) / High
- **Chaining**: Can be combined with other vulnerabilities for account takeover or remote code execution.

---

## How to Find And Exploit XSS Vulnerabilities

### Finding & Exploiting Reflected XSS

1. Identify endpoints where data is reflected.
   - Includes URL query strings, message bodies, file paths, and HTTP headers.
2. Submit a unique, arbitrary string (e.g., `test12564`) and monitor responses for reflections.
3. Determine the reflection context and test candidate payloads.

#### Example 1: A Tag Attribute Value

```html
<input type="text" name="address1" value="test12564">
```

#### Sample Payloads:

```html
<input type="text" name="address1" value="test12564"><script>alert(1)</script>">
<input type="text" name="address1" value="test12564" onfocus="alert(1)">
```

#### Example 2: A JavaScript String

```html
<script>var a = 'test12564'; var b = 123; ... </script>
```

#### Sample Payload:

```html
<script>var a = 'test12564'; alert(1); var foo=''; var b = 123; ... </script>
```
#### Example 3: An Attribute Containing a URL

```html
<a href="test12564">Click here ..</a>
```

#### Sample Payload:

```html
<a href="javascript:alert(1);">Click here ..</a>
<a href="#" onclick="javascript:alert(1)">Click here ..</a>
```
## Finding & Exploiting Stored XSS

1. **Identify endpoints** where user input is stored and later reflected.
2. **Submit a unique string** (e.g., `test12564`) and review the application for its appearance.
3. **Determine the context** of the reflected input and test payloads.

---

## Finding & Exploiting DOM-Based XSS

1. **Review client-side JavaScript** for sinks such as:
   - `document.write()`
   - `element.innerHTML`
   - `element.outerHTML`
   - `element.insertAdjacentHTML`
2. Use developer tools to **trace execution paths** and supply payloads.

---

## How to Prevent XSS Vulnerabilities

### Encode User-Controllable Data

- Apply combinations of HTML, URL, JavaScript, and CSS encoding based on the context.
- Use libraries like **Guava (Java)** for secure encoding.

### Filter Input

- Accept only expected or valid input.
  - **Example**: Age fields should only accept numeric values.

### Use Appropriate Response Headers

- Prevent XSS in HTTP responses with `Content-Type` and `X-Content-Type-Options` headers.

### Use Content Security Policy (CSP)

**Example**:
```http
Content-Security-Policy: default-src 'none'; script-src 'self'; connect-src 'self'; img-src 'self'; style-src 'self'; frame-ancestors 'self'; form-action 'self';
```

## Resources

- [Web Security Academy - Cross-site Scripting (XSS)](https://portswigger.net/web-security/cross-site-scripting)
- [OWASP - Cross-site Scripting (XSS)](https://owasp.org/www-community/attacks/xss)
- [OWASP Cross Site Scripting Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)
- [Cross-site Scripting (XSS) Cheat Sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)
- [OWASP XSS Filter Evasion Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html)
- [OWASP Content Security Policy Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Content_Security_Policy_Cheat_Sheet.html)
