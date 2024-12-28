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

### What is XSS?
Cross-Site Scripting (XSS) vulnerabilities allow attackers to inject malicious client-side scripts into an application. These scripts execute in the victim user's browser, leveraging their session and access permissions.

**Key Concepts**:
- **Same-Origin Policy (SOP)**: A security rule in browsers that restricts how resources are accessed across origins.
- **Sink**: A location in the DOM where data is rendered (e.g., `innerHTML`).
- **Payload**: The malicious script injected by the attacker.

---

## 2. What is XSS?

An attacker exploits an XSS vulnerability to execute arbitrary scripts in the victim's browser, leveraging their access and permissions on the vulnerable application.

**Example Scenario**:
1. An attacker injects a malicious payload:
   `<script>alert('XSS');</script>`
2. The application renders this payload in the victim’s browser, executing the attacker’s script.

**What XSS Can Be Used For**:
- Stealing cookies and session tokens.
- Impersonating the victim user.
- Defacing the website or injecting trojan functionality.

---

## 3. Impact of XSS Vulnerabilities

The impact of XSS depends on the vulnerability type and the attacker's goals:

1. **Confidentiality**:
   - Can expose sensitive user data like cookies and tokens.

2. **Integrity**:
   - Allows modification or injection of malicious content into the application.

3. **Availability**:
   - Can disrupt the application's functionality or usability.

**Chaining Vulnerabilities**:
- XSS is often chained with other vulnerabilities to achieve a larger impact, such as full account takeover or remote code execution.

---

## 4. Common Types of XSS Vulnerabilities

### 4.1. Reflected XSS

**Theory**  
Reflected XSS occurs when the application directly reflects user input in the response without proper validation or encoding.

**Steps to Find**
1. Identify endpoints that reflect user input:
   - Inspect URL query strings, POST parameters, and HTTP headers.
   - Use test strings like `test1234` to locate reflections in the response.
2. Test injection points:
   - Replace `test1234` with payloads like:
     `<script>alert(document.cookie);</script>`

**Steps to Exploit**
1. Craft a malicious URL:
   `https://example.com/search?q=<script>alert('XSS')</script>`
2. Share the link with a victim or embed it in a phishing email.

---

### 4.2. Stored XSS

**Theory**  
Stored XSS arises when malicious input is saved in the application (e.g., in a database) and later displayed to other users.

**Steps to Find**
1. Locate fields where user input is stored:
   - Comments, profile fields, or message boards.
2. Submit payloads in these fields:
   `<script>alert(1);</script>`

**Steps to Exploit**
1. Inject the payload into a field like a comment box.
2. When another user views the comment, the payload executes.

---

### 4.3. DOM-Based XSS

**Theory**  
DOM-based XSS arises when client-side JavaScript processes user input and directly manipulates the DOM.

**Vulnerable Code Example**:
```html
var search = document.getElementById('search').value; 
results.innerHTML = "You searched for: " + search;
```

**Steps to Find**
1. Identify sources (user-controlled data) and sinks (DOM manipulation methods like `innerHTML`).
2. Inject payloads:
   `<img src=1 onerror=alert('XSS')>`

**Steps to Exploit**
1. Submit the payload into a vulnerable field.
2. When the DOM processes the payload, it executes the script.

---

## 5. Prevention of XSS Vulnerabilities

### Encode User Input
1. Apply proper encoding to user input before rendering it in the browser:
   - HTML encoding: Convert `<` to `&lt;` and `>` to `&gt;`.
   - JavaScript encoding: Escape special characters like quotes.
2. Use secure libraries for encoding:
   - Example: Guava (Java), ESAPI (OWASP).

### Filter Input on Arrival
1. Define strict validation rules for each input field:
   - Example: Allow only alphanumeric characters in usernames.
2. Reject input that doesn't match the expected format.

### Use Secure Headers
1. Add HTTP response headers to restrict browser behavior:
   - `Content-Type`: Ensure the correct MIME type is used.
   - `X-Content-Type-Options: nosniff`.

### Content Security Policy (CSP)
1. Deploy a CSP to limit the execution of unauthorized scripts:
   
   `Content-Security-Policy: script-src 'self'; object-src 'none';`

---

## 6. References

- [PortSwigger Web Security Academy](https://portswigger.net/web-security/cross-site-scripting)  
- [OWASP – XSS](https://owasp.org/www-community/attacks/xss)  
- [OWASP XSS Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)  
- [OWASP XSS Filter Evasion Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html)  
- *Web Application Hacker’s Handbook*, Chapter 13: Attacking Users.  
