
#Web Security Review — OWASP Juice Shop

#1. Assessment Overview

This project documents a defensive web security review performed against an intentionally vulnerable OWASP Juice Shop application running locally in Docker.

Scope

Target: Local OWASP Juice Shop training application

URL: http://localhost:3000

Environment: Local Docker container

Testing type: Manual defensive security assessment

Scope restriction: No third-party systems were targeted.

#2. Findings Summary

ID   	Finding	Severity	Evidence
F-01 --->	Missing Content Security Policy	Medium	screenshots/finding-01-missing-csp.png
F-02 --->	SQL Injection — Authentication Bypass	High	screenshots/finding-02-sql-injection.png
F-03 --->	Publicly Accessible FTP Directory / Sensitive Files	Medium	screenshots/finding-03-ftp-exposure.png
F-04 --->	Cross-Site Scripting (XSS)	High	screenshots/finding-04-xss.png
F-05 --->	Unauthenticated Access to Admin Configuration	High	screenshots/finding-05-broken-access-control.png

#3. Detailed Findings
#F-01 — Missing Content Security Policy
Severity
Medium
Category
Security Misconfiguration
Description
The main HTML document response did not include a Content-Security-Policy response header.
Other security headers such as X-Content-Type-Options: nosniff and X-Frame-Options: SAMEORIGIN were observed.
Evidence
Impact
A CSP provides an additional browser-side security control that can reduce the impact of certain client-side injection attacks.
Remediation
Implement an appropriate Content Security Policy.
Start with a restrictive policy and gradually allow only required resources.
Avoid unnecessarily broad script and resource sources.
Test the policy before enforcing it in production.

#F-02 — SQL Injection Leading to Authentication Bypass
Severity
High
Category
Injection / Authentication
Description
A controlled SQL injection test was performed against the login functionality of the local training application.
The application accepted the crafted input and authentication was successfully bypassed without supplying a valid password.
Evidence
Impact
Successful SQL injection in an authentication function may allow unauthorized access to application functionality or accounts.
Remediation
Use parameterized queries or prepared statements.
Never concatenate user-controlled input directly into SQL queries.
Implement server-side input validation.
Use least-privilege database accounts.
Add SQL injection tests to the authentication security test suite.
Verification
Repeat the controlled test after remediation and confirm that authentication cannot be bypassed.

#F-03 — Publicly Accessible FTP Directory and Sensitive Files
Severity
Medium
Category
Security Misconfiguration / Information Exposure
Description
The /ftp directory was accessible through the web application and exposed a directory listing containing multiple internal and backup-related files.
Examples observed included:
package.json.bak
package-lock.json.bak
incident-support.kdbx
coupons_2013.md.bak
suspicious_errors.yml
Evidence
Impact
Publicly accessible internal or backup files may disclose application information and increase the application's attack surface.
Remediation
Disable directory listing in production.
Remove unnecessary backup and temporary files.
Keep sensitive files outside the web root.
Restrict access to internal resources using server-side authorization.
Prevent backup files from being deployed to public directories.

#F-04 — Cross-Site Scripting (XSS)
Severity
High
Category
Injection / Client-Side Security
Description
A controlled XSS test was performed against the local Juice Shop application. The test input was interpreted as executable browser content, demonstrating that user-controlled input could be rendered without sufficient output encoding or sanitization in the tested functionality.
Evidence
Impact
XSS can allow attacker-controlled JavaScript to execute in a victim's browser in the security context of the application.
Potential consequences can include unauthorized actions performed through the victim's session and manipulation of displayed content.
Remediation
Apply context-appropriate output encoding.
Sanitize untrusted HTML where HTML input is intentionally supported.
Implement a strong Content Security Policy.
Avoid unsafe DOM APIs such as innerHTML when not required.
Validate and sanitize user-controlled input server-side and client-side where appropriate.

#F-05 — Unauthenticated Access to Admin Configuration
Severity
High
Category
Broken Access Control / Missing Authorization
Affected Endpoint
GET /rest/admin/application-configuration
Description
While logged out of the local training application, the administrative configuration endpoint was directly requested.
The server returned HTTP 200 OK and JSON configuration data without an authenticated Authorization header or authentication cookie.
Evidence
Impact
Unauthenticated users may be able to access administrative configuration information that should be restricted to authorized administrators.
Depending on the exposed configuration, this may reveal internal application details or security-relevant settings.
Remediation
Require authentication for administrative endpoints.
Enforce server-side role-based authorization.
Do not rely only on frontend route protection.
Return 401 Unauthorized for unauthenticated requests.
Return 403 Forbidden when an authenticated user lacks the required administrative role.
Add automated authorization tests for administrative APIs.
Verification
After remediation, repeat the request while logged out and verify that the endpoint is rejected and configuration data is not disclosed.

#4. Security Controls Observed
The review also identified some existing security controls:
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
Some endpoints used specific CORS origins rather than a wildcard.
Authentication tokens were not present in requests after logout during the tested session.
These observations were not reported as vulnerabilities where the available evidence did not demonstrate exploitable behavior.

#5. Testing Limitations
This assessment was limited to the intentionally vulnerable local OWASP Juice Shop training environment.
Testing focused on manual identification and validation of common web security issues. No third-party systems were targeted, and no destructive actions or unnecessary data extraction were performed.

#6. Recommended Remediation Priority
The application should first address authentication and authorization controls, followed by injection/XSS protections and exposure of internal files. Security headers and deployment configuration should also be reviewed as defense-in-depth measures.

7. Conclusion

The review demonstrated several common web application security weaknesses in the intentionally vulnerable local training environment. The findings and screenshots provide evidence of the observed behavior and include recommended defensive remediation steps.
