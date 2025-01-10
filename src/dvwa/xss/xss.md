# XSS

## Security Level - Low

### DOM-Based XSS
Choose English language and intercept the request in Burp.
![dom-based-xss-low-security](images/dom-based-xss-low-security.png)

Notice that the choice is being passed to the `document.write()` sink

![value-passed-to-sink-low-security](images/value-passed-to-sink-low-security.png)

#### Exploit
Exploiting this requires escaping `<option value='English'>English</option>`. 

Submit `domain/DVWA/vulnerabilities/xss_d/?default='><script>alert(1)</script>`

#### Result

![reflected-dom-based-xss-low-security](images/reflected-dom-based-xss-low-security.png)

### Reflected XSS

![reflected-xss-low-security-level](images/reflected-xss-low-security-level.png)

In order to exploit this, we add a random value, submit it and then check the page source:

![page-source-reflected-xss-low-level-security-level](images/page-source-reflected-xss-low-level-security-level.png)

We notice that the value is reflected between `<pre></pre>` tags.

We can now submit `<script>alert(1)</script>`.

#### Result

![reflected-xss-low-security-level-result](images/reflected-xss-low-security-level-result.png)

### Stored XSS

![stored-xss-low-security-level](images/stored-xss-low-security-level.png)

Input a random value to the `Name` and `Message` fields and check how the values are stored.

![no-protection-measures-stored-xss](images/no-protection-measures-stored-xss.png)

For this stored XSS to work, we can try to add `<img src=x onerror=alert(1)>` in the message box.

#### Result
![stored-xss-low-level-security-result](images/stored-xss-low-level-security-result.png)