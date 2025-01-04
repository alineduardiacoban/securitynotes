# Cross-Site Scripting (XSS)
---

## What Are XSS Vulnerabilities?

Cross-Site Scripting (XSS) vulnerabilities allow an attacker to inject malicious client-side scripts in the application that get executed in the web browser of
the victim user.

### Same-Origin Policy
Same-Origin Policy (SOP) is a rule that is enforced by browsers to control access to data between web applications.
* This does not prevent writing between
web applications, it prevents reading
between web applications.
* Access is determined based on the origin.

### Types of XSS

#### Reflected XSS
* Arises when the application directly
reflects client-supplied input in the response of the
request in an unsafe way.
   * Vulnerable Request
      * `https://insecure-website.com/status?message=All+is+well.`
   * Response
      * `<p>Status: All is well.</p>`
   * Exploit
      * `https://insecure-website.com/status?message=<script>alert(document.cookie);</script>`
   * Exploit Response
      * `<p>Status: <script>alert(document.cookie);</script> </p>`

#### Stored XSS
* Arises when the application stores client
supplied input in an unsafe way and includes the input in
later responses.

#### DOM-Based XSS
* Arises when JavaScript takes data from
client-supplied input, and passes it to a sink that
supports dynamic code execution in the DOM.
   * Vulnerable Code
   ```javascript
   var search = document.getElementById('search').value;
   var results = document.getElementById('results’);
   results.innerHTML = 'You searched for: ' + search;
   ```
   * Exploit
      * `<img src=1 onerror=alert(1)>`

### What Can XSS Be Used For?

* Impersonate or masquerade as the victim user.
* Carry out any action that the user is able to perform.
* Read any data that the user is able to access.
* Inject trojan functionality into the website that captures the user’s
login credentials.
* Perform virtual defacement of the website.

### Impact of XSS Vulnerabilities
* Depends on the XSS vulnerability and attacker goal.
   * Confidentiality – can be None / Partial (Low) / High
   * Integrity – can be None / Partial (Low) / High
   * Availability – can be None / Partial (Low) / High
* Usually chained with other vulnerabilities to achieve maximum impact such as full account takeover and even remote code execution.

---

## How to Find And Exploit XSS Vulnerabilities

### Finding & Exploiting Reflected XSS

* Identify every endpoint where data is reflected.
   * This includes parameters or other data within the URL query string and
message body, and the URL file path. It also includes HTTP headers.
   * Submit a unique arbitrary string that is unlikely to be affected by any XSS-
filters.
   * Example String: test12564
* Monitor the application’s responses for appearance of this string.
* For each identified endpoint, determine the reflection context.
* Test candidate payloads.
* Test the attack in the browser.

#### Example 1: A Tag Attribute Value

```html
<input type="text" name="address1" value="test12564">
```

#### Sample XSS Payloads:

```html
<input type="text" name="address1" value="test12564"><script>alert(1)</script>">
OR
<input type="text" name="address1" value="test12564" onfocus="alert(1)">
```

#### Example 2: A JavaScript String

```html
<script>var a = 'test12564'; var b = 123; ... </script>
```

#### Sample XSS Payload:

```html
<script>var a = 'test12564'; alert(1); var foo=''; var b = 123; ... </script>
```
#### Example 3: An Attribute Containing a URL

```html
<a href="test12564">Click here ..</a>
```

#### Sample XSS Payloads:

```html
<a href="javascript:alert(1);">Click here ..</a>
OR
<a href="#" onclick="javascript:alert(1)">Click here ..</a>
```
## Finding & Exploiting Stored XSS

* Identify every endpoint where data is reflected in the response of the user.
   * This includes parameters or other data within the URL query string and message body,
and the URL file path. It also includes HTTP headers.
   * Submit a unique arbitrary string that is unlikely to be affected by any XSS-filters.
      * Example String: test12564
   * User controllable data entered in one location may be displayed in numerous places
throughout the application.
   * Review all the application’s content and functionality to identify any instances where this string is displayed back to the browser.
* For each identified endpoint, determine the context.
* Test candidate payloads.
* Test the attack in the browser.

---

## Finding & Exploiting DOM-Based XSS

* Review every piece of client-side JavaScript for sinks that may be used to
access the DOM and can be controlled by client-supplied input.
   * Example sinks include:
      * `document.write()`
      * `element.innerHTML`
      * `element.outerHTML`
      * `element.insertAdjacentHTML`
* Once you identify a source that leads to a sink, supply a unique string and
use developer tools to follow the path of execution to the sink.
* Test candidate payloads.

---

## How to Prevent XSS Vulnerabilities
* Encode any user-controllable data on output. Depending on the output context,
this might require applying combinations of HTML, URL, JavaScript, and CSS
encoding. DO NOT implement your own encoding library. Instead, use an existing
well-known secure library.
   * Example: Guava Java library for HTML encoding.
* Filter input on arrival. Only accept input based on what is expected or valid.
   * Example: Age field should only contain numbers.
* Use appropriate response headers. To prevent XSS in HTTP responses that aren't
intended to contain any HTML or JavaScript, you can use the Content-Type and X-
Content-Type-Options headers to ensure that browsers interpret the responses in
the way you intend.
* Use Content Security Policy (CSP) to help detect and mitigate cross-site scripting
attacks.
```http
Content-Security-Policy: default-src 'none'; script-src 'self'; connect-src 'self'; img-src 'self'; style-src 'self'; frame-ancestors 'self'; form-action 'self';
```

---

## Resources

- [Web Security Academy - Cross-site Scripting (XSS)](https://portswigger.net/web-security/cross-site-scripting)
- [OWASP - Cross-site Scripting (XSS)](https://owasp.org/www-community/attacks/xss)
- [OWASP Cross Site Scripting Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)
- [Cross-site Scripting (XSS) Cheat Sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)
- [OWASP XSS Filter Evasion Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html)
- [OWASP Content Security Policy Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Content_Security_Policy_Cheat_Sheet.html)
