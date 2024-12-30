# **Postmortem: SQL Injection in Beauty Parlour Management System (PHP, SQLite) v1.0**

## **1. Incident Summary**

- **System Affected**: **Beauty Parlour Management System PHP SQLite Project** (v1.0)  
- **File Location**: `/admin/add-customer-services.php`  
- **Vulnerability Type**: SQL Injection  
- **Authentication Required**: None  
- **Discovered By**: lings3346  

An SQL injection vulnerability has been found where an attacker can supply malicious payloads in the parameter labeled “Array-like #1* ((custom) POST).” This vulnerability enables unauthorized operations on the back-end database without proper validation or sanitization.

---

<img width="1232" alt="image" src="https://github.com/user-attachments/assets/42a45a6c-d621-439e-b9a2-9f0c6ca4f6fb" />


## **2. Timeline**

1. **Initial Discovery**  
   - Security testing uncovered abnormal query processing in `/admin/add-customer-services.php`.

2. **Proof-of-Concept (PoC) Confirmation**  
   - Payloads were tested, demonstrating time-based blind SQL injection.  
   - Example payload:  
     ```makefile
     sids[]=1' AND (SELECT 2169 FROM (SELECT(SLEEP(5)))yHIH) AND 'rcDZ'='rcDZ&sids[]=2&submit=
     ```

3. **Validation with sqlmap**  
   - `sqlmap` confirmed the injection flaw:
     ```bash
     sqlmap -r book.txt --batch --level=5 --risk=3 \
            --random-agent --tamper=space2comment --dbms=mysql
     ```
   - Screenshot evidence captured.
   - <img width="999" alt="image" src="https://github.com/user-attachments/assets/cd4f1f15-ceaf-47ea-b1e7-ac78f40599dd" />


4. **Vulnerability Report Drafted**  
   - Issue documented, including root cause and recommended steps.

---

## **3. Root Cause Analysis**

The primary root cause is **insufficient sanitization** of user inputs in the “Array-like #1* ((custom) POST)” parameter. The system constructs SQL statements with these parameters directly, allowing attackers to embed arbitrary code into the query.

---

## **4. Impact Analysis**

- **Database Compromise**  
  Attackers may escalate privileges, read sensitive data, or make unauthorized modifications.

- **Data Leakage**  
  Confidential information (e.g., customer details, service logs) could be exposed.

- **Service Interruption**  
  Malicious queries (like time-based “SLEEP” injections) may degrade system performance or trigger crashes.

- **System Control**  
  In some scenarios, attackers pivot from database to broader system-level access if combined with other exploits.

---

## **5. Remediation Measures**

1. **Parameterized Queries / Prepared Statements**  
   - **Action**: Implement prepared statements to separate SQL logic from user inputs.  
   - **Outcome**: Prevents malicious data from being interpreted as executable code.

2. **Input Validation & Sanitization**  
   - **Action**: Enforce strict checks on parameter formats (e.g., regex, escaping).  
   - **Outcome**: Minimizes the chance of injecting special characters or control sequences.

3. **Least Privilege Principle**  
   - **Action**: Limit DB account permissions for application usage. Avoid high-privilege accounts in everyday operations.  
   - **Outcome**: Reduces damage potential if SQL injection does occur.

4. **Periodic Security Audits**  
   - **Action**: Schedule routine code reviews and penetration tests.  
   - **Outcome**: Early detection of vulnerabilities and continuous risk mitigation.

---

## **6. Lessons Learned**

- **Importance of Secure Coding**  
  Relying solely on client-side validation or basic checks is insufficient. Proper prepared statements and rigorous server-side validation are essential.

- **Audit Open-Source Components**  
  Regular scans for known vulnerabilities in third-party libraries, frameworks, and code modules help maintain security hygiene.

- **Comprehensive Testing**  
  Tools like `sqlmap` are invaluable in discovering injection points missed by manual reviews. Incorporating automated scanners ensures broad coverage.

- **No Trust Boundary**  
  Even “internal” parameters or array-like POST fields can be used maliciously if left unvalidated. Assume all external input is potentially dangerous.

---

## **7. References**

- **Vendor Homepage**: [Beauty Parlour Management System PHP SQLite Project](https://1000projects.org/beauty-parlour-management-system-php-sqlite-project.html)  
- **Download Link**: [Beauty-Parlour-Management-System.7z](https://1000projects.org/wp-content/uploads/2023/01/Beauty-Parlour-Management-System.7z)  
- **SQL Injection (OWASP)**: <https://owasp.org/www-community/attacks/SQL_Injection>

---

## **8. Conclusion**

The SQL injection vulnerability in `/admin/add-customer-services.php` exposes the Beauty Parlour Management System to significant risks, including data leakage, system compromise, and potential service downtime. Implementing prepared statements, applying rigorous input validation, and adhering to least-privilege principles will address this issue and enhance overall application security. A schedule for regular security assessments is strongly advised to prevent similar vulnerabilities in the future.

---

**Disclaimer**: This postmortem report is provided for responsible disclosure. Any misuse of the information herein for malicious purposes may be subject to legal consequences.
