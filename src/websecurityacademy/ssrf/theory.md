# Server-Side Request Forgery (SSRF)
---

## What Is SSRF?

SSRF is a vulnerability class that occurs when an application is fetching a remote resource without first validating the user-supplied URL.

### Types of SSRF

1. **Regular / In-Band SSRF**:
   - The attacker sees the response of the malicious request in the application.

2. **Blind / Out-of-Band SSRF**:
   - The attacker does not directly see the response but observes effects or monitors interactions via external tools.

### Impact of SSRF Attacks
* Depends on the functionality in the application that is being exploited
   * Confidentiality – can be None / Partial (Low) / High
   * Integrity – can be None / Partial (Low) / High
   * Availability – can be None / Partial (Low) / High
* Can lead to sensitive information disclosure, scan of internal network, compromise of internal services, remote code execution, etc.

---

## How to Find SSRF Vulnerabilities

### Black-Box Testing

* Map the application
   * Identify any request parameters that contain hostnames, IP
addresses or full URLs
* For each request parameter, modify its value to
specify an alternative resource and observe how the
application responds
   * If a defense is in place, attempt to circumvent it using know
techniques
* For each request parameter, modify its value to a
server on the internet that you control and monitor
the server for incoming requests
   * If no incoming connections are received, monitor the time
taken for the application to respond

### White-Box Testing

* Review source code and identify all request
parameters that accept URLs
   * This could be done by combining both a black-box
and white-box testing perspective
* Determine what URL parser is being used and if it
can be bypassed. Similarly determine what
additional defenses are put in place that can be
bypassed
* Test any potential SSRF vulnerabilities

---

## How to Exploit SSRF Vulnerabilities

### Regular / In-Band SSRF

* If the application allows for user-supplied arbitrary URLs, try the following attacks:
   * Determine if a port number can be specified
   * If successful, attempt to port-scan the internal network using Burp Intruder
   * Attempt to connect to other services on the loopback address

* If the application does not allow for arbitrary user-supplied URLs, try to bypass defenses using the following techniques:
   * Use different encoding schemes
      * decimal-encoded version of 127.0.0.1 is 2130706433
      * 127.1 resolves to 127.0.0.1
      * Octal representation of 127.0.0.1 is 017700000001
   * Register a domain name that resolves to internal IP address (DNS Rebinding)
   * Use your own server that redirects to an internal IP address (HTTP Redirection)
   * Exploit inconsistencies in URL parsing

### Blind / Out-of-Band SSRF

* If the application is vulnerable to blind SSRF, try to exploit the vulnerability using the following techniques:
* Attempt to trigger an HTTP request to an external system you control and monitor the system for network interactions from the vulnerable server
* Can be done using Burp Collaborator
* If defenses are put in place, use the techniques mentioned in the previous slides to obfuscate the external malicious domain
* Once you’ve proved that the application is vulnerable to blind SSRF, you need to determine what your end goal is
* An example would be to probe for other vulnerabilities on the server itself or other backend systems

---

## How to Prevent SSRF Vulnerabilities

Defense in depth approach:
* Application Layer defenses
* Network Layer Defenses

### Application Layer Defenses
* Sanitize and validate all client-supplied input data
* Enforce the URL schema, port, and destination with a positive allow list
* Do not send raw responses to clients
* Disable HTTP redirections
**Note**: Do not mitigate SSRF vulnerabilities using deny lists or regular expressions.

### Network Layer Defenses
* Segment remote resource access functionality in separate networks to reduce the impact of SSRF
* Enforce “deny by default” firewall policies or network access control rules to block all but essential intranet traffic

---

## Resources

- [Web Security Academy - SSRF](https://portswigger.net/web-security/ssrf)
- [OWASP - SSRF](https://owasp.org/www-community/attacks/Server_Side_Request_Forgery)
- [Server-Side Request Forgery Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html)
- [SSRF Bible Cheat Sheet](https://cheatsheetseries.owasp.org/assets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet_SSRF_Bible.pdf)
- [Preventing Server-Side Request Forgery Attacks](https://seclab.nu/static/publications/sac21-prevent-ssrf.pdf)
- [A New Era of SSRF - Exploiting URL Parser in Trending Programming Languages](https://www.blackhat.com/docs/us-17/thursday/us-17-Tsai-A-New-Era-Of-SSRF-Exploiting-URL-Parser-In-Trending-Programming-Languages.pdf)
