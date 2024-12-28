# Cross-Site Request Forgery (CSRF)

## CONTENTS
1. [What is CSRF](#1-csrf---continued)
2. [Impact of CSRF Vulnerabilities](#2-impact-of-csrf-vulnerabilities)
3. [Common Types of CSRF Vulnerabilities](#3-common-types-of-csrf-vulnerabilities)
   - [GET-Based CSRF](#31-get-based-csrf)
   - [POST-Based CSRF](#32-post-based-csrf)
4. [Prevention of CSRF Vulnerabilities](#4-prevention-of-csrf-vulnerabilities)
5. [References](#5-references)

---

## 1. What is CSRF?

CSRF (Cross-Site Request Forgery) occurs when an attacker tricks a user into performing unintended actions on a web application where the user is authenticated. This often exploits session-based authentication mechanisms (e.g., cookies).

**Key Conditions for CSRF**:
1. **A Relevant Action**: The application performs actions that change state (e.g., changing email addresses, transferring funds).
2. **Cookie-Based Session Handling**: The application relies on session cookies for authentication.
3. **No Unpredictable Request Parameters**: The request parameters can be guessed or forged by the attacker.


In a CSRF attack:
1. A user authenticates with a web application (e.g., bank.com) and receives a session cookie.
2. The attacker tricks the user into visiting a malicious page or clicking a link.
3. The malicious request (crafted by the attacker) uses the user's session to perform unauthorized actions on the web application.

**Example Scenario**:
- A user is logged into `bank.com`.
- The attacker sends the user a malicious link:  
  `https://bank.com/email/change?email=attacker@gmail.com`.
- When clicked, the victim unknowingly changes their email address to the attacker's.

---

## 2. Impact of CSRF Vulnerabilities

The impact of a CSRF attack depends on the functionality exploited:

1. **Confidentiality**:
   - Can range from **none** to **high**, depending on what data is accessed.
2. **Integrity**:
   - Usually **low or high**, as the attacker can modify data (e.g., change account settings).
3. **Availability**:
   - May lead to **none**, **low**, or **high** disruption depending on the application’s functionality.

---

## 3. Common Types of CSRF Vulnerabilities

### 3.1. GET-Based CSRF

**Theory**  
GET-based CSRF exploits actions triggered by HTTP GET requests, where sensitive operations are performed without validation.

**Why This Happens**:
- Sensitive actions are incorrectly implemented using GET requests.
- Applications fail to differentiate legitimate requests from malicious ones.

**Steps to Find**
1. Map the application to identify sensitive GET endpoints (e.g., changing email, transferring funds).
2. Craft malicious URLs for these actions.

**Steps to Exploit**
1. Send the victim a crafted URL, such as:  
   `https://bank.com/email/change?email=attacker@gmail.com`.
2. Use HTML tags like `<img>` or `<iframe>` to execute the request invisibly:
   ```html
   <img src="https://bank.com/email/change?email=attacker@gmail.com" width="0" height="0">

### 3.2. POST-Based CSRF

**Theory**  
POST-based CSRF exploits actions performed using HTTP POST requests, often hidden within forms.

**Why This Happens**:
- Applications lack CSRF defenses such as CSRF tokens.
- Attackers can forge forms to simulate legitimate requests.

**Steps to Find**
1. Identify endpoints that accept POST requests for sensitive actions.
2. Analyze the required parameters to craft a malicious request.

**Steps to Exploit**
1. Create an exploit using an auto-submitting form:
   ```html
   <form action="https://bank.com/email/change" method="POST">
       <input type="hidden" name="email" value="attacker@gmail.com">
   </form>
   <script>document.forms[0].submit();</script>

2. Host the malicious form on the attacker's website and trick the victim into visiting it.

---

## 4. Prevention of CSRF Vulnerabilities

### Primary Defense: CSRF Tokens

1. **Generate Unpredictable Tokens**:
   - Tokens should have high entropy, tied to user sessions.

2. **Transmit Securely**:
   - Include tokens as hidden form fields or custom headers for POST requests.
   - Avoid using query strings or cookies to transmit tokens.

3. **Validate Tokens**:
   - Compare the token in the request with the one stored server-side.
   - Reject requests with invalid or missing tokens.

---

### Additional Defense: SameSite Cookies

Configure cookies with the `SameSite` attribute:
- **Strict**: Cookies are sent only with same-origin requests.
- **Lax**: Cookies are sent with top-level navigation GET requests.
- **None**: Cookies are sent across origins but require `Secure`.

**Example**:
```plaintext
Set-Cookie: session=test; SameSite=Strict
```

### Inadequate Defense: Referer Header

**Limitations**:
- Referer headers can be spoofed.
- Some browsers omit the Referer header, bypassing checks.

**Why It’s Insufficient**:
- Applications relying solely on the Referer header are vulnerable to bypasses if headers are absent or manipulated.

---

## 5. References

- [PortSwigger Web Security Academy](https://portswigger.net/web-security/csrf)  
- [OWASP – CSRF](https://owasp.org/www-community/attacks/csrf)  
- [Cross-Site Request Forgery Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)  
