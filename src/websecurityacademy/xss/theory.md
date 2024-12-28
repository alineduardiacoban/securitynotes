# Cross-Site Scripting (XSS)

## CONTENTS
1. [Prerequisites](#1-prerequisites)
2. [What is XSS?](#2-what-is-xss)
3. [Impact of XSS Vulnerabilities](#3-impact-of-xss-vulnerabilities)
4. [Common Types of XSS Vulnerabilities](#4-common-types-of-xss-vulnerabilities)
   - [Reflected XSS](#41-reflected-xss)
   - [Stored XSS](#42-stored-xss)
   - [DOM-Based XSS](#43-dom-based-xss)
5. [Prevention of XSS Vulnerabilities](#5-prevention-of-xss-vulnerabilities)
6. [References](#6-references)

---

## 1. Prerequisites

1. **Same-Origin Policy (SOP)**: A browser security rule that restricts resource sharing between origins. 
2. **Sink**: A location in the DOM where data is rendered (e.g., innerHTML).

---

## 2. What is XSS?

Cross-Site Scripting (XSS) is a web security vulnerability that allows attackers to inject malicious scripts into a web application. These scripts execute in the victim's browser, often leveraging their session and access permissions.

XSS occurs when:
1. An attacker injects a malicious script into a vulnerable application.
2. The injected script is executed in the browser of an unsuspecting user.
3. The script gains access to sensitive data, impersonates the user, or defaces the website.

### Example Scenario
A status message application reflects user input in its response:

**URL**:  
`https://example.com/status?message=Hello`

**Vulnerable Response**:  
`<p>Status: Hello</p>`

**Exploit**:  
`https://example.com/status?message=<script>alert(document.cookie);</script>`

**Exploit Response**:  
`<p>Status: <script>alert(document.cookie);</script></p>`

---

## 3. Impact of XSS Vulnerabilities

1. **Confidentiality**:
   - Stealing cookies, session tokens, or sensitive data.
2. **Integrity**:
   - Injecting malicious content or modifying application functionality.
3. **Availability**:
   - Disrupting the application's usability (e.g., infinite alert loops).

### Potential Use Cases for XSS Exploits:
- Masquerading as a victim user to perform unauthorized actions.
- Capturing user credentials via phishing forms.
- Injecting trojan functionality into a website.

---

## 4. Common Types of XSS Vulnerabilities

### 4.1. Reflected XSS

**Theory**  
Reflected XSS occurs when the application reflects user input directly in the response without validation or sanitization.

**Why This Happens**:
- Input is reflected in dynamic content, such as query strings or form submissions, without proper encoding or validation.

**Steps to Find**
1. Identify endpoints reflecting user input in the response.
2. Test reflection contexts (e.g., HTML, JavaScript, attributes).
3. Inject test strings to determine payload execution.

**Steps to Exploit**
1. Craft a malicious payload:
   - Example:
     `<script>alert(1);</script>`
2. Send the payload in a query parameter:
   - Example:
     `https://example.com/search?q=<script>alert(1);</script>`
3. Observe script execution in the browser.

---

### 4.2. Stored XSS

**Theory**  
Stored XSS occurs when user input is saved on the server and reflected in later responses, making it more persistent and dangerous.

**Why This Happens**:
- Applications save user input (e.g., comments, profiles) without sanitization or validation.

**Steps to Find**
1. Submit a unique test string in fields that store user input (e.g., comment sections).
2. Inspect application responses for reflections of the stored data.
3. Test multiple contexts where data is rendered (e.g., web pages, email templates).

**Steps to Exploit**
1. Inject a malicious payload into a comment or form field:
   - Example:
     `<script>alert(document.cookie);</script>`
2. Wait for the payload to execute when another user views the stored data.

---

### 4.3. DOM-Based XSS

**Theory**  
DOM-Based XSS arises when client-side JavaScript processes user input and injects it into the DOM without validation.

**Why This Happens**:
- Insecure JavaScript functions (e.g., `innerHTML`, `document.write`) process unvalidated data from sources like query strings or user inputs.

**Steps to Find**
1. Trace user input in the DOM using browser developer tools.
2. Identify unsafe sinks (e.g., `document.write`, `innerHTML`).
3. Test execution by injecting payloads:
   - Example:
     `<img src=x onerror=alert(1)>`

**Steps to Exploit**
1. Inject a payload into a vulnerable input source.
2. Confirm execution in the DOM by observing changes or alerts.

---

## 5. Prevention of XSS Vulnerabilities

### General Measures
1. **Input Validation**:
   - Accept only expected data formats (e.g., numbers for age fields).
2. **Output Encoding**:
   - Use context-aware encoding (e.g., HTML, JavaScript).
   - Escape characters like `<`, `>`, `&`, and quotes.

### Specific Mitigations
1. **Reflected XSS**:
   - Encode all user-supplied input in HTTP responses.
2. **Stored XSS**:
   - Validate and sanitize stored input before rendering it.
3. **DOM-Based XSS**:
   - Avoid using insecure DOM manipulation methods like `innerHTML`.

### Additional Protections
1. **Content Security Policy (CSP)**:
   - Example:
     `Content-Security-Policy: script-src 'self'; object-src 'none';`
2. **Secure HTTP Headers**:
   - Add headers to restrict browser behavior:
     - `X-Content-Type-Options: nosniff`
     - `X-XSS-Protection: 1; mode=block`

---

## 6. References

- [PortSwigger Web Security Academy – XSS](https://portswigger.net/web-security/cross-site-scripting)  
- [OWASP XSS Cheat Sheet](https://owasp.org/www-community/attacks/xss)  
- [XSS Filter Evasion Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html)  