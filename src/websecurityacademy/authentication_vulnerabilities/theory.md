# Authentication Vulnerabilities

## Table Of Contents
- [What Are Authentication Vulnerabilities](#what-are-authentication-vulnerabilities)
- [How To Find and Exploit Authentication Vulnerabilities](#how-to-find-and-exploit-authentication-vulnerabilities)
- [How To Prevent Authentication Vulnerabilities](#how-to-prevent-authentication-vulnerabilities)

---

## Preqrequisites

Authentication identifies the user and confirms that they are who they claim to be.

- **HTML form-based authentication**
- **Multi-factor mechanisms**
- **Windows-integrated authentication** (NTLM or Kerberos)

---

## What Are Authentication Vulnerabilities?

Authentication vulnerabilities arise from insecure implementation of authentication mechanisms in an application.

### Examples:
1. **Weak Password Requirements**
   - Very short or blank passwords
   - Common dictionary words or names
   - Password is the same as the username
   - Use of default passwords
   - Missing or ineffective MFA

   **Reference:** [CASMM Consumer Authentication Security Maturity Model](https://danielmiessler.com/blog/casmm-consumer-authentication-security-maturity-model/)

2. **Improper Restriction of Authentication Attempts**
   - Application permits brute force or other automated attacks:
     - Login page
     - OTP/MFA page
     - Change password page

     **Example:**
     - **Login Page:**
       - Username
       - Password
     - **OTP Page:**
       - Enter a one-time code from the Authenticator app.

3. **Verbose Error Messages**
   - Application outputs verbose error messages, allowing username enumeration.
     - Incorrect Username: `Sign in failed. Incorrect username.`
     - Incorrect Password: `Sign in failed. Incorrect password.`
     - This enables attackers to differentiate valid and invalid usernames.

4. **Vulnerable Transmission of Credentials**
   - Application uses unencrypted HTTP connections to transmit credentials.
     - Example:
       - **HTTP** vs **HTTPS**

5. **Insecure Forgot Password Functionality**
   - Weaknesses in forgotten password functionality often make it the weakest link in an application's authentication logic.

6. **Defects in Multistage Login Mechanisms**
   - Insecure implementation of MFA or other multi-stage login processes.

     **Example:**
     - **Request 1:**
       `POST /login-steps/first HTTP/1.1`
       `username=carlos&password=qwerty`
     - **Request 2:**
       `POST /login-steps/second HTTP/1.1`
       `verification-code=123456`

     **Exploit:** Change the `account` cookie to the victim's username and compromise their account.

7. **Insecure Storage of Credentials**
   - Storing passwords in plaintext, weak encryption, or weakly hashed formats.

---

### Impact of Authentication Vulnerabilities
- **Confidentiality:** Access to other users’ data.
- **Integrity:** Ability to update other users’ data.
- **Availability:** Deletion of user accounts and data.

---

## How to Find and Exploit Authentication Vulnerabilities

### Weak Password Complexity
- Review the website for descriptions of password rules.
- If self-registration is possible, attempt to register accounts with weak passwords:
  - Blank or very short passwords.
  - Common dictionary words or names.
  - Password is the same as the username.
- If password change is possible, attempt to change the password to weak values.

### Improper Restriction of Authentication Attempts
- Manually submit multiple failed login attempts.
- After 10 attempts, verify whether:
  - The account is locked out.
  - A successful login is still possible.
- Tools: Hydra, Burp Intruder

### Verbose Error Messages
- Submit requests with:
  - A valid username and an invalid password.
  - An invalid username.
- Compare responses for differences in:
  - Status codes
  - Redirects
  - Processing time

### Vulnerable Transmission of Credentials
- Monitor traffic during login for:
  - Credentials submitted via URL or cookie.
  - Credentials transmitted over HTTP instead of HTTPS.

### Insecure Forgot Password Functionality
- Test forgotten password functionality for:
  - Username enumeration
  - Predictable recovery URLs
  - Long-lived recovery tokens

### Defects in Multistage Login Mechanisms
- Walk through the entire login flow with controlled accounts.
- Look for username enumeration or brute force vulnerabilities.

---

## How to Prevent Authentication Vulnerabilities

1. **Enforce Multi-Factor Authentication (MFA).**
2. Only **POST** requests should be used to transmit credentials to the 
server.
3. **Change all default credentials.**
4. **Always use HTTPS for secure transmissions.**
5. **Hash and salt stored credentials with secure algorithms.**
6. **Use identical, generic error messages for failed logins.**
7. **Implement strong password policies** (e.g., real-time feedback using [zxcvbn](https://github.com/dropbox/zxcvbn)).
8. **Enable account lockout mechanisms** after multiple failed login attempts.

---

## Resources
- [Web Security Academy – Authentication Vulnerabilities](https://portswigger.net/web-security/authentication)
- [OWASP Web Security Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)
- [OWASP Top 10 – Authentication Failures](https://owasp.org/Top10/)
- [OWASP Application Security Verification Standard (ASVS)](https://owasp.org/www-pdf-archive/OWASP_Application_Security_Verification_Standard_4.0-en.pdf)
