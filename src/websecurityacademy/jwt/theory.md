# JSON Web Tokens (JWT)

## Table Of Contents

- [What are JWTs](#what-are-jwts)
- [What Are JWT Vulnerabilities](#what-are-jwt-vulnerabilities)
- [How To Find And Exploit JWT Vulnerabilities](#how-to-find-and-exploit-jwt-vulnerabilities)
- [How To Secure JWTs](#how-to-secure-jwts)

---

## What Are JWTs?

JSON Web Tokens (JWTs) are a standardized format for sending cryptographically signed JSON data between systems.

### Components of a JWT
1. **Header**  
   Contains metadata such as:
   - `alg` (signature algorithm, e.g., HS256, RS256)
   - `typ` (token type)
   - Optional fields: `kid`, `jwk`, `jku`

2. **Payload**  
   Contains claims such as:
   - `iss` (issuer)
   - `sub` (subject)
   - `aud` (audience)
   - `exp` (expiration timestamp)
   - `iat` (issued at timestamp)
   - Other custom claims (e.g., `role`, `email`)

3. **Signature**  
   Verifies the integrity of the token:
   - **Symmetric algorithms**: Use a single key for signing and verification.
   - **Asymmetric algorithms**: Use a private key for signing and a public key for verification.

---

## What Are JWT Vulnerabilities?

### Common Causes of JWT Vulnerabilities:
- Weak or misconfigured cryptographic algorithms.
- Poor key management.
- Issues with token expiration and revocation.
- Signature bypasses.
- Information leakage.

### Examples of Vulnerabilities:
1. JWT contains sensitive information.
2. JWT is stored insecurely (e.g., in LocalStorage or cookies without proper flags).
3. JWT transmitted over an insecure connection (e.g., HTTP).
4. Long or missing expiration times.
5. Expired JWTs are still accepted.
6. Weak or brute-forceable signing keys.
7. Header injection vulnerabilities via `jwk`, `jku`, or `kid`.
8. Algorithm confusion attacks.

---

## How to Find and Exploit JWT Vulnerabilities

### Example Vulnerabilities and Exploitation Steps:

#### Sensitive Information in JWT
1. Intercept the request in Burp.
2. Under the **JSON Web Token** subtab, review the decoded payload.
3. Identify any sensitive information, such as passwords or personal data.

#### JWT Stored in Insecure Location
1. Open the browser's Developer Tools.
2. Inspect **LocalStorage** or **Cookies** for stored JWTs.
3. Check if cookies are missing `HttpOnly` and `Secure` flags.

#### Long or Missing Expiration Time
1. Decode the JWT payload and look for the `exp` claim.
2. If the expiration is too far in the future or missing, test access to expired tokens.

#### Signature Bypass
1. Modify the JWT signature using Burp Repeater.
2. Test if the server accepts the modified signature without validation.

#### None Algorithm
1. Change the `alg` field in the JWT header to `none`.
2. Remove the signature and replay the token to test if the server accepts it.

#### Weak Signing Keys
1. Extract the JWT and use a tool like `hashcat` to brute force the signing key:

`hashcat -a 0 -m 16500 <YOUR-JWT> /path/to/jwt.secrets.list`


---

## How to Secure JWTs

### Best Practices:
1. **Avoid storing sensitive information** in the JWT payload.
2. **Use robust libraries** to handle JWTs securely.
3. **Verify signatures** thoroughly, ensuring proper methods are used.
4. **Store JWTs securely**:
- Use `SessionStorage` for Authorization headers.
- Use cookies with `HttpOnly` and `Secure` flags if applicable.
5. **Transmit JWTs securely**:
- Use HTTPS.
- Avoid sending JWTs in URL parameters.
6. **Opt for strong signing algorithms**:
- Prefer `RS256` or `ES256` over weaker algorithms like `HS256`.
7. **Enforce expiration dates**:
- Use short-lived tokens with refresh mechanisms.
- Ensure the `exp` claim is set for all tokens.
8. **Implement key management**:
- Rotate keys regularly.
- Securely store signing keys.
9. **Restrict claims** to follow the principle of least privilege.
10. **Prevent header injection attacks**:
 - Whitelist permitted public keys and hosts for `jwk` and `jku` headers.

---

## Resources

- [Web Security Academy - JWT Attacks](https://portswigger.net/web-security/jwt)
- [Web Security Academy - JWT Labs](https://portswigger.net/web-security/all-labs#jwt)
- [OWASP - JSON Web Token Testing Guide](https://owasp.org/www-project-web-security-testing-guide/latest/4Web_Application_Security_Testing/06-Session_Management_Testing/10-Testing_JSON_Web_Tokens)
- [What Are Refresh Tokens and How to Use Them Securely](https://auth0.com/blog/refresh-tokens-what-are-they-and-when-to-use-them/)
- [jsonwebtoken Signature Bypass Advisory](https://github.com/advisories/GHSA-qwph-4952-7xr6)
- [node-jose Improper Signature Validation](https://nvd.nist.gov/vuln/detail/CVE-2018-0114)
- [PyJWT Algorithm Confusion](https://www.vicarius.io/vsociety/posts/risky-algorithms-algorithm-confusion-in-pyjwt-cve-2022-29217)
