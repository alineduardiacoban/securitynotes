# JSON Web Token (JWT) Attacks

## CONTENTS
1. [What is JWT?](#1-jwt---continued)
2. [Impact of JWT Vulnerabilities](#2-impact-of-jwt-vulnerabilities)
3. [Common Types of JWT Vulnerabilities](#3-common-types-of-jwt-vulnerabilities)
   - [Sensitive Information in JWT](#31-sensitive-information-in-jwt)
   - [JWT Stored in Insecure Location](#32-jwt-stored-in-insecure-location)
   - [JWT Transmitted Over Insecure Connection](#33-jwt-transmitted-over-insecure-connection)
   - [Expired JWTs Are Accepted](#34-expired-jwts-are-accepted)
   - [None Algorithm Accepted](#35-none-algorithm-accepted)
   - [Weak Signing Key](#36-weak-signing-key)
   - [Header Injection via jwk Parameter](#37-header-injection-via-jwk-parameter)
   - [Algorithm Confusion Attack](#38-algorithm-confusion-attack)
4. [Prevention of JWT Vulnerabilities](#4-prevention-of-jwt-vulnerabilities)
5. [References](#5-references)

---

## 2. What is JWT?

JSON Web Tokens (JWTs) are a compact and standardized format for transmitting cryptographically signed JSON data between parties. A JWT consists of three parts:
1. **Header**: Metadata about the token (e.g., algorithm, type).
2. **Payload**: Claims or data being transmitted (e.g., user roles, expiration time).
3. **Signature**: A cryptographic signature verifying the token's authenticity.

A JWT is encoded as a Base64URL string: `header.payload.signature`

- **Header**:
  - Contains metadata like `alg` (algorithm) and `typ` (type of token).
- **Payload**:
  - Contains claims like `sub` (subject), `exp` (expiration), `aud` (audience), etc.
- **Signature**:
  - Ensures the token has not been tampered with.

---

## 3. Impact of JWT Vulnerabilities

1. **Confidentiality**:
   - Sensitive information, if included in JWTs, can be exposed.
2. **Integrity**:
   - Malicious actors can modify claims if the signature isn’t properly verified.
3. **Availability**:
   - Weak signing mechanisms or long expiration times can disrupt secure operations.

---

## 4. Common Types of JWT Vulnerabilities

### 4.1. Sensitive Information in JWT

**Theory**  
Sensitive information (e.g., passwords, credit card numbers) is improperly included in the payload of JWTs.

**Steps to Find and Exploit**
1. Intercept a request in Burp Suite.
2. Decode the JWT payload and inspect its claims for sensitive data.

---

### 4.2. JWT Stored in Insecure Location

**Theory**  
JWTs stored in local storage or cookies without proper flags (e.g., `HttpOnly`, `Secure`) can be accessed by malicious scripts.

**Steps to Find and Exploit**
1. Open the browser’s **Application** tab.
2. Inspect **Cookies** or **Local Storage** for JWT tokens.
3. Run: `console.log(localStorage); console.log(document.cookie);`

---

### 4.3. JWT Transmitted Over Insecure Connection

**Theory**  
JWTs transmitted over HTTP (instead of HTTPS) are vulnerable to interception.

**Steps to Find and Exploit**
1. Use tools like Burp Suite to inspect traffic.
2. Look for JWTs transmitted via unencrypted HTTP connections.

---

### 4.4. Expired JWTs Are Accepted

**Theory**  
The application fails to enforce the `exp` claim, accepting tokens past their expiration time.

**Steps to Find and Exploit**
1. Intercept a request and decode the JWT to find the `exp` claim.
2. Replay the expired token to check if it is still accepted.

---

### 4.5. None Algorithm Accepted

**Theory**  
The application improperly accepts JWTs signed with the `none` algorithm, bypassing signature verification.

**Steps to Find and Exploit**
1. Intercept a request and decode the JWT header.
2. Change the `alg` field to `none` and remove the signature.
3. Replay the token to test acceptance.

---

### 4.6. Weak Signing Key

**Theory**  
The application uses a weak or brute-forceable signing key.

**Steps to Find and Exploit**
1. Use `hashcat` to brute-force the secret:

`hashcat -a 0 -m 16500 <YOUR-JWT> <path-to-wordlist>`

---

### 4.7. Header Injection via jwk Parameter

**Theory**  
The application improperly trusts embedded JWK (JSON Web Key) in the header.

**Steps to Find and Exploit**
1. Generate a malicious JWK.
2. Inject it into the `jwk` parameter in the JWT header.

---

### 4.8. Algorithm Confusion Attack

**Theory**  
The server is tricked into verifying JWTs using a different algorithm than intended (e.g., asymmetric vs. symmetric).

**Steps to Find and Exploit**
1. Change the `alg` parameter in the JWT header.
2. Forge a valid token using an unintended algorithm.

---

## 5. Prevention of JWT Vulnerabilities

1. **Avoid Sensitive Data in JWTs**:
- Never store passwords, credit card details, or other sensitive information in JWTs.

2. **Use Secure Storage**:
- Use `HttpOnly` and `Secure` flags for cookies.
- Prefer session storage over local storage for tokens.

3. **Verify Signature Properly**:
- Always verify the JWT signature using a secure library.
- Ensure the `alg` field is validated.

4. **Limit Token Lifetime**:
- Use short-lived tokens with refresh tokens.
- Always validate the `exp` claim.

5. **Secure Transmission**:
- Always use HTTPS for transmitting JWTs.
- Never send tokens in URLs.

6. **Enforce Strong Signing Keys**:
- Use robust algorithms like `RS256` or `ES256`.
- Rotate keys regularly.

7. **Validate Headers and Claims**:
- Whitelist allowed `alg` values.
- Validate claims like `aud`, `iss`, and `sub`.

---

## 6. References

- [PortSwigger Web Security Academy](https://portswigger.net/web-security/jwt)  
- [OWASP – JWT Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)  
- [JWT.io](https://jwt.io)  