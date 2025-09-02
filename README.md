📑 DVWA XSS Testing Report
1. Executive Summary

This report documents the results of Cross-Site Scripting (XSS) testing on the Damn Vulnerable Web Application (DVWA). Multiple XSS types—DOM, Reflected, and Stored—were tested across security levels: Low, Medium, and High.

Findings:

Low security: fully vulnerable to all XSS types.

Medium security: partial mitigations; still bypassable using creative payloads.

High security: properly mitigated via input validation, output encoding, and whitelisting.

Impact: An attacker could steal session cookies, inject malicious scripts, deface content, or perform actions on behalf of users.

2. Testing Methodology

DVWA Setup

Login: admin / password

Setup/Reset Database → DVWA Security Level: Low → Medium → High

PHPIDS: Disabled for initial tests

Tools

Web Browser: Chrome

Proxy: Burp Suite (Intercept on) to inspect requests and responses

Approach

Input payloads via forms, URL parameters, and comments.

Observe execution and document results.

Incrementally increase security level to evaluate mitigations.

3. Findings
A. DOM-Based XSS
Security Level	How it Works	Payload Example	Result	Vulnerability / Fix
Low	Input from URL parameter is directly written into the DOM using document.write() or .innerHTML	http://<IP>/dvwa/vulnerabilities/xss_d/?default=<script>alert('DOM-Low')</script>	JavaScript executed → alert box appears	No filtering → fully exploitable
Medium	Input is reflected into DOM, e.g., inside a dropdown	http://<IP>/dvwa/vulnerabilities/xss_d/?default=<script>alert('DOM-Medium')</script>	Payload executes → alert pops	Dangerous HTML allowed → still vulnerable
High	Input validated against a whitelist (en, fr, es)	http://<IP>/dvwa/vulnerabilities/xss_d/?default=<script>alert(1)</script>	Ignored → no alert	Proper whitelisting → secure

Impact: Attacker can execute arbitrary scripts in user browsers, leading to cookie theft or session hijacking.

Fix Recommendation: Avoid innerHTML / document.write; use textContent or setAttribute. Implement strict whitelisting and CSP.

B. Reflected XSS
Security Level	How it Works	Payload Example	Result	Vulnerability / Fix
Low	User input reflected in response without sanitization	<script>alert('Reflected-Low')</script>	Immediate execution → alert pops	Fully exploitable
Medium	Basic sanitization applied (e.g., <script> stripped)	<img src=x onerror=alert('Reflected-Medium')>	Script executes via attribute → alert pops	Partial filtering → bypassable
High	Input encoded before reflection (< → &lt;)	<script>alert(1)</script>	Appears as text → no execution	Proper output encoding → mitigated

Impact: Reflected XSS can steal sensitive data or perform malicious actions on behalf of users.

Fix Recommendation: Apply context-aware output encoding and avoid reflecting untrusted input.

C. Stored XSS
Security Level	How it Works	Payload Example	Result	Vulnerability / Fix
Low	User-submitted content stored in DB and displayed without filtering	<script>alert('Stored-Low')</script>	Every page load → alert pops	Persistent attack → affects all visitors
Medium	<script> tags blocked; attributes and HTML allowed	<img src=x onerror=alert('Stored-Medium')>	Executes on page reload	Partial filtering → still bypassable
High	Strong sanitization + encoding before storage/display	<script>alert('Stored')</script>	Displayed as harmless text → no execution	Proper sanitization and output encoding → secure

Impact: Persistent XSS could compromise multiple users, steal cookies, or modify site content.

Fix Recommendation: Validate input, encode output, and use frameworks with auto-escaping.

4. Summary Table
XSS Type	Low	Medium	High
DOM	Direct execution via document.write	Executes inside dropdown → alert pops	Whitelisted → no execution
Reflected	Full <script> works	<script> blocked, bypass via <img onerror>	Encoded → no execution
Stored	Persistent <script> execution	<script> blocked, bypass via <img>	Encoded → no execution
5. Key Learnings

Low Security: Fully vulnerable to all payloads → critical risk.

Medium Security: Filters basic <script> tags, but creative payloads bypass → moderate risk.

High Security: Proper mitigations (validation, encoding, whitelisting) → low risk.

Overall: XSS vulnerabilities can lead to session theft, defacement, and malicious script execution if unpatched.

6. References / Resources

DVWA Official: http://www.dvwa.co.uk

OWASP XSS: https://owasp.org/www-community/attacks/xss/
