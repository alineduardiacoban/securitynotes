# Authentication Vulnerabilities
---

## Preqrequisites

Authentication identifies the user and confirms that they are who they claim to be.

- **HTML form-based authentication**
- **Multi-factor mechanisms**
- **Windows-integrated authentication** (NTLM or Kerberos)

---

## What Are Authentication Vulnerabilities?

Authentication vulnerabilities arise from insecure implementation of authentication mechanisms in an application.

---

## Types Of Authentication Vulnerabilities:

### Weak Password Requirements
   - Very short or blank passwords
   - Common dictionary words or names
   - Password is the same as the username
   - Use of default passwords
   - Missing or ineffective MFA

   **Reference:** [CASMM Consumer Authentication Security Maturity Model](https://danielmiessler.com/blog/casmm-consumer-authentication-security-maturity-model/)

### Improper Restriction of Authentication Attempts
   - Application permits brute force or other automated attacks:
     - Login page
     - OTP/MFA page
     - Change password page

### Verbose Error Messages
   - Application outputs verbose error messages, allowing username enumeration.

### Vulnerable Transmission of Credentials
   - Application uses unencrypted HTTP connections to transmit credentials.

### Insecure Forgot Password Functionality
   - Design weaknesses in forgotten password functionality often make it the weakest link in an application's authentication logic

### Defects in Multistage Login Mechanisms
   - Insecure implementation of MFA or other multi-stage login processes.

### Insecure Storage of Credentials
   - Storing passwords in plaintext, weak encryption, or weakly hashed formats.

### Impact of Authentication Vulnerabilities
* Unauthorized access to the application.
   * **C**onfidentiality: Access to other users’ data.
   * **I**ntegrity: Ability to update other users’ data.
   * **A**vailability: Deletion of user accounts and data.
* Can sometimes be chained with other vulnerabilities to gain
remote code execution on the host operating system.
---

## How to Find And Exploit Authentication Vulnerabilities

### Weak Password Complexity Requirements
* Review the website for any description of the
rules.
* If self registration is possible, attempt to
register several accounts with different kinds
of weak passwords to discover what rules are
in place.
  * Very short or blank.
  * Common dictionary words or names.
  * Password is the same as the username.
* If you control a single account and password
change is possible, attempt to change the
password to various weak values.

### Improper Restriction of Authentication Attempts
* Manually submit several bad login attempts for an
account you control.
* After 10 failed login attempts, if the application does not
return a message about account lockout, attempt to log
in correctly. If it works, then there is no lockout
mechanism.
  * Run a brute force attack to enumerate the valid
password. Tools: Hydra, Burp Intruder, etc.
* If the account is locked out, monitor the requests and
responses to determine if the lockout mechanism is
insecure.
**NOTE:** Apply this test on all authentication pages.

### Verbose Error Messages
* Submit a request with a valid username and an invalid
password.
* Submit a request with an invalid username.
* Review both responses for any differences in the status
code, any redirects, information displayed on the screen,
HTML page source, or even the time to process the
request.
* If there is a difference, run a brute force attack to
enumerate the list of valid usernames in the application.
**NOTE:** Apply this test on all authentication pages.

### Vulnerable Transmission of Credentials
* Perform a successful login while monitoring all traffic in both
directions between the client and server.
* Look for instances where credentials are submitted in a URL
query string or as a cookie, or are transmitted back from the
server to the client.
* Attempt to access the application over HTTP and if there are
any redirections to HTTPS.

### Insecure Forgot Password Functionality
* Identify if the application has any forgotten password functionality.
* If it does, perform a complete walk-through of the forgot password functionality using an account you have control of while intercepting the requests / responses in a proxy.
* Review the functionality to determine if it allows for username enumeration or brute-
force attacks.
* If the application generates an email containing a recovery URL, obtain a number of these URLs and attempt to identify any predictable patterns or sensitive information included in the URL. Also check if the URL is long lived and does not expire.

### Defects in Multistage Login Mechanisms
* Identify if the application uses a multistage login mechanism.
* If it does, perform a complete walk-through using an account you have control of
while intercepting the requests / responses in a proxy.
* Review the functionality to determine if it allows for username enumeration or brute-
force attacks.

#### Insecure Storage Of Credentials
* Review all the application’s authentication related functionality. If you find any
instances where the user’s password is transmitted to the client plaintext or obfuscated, this indicates the passwords are being stored insecurely.
* If you gain remote code execution (RCE) on the server, review the database to
determine if the passwords are stored insecurely.
* Conduct technical interviews with the developers to review how passwords are
stored in the backend database.

---

## How to Prevent Authentication Vulnerabilities

* Wherever possible, implement multi-factor authentication.
* Change all default credentials.
* Always use an encrypted channel / connection (HTTPS) when
sending user credentials.
* Only POST requests should be used to transmit credentials to the
server.
* Stored credentials should be hashed and salted using
cryptographically secure algorithms.
* Use identical, generic error messages on the login form when the
user enters incorrect credentials.
* Implement an effective password policy that is compliant with NIST 800-63-b’s guidelines.
  * Use a simple password checker to provide real time feedback on
  the strength of the password. For example: zxcvbn JavaScript
  library.
* Implement robust brute force protection on all authentication pages.
* Audit any verification or validation logic thoroughly to eliminate flaws.

---

## Resources
- [Web Security Academy – Authentication Vulnerabilities](https://portswigger.net/web-security/authentication)
- [OWASP Web Security Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)
- [OWASP Top 10 – Authentication Failures](https://owasp.org/Top10/)
- [OWASP Application Security Verification Standard (ASVS)](https://owasp.org/www-pdf-archive/OWASP_Application_Security_Verification_Standard_4.0-en.pdf)
