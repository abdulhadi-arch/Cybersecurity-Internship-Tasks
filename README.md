# 🛡️ Cybersecurity Internship: Week 1 Security Assessment

**Intern:** [Aapka Naam Yahan Likhein]
**Target Application:** Vulnerable Node.js User Management System
**Tech Stack:** Node.js, Express, MongoDB
**Date:** May 2026

## 📝 1. Assessment Overview
During Week 1, I successfully set up the mock vulnerable web application locally on my Kali Linux environment. My objective was to evaluate the application's security posture by combining automated scanning (OWASP ZAP) with manual penetration testing techniques (Browser Developer Tools). 

Since the application uses MongoDB instead of a traditional relational database, I had to adapt my testing methodology—shifting from standard SQL Injection techniques to NoSQL injection vectors.

---

## 🔎 2. Vulnerability Findings & Proof of Concept

### Finding 1: Broken Access Control (Admin Panel Bypass)
* **Risk Level:** High
* **Observation:** While exploring the application routes, I discovered that the `/admin` endpoint lacks proper session validation. 
* **Exploitation:** By simply navigating to `http://localhost:3000/admin` as an unauthenticated user, I was granted full access to the administrative interface, including the ability to view the User List and add new users.
* **Evidence:**
  ![Broken Access Control - Admin Panel](broken-access-control.png)

### Finding 2: Authentication Bypass via NoSQL Injection
* **Risk Level:** Critical
* **Observation:** The assignment required testing for weak password storage and SQL injection (e.g., `' OR '1'='1`). However, upon identifying the backend as MongoDB, I realized standard SQL payloads wouldn't work.
* **Exploitation:** I inspected the login mechanism and crafted a NoSQL injection payload using the `$gt` (greater than) operator. By injecting `username[$gt]=&password[$gt]=` via the browser console's Fetch API, I manipulated the database query to evaluate as true. This successfully bypassed the login screen and exposed the admin's sensitive information (Credit Card, Location, etc.) without requiring a password.
* **Evidence:**
  ![NoSQL Authentication Bypass](no-sql-login-bypass.png) 

### Finding 3: Reflected Text Injection (Content Spoofing)
* **Risk Level:** Medium
* **Observation:** I tested the application for Cross-Site Scripting (XSS) using standard payloads like `<script>alert('XSS')</script>`. The templating engine successfully escaped the HTML tags. However, it failed to sanitize the input structure.
* **Exploitation:** The `/order` route blindly reflects the `name` query parameter into the DOM. I was able to inject misleading text/links (e.g., `?name=Click_Here_To_Reset_Password_[http://malicious-site.com]`), which could easily be used in a social engineering or phishing attack against users.
* **Evidence:**
  ![Content Spoofing](content-spoofing.png) 

### Finding 4: Security Misconfigurations & Weak Headers (OWASP ZAP)
* **Risk Level:** Medium to Low
* **Observation:** To ensure I didn't miss any infrastructural flaws, I ran an automated scan using OWASP ZAP against the local server.
* **Automated Findings:** 
  1. **Vulnerable JS Library:** The app uses an outdated jQuery version (v2.1.1) which is vulnerable to known CVEs.
  2. **Missing CSP:** No Content Security Policy is defined, increasing the risk of data injection.
  3. **Missing Clickjacking Protection:** The `X-Frame-Options` header is absent.
  4. **Cookie Misconfiguration:** Session cookies lack the `SameSite` attribute, opening the door for CSRF attacks.
* **Evidence:**
  ![OWASP ZAP Automated Scan Alerts](zap-report.png) 

---

## 🛠️ 3. Areas of Improvement & Remediation Plan

Based on my manual and automated testing, here is what needs to be fixed in the upcoming weeks:

1. **Secure the Admin Route:** Implement strict session validation and JWT (JSON Web Tokens) to ensure only authenticated administrators can access the `/admin` dashboard.
2. **Sanitize Database Inputs:** Use libraries like `validator` to ensure inputs are strictly strings and strip out NoSQL operators (like `$gt`, `$ne`) before passing them to MongoDB.
3. **Enforce Strong Password Policies:** Implement `bcrypt` to securely hash passwords. The current system appears to store or handle passwords insecurely.
4. **Implement Security Headers:** Integrate `helmet.js` in the Express app to automatically set crucial HTTP headers, including `Content-Security-Policy`, `X-Frame-Options: DENY`, and `X-Content-Type-Options: nosniff`.
5. **Secure Session Cookies:** Update the cookie configuration to include the `SameSite=Lax` or `Strict` attribute.
