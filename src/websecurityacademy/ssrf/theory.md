# Server-Side Request Forgery (SSRF)

## Table Of Contents
- [What Is SSRF?](#what-is-ssrf)
- [How To Find SSRF Vulnerabilities](#how-to-find-ssrf-vulnerabilities)
- [How To Exploit SSRF Vulnerabilities](#how-to-exploit-ssrf-vulnerabilities)
- [How To Prevent SSRF Vulnerabilities](#how-to-prevent-ssrf-vulnerabilities)

---

## What Is SSRF?

Server-Side Request Forgery (SSRF) is a vulnerability that occurs when an application fetches a remote resource without first validating user-supplied URLs. This can allow attackers to manipulate server-side requests to access unauthorized internal systems or third-party services.

---

### Types of SSRF

1. **Regular / In-Band SSRF**:
   - The attacker sees the response of the malicious request in the application.

2. **Blind / Out-of-Band SSRF**:
   - The attacker does not directly see the response but observes effects or monitors interactions via external tools.

---

### Impact of SSRF Attacks

- **Confidentiality**: Can lead to sensitive data exposure.
- **Integrity**: May compromise the internal systems.
- **Availability**: Can lead to denial-of-service attacks.
- **Other Impacts**:
  - Scanning internal networks.
  - Compromising internal services.
  - Remote code execution on the server.

---

## How to Find SSRF Vulnerabilities

### Black-Box Testing

1. **Map the Application**:
   - Identify parameters containing hostnames, IP addresses, or full URLs.
   
2. **Modify Parameter Values**:
   - Attempt to specify alternative resources or servers and observe responses.

3. **Monitor for Incoming Connections**:
   - Use a server you control and check for requests from the application.

4. **Test Common Defenses**:
   - Try known techniques like encoding schemes, DNS rebinding, and HTTP redirection.

---

### White-Box Testing

1. **Source Code Review**:
   - Identify all parameters that accept URLs.

2. **Analyze URL Parsers**:
   - Determine if the URL parser can be bypassed.

3. **Test Additional Defenses**:
   - Explore custom rules or bypass mechanisms in the application.

---

## How to Exploit SSRF Vulnerabilities

### Regular / In-Band SSRF

1. **Modify URL Parameters**:
   - Example Request:
     ```http
     POST /product/stock HTTP/1.0
     Content-Type: application/x-www-form-urlencoded
     Content-Length: 118

     stockApi=http://localhost/admin
     ```

2. **Bypass Defenses**:
   - Use different encoding schemes (e.g., decimal, octal).
   - Exploit DNS rebinding or HTTP redirection.
   - Leverage inconsistencies in URL parsing.

---

### Blind / Out-of-Band SSRF

1. **Trigger HTTP Requests**:
   - Example: Use a tool like Burp Collaborator to monitor for network interactions.

2. **Evade Defenses**:
   - Obfuscate malicious domains to bypass filtering mechanisms.

3. **Determine Goals**:
   - Use SSRF to find further vulnerabilities or exploit backend systems.

---

## How to Prevent SSRF Vulnerabilities

### Application Layer Defenses

1. **Validate Input**:
   - Enforce URL schema, port, and destination using a positive allow list.

2. **Disable HTTP Redirection**:
   - Prevent automatic handling of redirects.

3. **Do Not Send Raw Responses**:
   - Ensure client-supplied data is sanitized before sending responses.

---

### Network Layer Defenses

1. **Segment Remote Resource Access**:
   - Isolate network access to specific resources.

2. **Firewall Policies**:
   - Use "deny by default" policies to restrict internal traffic.

---

## Resources

- [Web Security Academy - SSRF](https://portswigger.net/web-security/ssrf)
- [OWASP - SSRF](https://owasp.org/www-community/attacks/Server_Side_Request_Forgery)
- [Server-Side Request Forgery Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html)
- [SSRF Bible Cheat Sheet](https://cheatsheetseries.owasp.org/assets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet_SSRF_Bible.pdf)
- [Preventing Server-Side Request Forgery Attacks](https://seclab.nu/static/publications/sac21-prevent-ssrf.pdf)
- [A New Era of SSRF - Exploiting URL Parser in Trending Programming Languages](https://www.blackhat.com/docs/us-17/thursday/us-17-Tsai-A-New-Era-Of-SSRF-Exploiting-URL-Parser-In-Trending-Programming-Languages.pdf)
