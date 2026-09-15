# PENETRATION TESTING REPORT

## Mediroza General Hospital
### Web Application Security Assessment

**Prepared by:** SHERIFFDEEN ABDULLAH OGANIJA  
**Cybersecurity Mentor:** Waqas Karim, CCIE  
**Organisation:** Networkwalks  
**Batch:** B082 | Week 4 Capstone Project  
**Target:** https://medirozahospital.com  
**Classification:** Confidential  

---

# 1. Executive Summary

I was assigned to conduct a black-box penetration test on the web infrastructure of Mediroza General Hospital at https://medirozahospital.com. The assessment was carried out as part of the Networkwalks cybersecurity training program with authorization for testing.

The objective of the assessment was to identify security vulnerabilities, demonstrate their impact through controlled testing, and provide recommendations that could improve the security of the application.

During the assessment, I identified several security weaknesses ranging from Medium to Critical severity. The most significant issue was a SQL injection vulnerability on the patient portal login page, which allowed authentication to be bypassed without knowing a valid password.

This provided access to three password-protected patient laboratory report PDFs. The PDF passwords were then tested using password-cracking tools. After opening the documents, metadata within one of the reports revealed information pointing to an old database backup located in a publicly accessible directory.

The database backup contained sensitive staff and shareholder information.

The assessment demonstrated that multiple weaknesses could be connected together to expose confidential information with relatively little effort.

**Overall Risk Rating: CRITICAL — Immediate remediation is recommended.**

---

# 2. Scope and Methodology

## 2.1 Scope

The assessment was limited to the following authorized target:

**Target domain:** https://medirozahospital.com

The following activities were excluded from the assessment:

- Social engineering
- Denial-of-service attacks
- Testing outside the agreed target domain

---

## 2.2 Methodology

I followed a structured black-box penetration-testing methodology consisting of four main phases:

- **Reconnaissance:** Gathering information about the target using publicly available information and web-based techniques.
- **Vulnerability Identification:** Examining application behaviour to identify weaknesses in authentication, input handling, and file access.
- **Exploitation:** Demonstrating the impact of identified vulnerabilities in a controlled manner.
- **Documentation:** Recording the findings, evidence, impact, and recommended remediation.

---

## 2.3 Tools Used

- **curl** — Used for sending HTTP requests and examining web server responses.
- **Browser Developer Tools** — Used for inspecting pages and login form behaviour.
- **Networkwalks Hash Calculator** — Used for extracting hashes from password-protected PDF files.
- **Networkwalks Password Cracker** — Used for testing and recovering PDF passwords.
- **qpdf** — Used for decrypting password-protected PDF files after password recovery.
- **exiftool** — Used for examining PDF metadata.
- **wget** — Used for downloading files from the authorized web server.
- **ChatGPT** — Used to convert raw SQL data into readable tables.

---

# 3. Findings and Proof of Exploitation

## 3.1 Summary Table

| # | Vulnerability | Location | Risk |
|---|---|---|---|
| 1 | Username enumeration on login page | `patient/login.php` | Medium |
| 2 | SQL injection login bypass | `patient/login.php` | Critical |
| 3 | Encrypted PDFs accessible after login bypass | `patient/reports/` | High |
| 4 | Weak PDF passwords crackable with a wordlist | `patient_report_*.pdf` | High |
| 5 | Sensitive metadata left in patient PDF files | `patient_report_3.pdf` | Medium |
| 6 | Forgotten backup folder with directory listing enabled | `old/` | Critical |
| 7 | Confidential staff and shareholder data in plain text | `old/mediroza_db_backup_2019.sql` | Critical |

---

## 3.2 Finding 1 — Username Enumeration

**Risk Rating: Medium**

**Location:** `patient/login.php`

### Description

Username enumeration occurs when a login page reveals whether a username exists by displaying different messages for an incorrect username and an incorrect password.

A secure login system should provide the same general error message in both situations so that valid usernames cannot easily be identified.

### Steps Taken

I opened the patient portal login page and tested the login responses using different username and password combinations.

First, I entered a username that I assumed was not registered.

```text
Username: ABDUL
Password: TEST123
RESPONSE



I then tested another username using an incorrect password.

Username: [INSERT USERNAME]
Password: [INSERT PASSWORD]

RESPONSE

[INSERT ACTUAL RESPONSE]

The different responses showed that the application could reveal information about whether a username was valid.

Evidence

[INSERT SCREENSHOT: Browser showing first login response]

[INSERT SCREENSHOT: Browser showing second login response]

3.3 Finding 2 — SQL Injection Login Bypass

Risk Rating: Critical

Location: patient/login.php

Description

SQL injection occurs when an application places user input directly into a database query without properly handling the input.

An attacker may insert SQL code into an input field and change how the database query is processed. In this assessment, the login function was tested to determine whether authentication could be bypassed.

Steps Taken

I first tested the username field by entering a single quotation mark.

Username: [INSERT TEST INPUT]
Password: [INSERT TEST PASSWORD]

RESPONSE

[INSERT ACTUAL RESPONSE]

The response was reviewed to determine whether the input affected the database query.

I then performed the authorized SQL injection test used in the training exercise.

Username: [INSERT ACTUAL TEST INPUT]
Password: [INSERT PASSWORD]

RESPONSE

[INSERT ACTUAL RESPONSE]

The result showed that the input affected the login query.

Evidence

[INSERT SCREENSHOT: Database error after SQL injection test]

[INSERT SCREENSHOT: Logged-in patient portal after successful test]

3.4 Finding 3 — Confidential PDFs Accessible After Login Bypass

Risk Rating: High

Location: patient/reports/

Description

After gaining access to the patient portal through the login vulnerability, I found three patient laboratory report PDFs available for download.

These documents contained confidential information and should only be accessible to authorized users.

Steps Taken

After successfully accessing the patient portal, I observed three downloadable PDF files.

patient_report_1.pdf
patient_report_2.pdf
patient_report_3.pdf

I downloaded the three files for the authorized password-recovery and security assessment exercise.

Evidence

[INSERT SCREENSHOT: Patient portal showing the three PDF reports]

[INSERT SCREENSHOT: Downloaded PDF files]

3.5 Finding 4 — Weak PDF Passwords Crackable with a Wordlist

Risk Rating: High

Description

The three PDF reports were password protected. However, the passwords were weak enough to be recovered using password-cracking tools and wordlists.

Using weak passwords to protect confidential documents does not provide adequate protection against password-recovery attacks.

Steps Taken

I used the Networkwalks Hash Calculator to extract the password hash from each PDF.

The extracted hashes were then submitted to the Networkwalks Password Cracker.

The password-recovery process was performed against the authorized training files.

Results
patient_report_1.pdf → [INSERT RECOVERED PASSWORD]

patient_report_2.pdf → [INSERT RECOVERED PASSWORD]

patient_report_3.pdf → [INSERT RECOVERED PASSWORD]

After recovering the passwords, I used them to open the corresponding PDF files and verify the results.

Evidence

[INSERT SCREENSHOT: Networkwalks Hash Calculator showing PDF hash]

[INSERT SCREENSHOT: Password Cracker showing recovered password]

[INSERT SCREENSHOT: Password Cracker result for another PDF]

[INSERT SCREENSHOT: Successfully opened PDF]

3.6 Finding 5 — Sensitive Metadata in Patient PDF Files

Risk Rating: Medium

Description

PDF files can contain metadata such as the author, creation date, modification information, and comments.

This information may not be visible when simply reading a document but can be extracted using tools such as exiftool.

Steps Taken

After recovering the password for the relevant PDF, I examined the document metadata using exiftool.

exiftool [INSERT PDF FILE]
Key Fields
Author: [INSERT ACTUAL RESULT]

Comments: [INSERT ACTUAL RESULT]

The metadata contained information that provided additional details about the internal environment.

Evidence

[INSERT SCREENSHOT: ExifTool output showing the metadata]

3.7 Finding 6 — Forgotten Backup Folder with Directory Listing Enabled

Risk Rating: Critical

Location: old/

Description

Directory listing is a web server configuration issue that allows visitors to view the contents of a directory when an index page is not present.

During the assessment, the old backup directory was accessible through the web server and exposed a database backup file.

Steps Taken

I accessed the identified directory through the browser.

https://medirozahospital.com/old/

The directory listing displayed the available files.

[INSERT ACTUAL BACKUP FILE NAME]

The file was downloaded for the authorized assessment.

wget [INSERT AUTHORIZED FILE URL]
Evidence

[INSERT SCREENSHOT: Browser showing the /old/ directory]

[INSERT SCREENSHOT: Database backup file visible in the directory]

3.8 Finding 7 — Confidential Staff Salaries and Shareholder Data in Plain Text

Risk Rating: Critical

Location: old/[INSERT BACKUP FILE]

Description

The database backup contained sensitive information stored in plain text.

The information included staff details and shareholder information.

Steps Taken

I opened the SQL backup file and reviewed the database contents.

The staff information was converted into a readable table containing:

Name
Job title
Department
Monthly salary

The shareholder information was also reviewed and presented using:

Shareholder name
Share percentage
Share class
Evidence

[INSERT REDACTED SCREENSHOT: Staff salary table]

[INSERT REDACTED SCREENSHOT: Shareholder table]

Sensitive personal information should be redacted before publishing screenshots to GitHub.

4. Full Attack Chain Summary

The following shows the complete sequence of the assessment and how each finding led to the next:

Step 1: Reconnaissance revealed publicly accessible directories and application paths.
Step 2: The patient login page returned different responses for different username conditions, allowing username enumeration. (Finding 1)
Step 3: A controlled SQL injection test showed that the login input could affect the database query. (Finding 2)
Step 4: The authorized SQL injection test allowed access to the patient portal. (Finding 2)
Step 5: Three confidential patient PDF reports were available through the portal. (Finding 3)
Step 6: The PDF passwords were tested and recovered using password-cracking tools. (Finding 4)
Step 7: Metadata from the unlocked PDF revealed additional internal information. (Finding 5)
Step 8: The old directory exposed a database backup through directory listing. (Finding 6)
Step 9: The database backup contained staff and shareholder information in plain text. (Finding 7)
5. Recommendations and Remediation
5.1 Fix Username Enumeration

Change the login page so that the same generic message is displayed for every failed login attempt, regardless of whether the username or password is incorrect.

For example:

Invalid credentials. Please try again.
5.2 Fix SQL Injection

Replace the existing login query with a parameterised query or prepared statement.

This separates the SQL commands from user input and prevents crafted input from changing the structure of the query.

Example using PHP PDO:

$stmt = $pdo->prepare(
    "SELECT * FROM users WHERE username = ? AND password = ?"
);

$stmt->execute([$username, $password]);
5.3 Fix PDF Access and Password Strength

Move confidential PDF files outside the publicly accessible web root so that they cannot be directly served by the web server.

Access to the files should be controlled through server-side authentication and authorization.

If passwords are used to protect sensitive documents, strong passwords should be enforced.

5.4 Strip PDF Metadata

Remove unnecessary metadata from documents before distributing them.

Internal comments and staff information should not be stored inside documents that may leave the organisation.

The following command can be used to remove metadata:

exiftool -all= patient_report_3.pdf
5.5 Fix Directory Listing and Remove the Backup

Directory listing should be disabled on publicly accessible folders.

For Apache servers, directory listing can be disabled using:

Options -Indexes

The exposed database backup should be removed from the web root immediately.

Database backups should be stored in a private and access-controlled location outside the publicly accessible part of the server.

6. Conclusion

This assessment identified a complete attack chain beginning with the login page and eventually leading to the exposure of highly sensitive internal information.

The assessment demonstrated how weaknesses in authentication, SQL input handling, document password protection, PDF metadata, directory configuration, and backup storage can combine to create a serious security risk.

The identified vulnerabilities have established remediation measures, and the Critical and High severity findings should be addressed immediately.

Submitted by: [YOUR NAME]
Cybersecurity Mentor: Waqas Karim, CCIE
Organisation: Networkwalks
Batch: B082 | Week 4 Capstone Project

This report was produced as part of a controlled educational exercise by Networkwalks. The target was authorized for security testing. These techniques must never be applied to any system without explicit written permission from the owner.
