# JSON Web Tokens (JWT)
---

## What Are JWTs?

JSON Web Tokens (JWTs) are a standardized format for sending cryptographically signed JSON data between systems.

### JWT Components
#### Header  
The first component is the header which contains metadata.
* alg — Signature algorithm.
   * Symmetric key algorithms such as HS256.
   * Asymmetric key algorithms such as RS256.
   * No signature such as the “none” algorithm.
* typ — Token type.
* kid — Key ID for identifying the key used.
* jwk — Embedded JSON object representing the key.
* jku — URL to retrieve key information.

#### Payload
The second component is the payload which contains a set of
claims.
* iss — Identifies the issuer of the JWT.
* sub — Identifies the subject of the JWT.
* aud — Identifies the audience that the JWT is intended for.
* exp — A timestamp (in UNIX time format) indicating when the JWT
expires and should no longer be accepted.
* iat — A timestamp indicating when the JWT was issued.
* Claims such as name, role, email, etc.

#### Signature
The third component is the signature which contains the signature
of the token.
* Symmetric Algorithm
   * The server uses a single key to both sign and verify the token.
* Asymmetric Algorithm
   * This consists of a private key, which the server uses to sign the token, and a public key that can be used to verify the signature.

### What Causes JWT Vulnerabilities?
* Weak or misconfigured cryptographic algorithms.
* Poor key management.
* Token expiration & revocation issues.
* Signature bypasses.
* Information leakage.

---

## How to Find And Exploit JWT Vulnerabilities?

### JWT Vulnerabilities
* JWT contains sensitive information.
* JWT is stored in an insecure location.
* JWT is transmitted over an insecureconnection.
* JWT expiration time is too lengthy or missing.
* Expired JWT is accepted.
* JWT signature is not verified, or arbitrary signatures are accepted.
* “None” algorithm is accepted / JWT without a signature are accepted.
* JWT is signed with a weak or brute-forceable key.
* JWT is vulnerable to header injection via the jwk parameter.
* JWT is vulnerable to header injection via jku parameter.
* kid parameter is vulnerable to injection attacks.
* Algorithm confusion attack.


### JWT Contains Sensitive Information
* JWT tokens stored in LocalStorage or in a cookie that is not configured securely.
* Insecurely Configured Cookie (Missing HttpOnly and Secure Flags)
`Set-Cookie: session=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...; Path=/; Domain=example.com;`
* JWT Stored in Local Storage

***Steps to find and exploit:***
1) In the browser, right click and select Inspect and select the Application tab.
2) There you can see the Cookies jar and Local storage.
3) In the Console tab, run localStorage or document.cookie.

### JWT Transmitted Over Insecure Connection
* JWT tokens transmitted over an unencrypted channel (such as HTTP instead of HTTPS).

### Long JWT Expiration Time
* JWT token does not expire or has a very long expiration time.
```xml
{
"sub": "user123",
"name": "Alice Smith",
"email": "alice.smith@example.com",
"role": "admin",
"exp": 1821869841 // Example expiration timestamp (September 25, 2027)
}
```
***Steps to find and exploit:***
1) Intercept the request in Burp.
2) Under the Request tab, click on the JSON Web Token subtab.
3) Review the decoded payload to identify the expiration time set in the “exp” parameter.
4) Confirm the expiration time.

### Expired JWTs are Accepted
The application accepts expired JWTs.
```xml
{
"sub": "user123",
"name": "Alice Smith",
"email": "alice.smith@example.com",
"role": "admin",
"exp": 1726790400 // Example expiration timestamp (September 20, 2025)
}
```
***Steps to find and exploit:***
1) Intercept the request in Burp.
2) Under the Request tab, click on the JSON Web Token subtab.
3) Review the decoded payload to identify the expiration time set in the “exp” parameter.
4) Replay the token to access an authenticated page.

### JWT Signature Not Verified
The application does not verify the signature of the JWT / accepts arbitrary signatures. This is sometimes
possible because developers confuse the verify() and decode() methods and only pass incoming tokens
to the decode function.

```csharp
{
app.use((req, res, next) => {
const token = req.headers['authorization']?.split(' ')[1];
if (token) {
// Decode the token without verifying
const decoded = jwt.decode(token);
...
```

***Steps to find and exploit:***
1) Intercept an authenticated request in
Burp.
2) 3) Send the request to Repeater.
In Repeater, change the signature to
an arbitrary value.
4) Click on Send.

### None Algorithm Accepted
The application accepts a token with the “none” algorithm and does not properly verify the algorithm specified in the JWT header.
***Steps to find and exploit:***
1) Intercept an authenticated request in
Burp.
2) Send the request to Repeater.
3) In Repeater, change the algorithm field to “none” in the JSON Web Token subtab.
4) Go back to the Pretty subtab and remove the signature of the token.
5) Click on Send.

### Weak / Brute-forceable Signing Key
The application uses a weak, predictable, or easily guessable signing key, that can be easily brute-forced.

***Steps to find and exploit:***

**Part 1**: Brute force the secret key
1) Intercept an authenticated request in Burp.
2) Extract the token and use hashcat to brute force the key.
hashcat -a 0 -m 16500 <YOUR-JWT> /path/to/jwt.secrets.list

**Part 2**: Generate a forged signing key
1) Use Burp Decoder to base64 encode the secret that was brute-forced.
2) Visit the JWT Editor tab and click on New
Symmetric Key. Click Generate.
3) Replace the generated value for the k property with the base64-encoded secret.
4) Click OK to save the key.

**Part 3**: Modify and sign the JWT
1) Go back to the request and alter the token.
2) At the bottom of the tab, click Sign and select the key that was generated.
3) Click OK.
4) Click on Send.

### Header Injection via the jwk Parameter
The application trusts any key that is embedded in the jwk header.
***Steps to find and exploit:***
1) Go to the JWT Editor tab.
2) Click New RSA Key.
3) Click Generate and then click OK to save the key.
4) Intercept an authenticated request and send it to Repeater.
5) In Repeater, alter the token and then click on the Attack button.
6) Select Embedded JWK. When prompted, select your newly
generated RSA key and click OK.
7) Send the request.

### Header Injection via the jku Parameter
The application trusts any key that is embedded in the jku (JWK Set URL) header.

***Steps to find and exploit:***

**Part 1** – Generate & Upload a malicious JWK
Set
1) Generate a key pair in the JWT Editor tab
and store it in the attacker controlled
server.

**Part 2** - Modify and sign the JWT
1) Intercept an authenticated request in Burp
and send it to Repeater.
2) Add a new jku parameter to the header of the JWT. Set its value to the URL of your JWK Set on the attacker server.
3) Click Sign, and select the generated RSA
key.
4) Send the request.
---

### Header Injection via the kid Parameter
The kid header parameter is vulnerable to injection attacks.

***Steps to find and exploit:***

**Part 1** - Generate a suitable signing key
1) Go to the JWT Editor tab.
2) Click New Symmetric Key and click Generate to generate a new key in JWK.
3) Replace the k property with a Base64-encoded null byte (AA==).
4) Click OK.
**Part 2** – Modify and sign the token
1) Intercept an authenticated request in Burp and send it to Repeater.
2) Change the kid parameter to `../../../../../../../dev/null`.
3) Click on Sign and send the request.

### Algorithm Confusion Attack
Algorithm confusion attacks (also known as key confusion attacks) occur when the attacker is able to force the server to verify the signature of the JWT using a different algorithm than is intended by the application developers.

**Part 1** - Obtain the server's public key
1) Obtain the server’s public key
**Part 2** - Generate a malicious signing key
1) Generate an RSA key pair in the JWT
Editor tab.
**Part 3** – Modify and sign the token
1) Intercept an authenticated request in Burp and send it to Repeater.
2) Change the value of the alg parameter to HS256.
3) Click on Sign and send the request.

---

## How to Secure JWTs

* Avoid storing sensitive information in the JWT.
* Use an up-to-date library for handling JWTs.
* Perform robust signature verification on JWTs.
* Store JWTs securely. If you’re using an Authorization header to transmit JWTs then opt
for Session Storage instead of Local Storage. Similarly, if you’re using cookies to store
JWTs, then ensure the Secure and HTTPOnly flags are set.
* Transmit JWTs securely. Do not send tokens in URL parameters.
* Opt for stronger signing algorithms when possible. For example, RS256 or ES256 are
stronger than HS256 with a weak secret key.
* Always transmit JWTs over a secure encrypted channel (HTTPS) to protect against
eavesdropping.
* Use the principle of least privilege when setting claims in the token. Users should only
be given access to what is required / necessary.
* Implement proper key management practices, including regular key rotation and ensure
secret keys are stored securely.
* Enforce a strict whitelist of permitted public keys and hosts for the jwk and jku headers.
* Include the aud (audience) claim to specify the intended recipient of the token. This
prevents it from being used on different websites.
* Ensure that the kid header parameter is not vulnerable to injection attacks such as path
traversal or SQL injection.
* Always set the expiration date (exp claim) for tokens to limit their validity to the shortest period of time acceptable by the business. Use short-lived tokens and refresh tokens when possible.
* If possible, enable the issuing server to revoke tokens.

---

## Resources

- [Web Security Academy - JWT Attacks](https://portswigger.net/web-security/jwt)
- [Web Security Academy - JWT Labs](https://portswigger.net/web-security/all-labs#jwt)
- [OWASP - JSON Web Token Testing Guide](https://owasp.org/www-project-web-security-testing-guide/latest/4Web_Application_Security_Testing/06-Session_Management_Testing/10-Testing_JSON_Web_Tokens)
- [What Are Refresh Tokens and How to Use Them Securely](https://auth0.com/blog/refresh-tokens-what-are-they-and-when-to-use-them/)
- [jsonwebtoken Signature Bypass Advisory](https://github.com/advisories/GHSA-qwph-4952-7xr6)
- [node-jose Improper Signature Validation](https://nvd.nist.gov/vuln/detail/CVE-2018-0114)
- [PyJWT Algorithm Confusion](https://www.vicarius.io/vsociety/posts/risky-algorithms-algorithm-confusion-in-pyjwt-cve-2022-29217)
