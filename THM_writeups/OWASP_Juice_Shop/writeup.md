## Summary
The application suffers from multiple security flaws that allow a low-privileged user to escalate into full administrative control.
First, the login page is vulnerable to a basic SQL Injection (' OR 1=1--), which allows bypassing authentication and logging in as the admin user without a password.
After gaining admin access, the attacker can browse internal pages such as /ftp/legal.md, where sensitive files are exposed due to improper access control. This leads to accessing the Administration Panel, which should only be available to privileged users.
Inside the admin area, the Customer Feedback and Track Order features contain unsafe handling of user input, resulting in both Reflected XSS and DOM-based XSS vulnerabilities. These flaws allow arbitrary JavaScript execution inside the victim’s browser
---

## Description
The machine contains multiple security flaws that allowed full access to the application. First, the login page was vulnerable to a basic SQL Injection, enabling authentication bypass using the payload ' OR 1=1 -- and granting direct access to the admin dashboard without a valid password. Once inside, the application exposed sensitive internal files such as legal.md and backup documents through unsecured endpoints, indicating poor access control and improper file permission management. The admin panel also disclosed customer feedback and internal admin routes, further strengthening the impact of the broken authorization. Additionally, several user-input features suffered from client‑side vulnerabilities: the search bar and the /#/track-result?id= endpoint were both vulnerable to Reflected DOM‑Based XSS and HTML Injection because user input was dynamically inserted into the page without sanitization. This allowed malicious payloads like <iframe src="javascript:alert('xss')"> or <u>test</u> to execute arbitrary JavaScript in the victim's browser. Altogether, these issues—SQLi, broken access control, information disclosure, and XSS—demonstrate a severely insecure application that could lead to account compromise, data leakage, and full application takeover.

---

## Steps to Reproduce
1.Navigate to the login page of the application.

2.In the email/username field, inject the payload:
' OR 1=1 --
and submit the form to bypass authentication and gain access to the admin account.

3.Once logged in, browse to the About Us section, where an external link redirects to:
http://MACHINE_IP/ftp/legal.md.
From here, access both “Confidential Document” and “Backup File”, confirming improper access control.

4.After retrieving the files, navigate to the admin interface at:
/#/administration
where sensitive admin functions such as customer feedback are fully accessible.

5.Test the search functionality and the tracking endpoint:
#/search
/#/track-result?id=
Insert the payload:
<iframe src="javascript:alert('xss')">
to successfully trigger a DOM‑Based Reflected XSS.

6.Additionally, verify HTML injection by inserting simple payloads such as <u>test</u>, confirming that
### Request:
```
<iframe src="javascript:alert('xss')">
```
<u>test</u>
```

## Impact
The vulnerabilities identified in the application pose a high security risk due to the combination of broken authentication, improper access control, and multiple XSS flaws. By exploiting an SQL Injection (' OR 1=1 --) in the login form, an attacker can completely bypass the authentication mechanism and gain unauthorized access to the admin account. This compromises the entire application’s trust model, as the attacker can now perform privileged actions intended only for administrators.

Once inside the admin panel, the lack of proper access restrictions allows the attacker to access sensitive files such as confidential legal documents and system backup archives. This can lead to data leakage, exposure of internal information, and potential escalation into full system compromise if the leaked files contain credentials, API keys, or configuration details.

Additionally, the presence of both reflected and DOM‑based XSS vulnerabilities means that an attacker can inject arbitrary JavaScript into the victim’s browser. This could be used to steal session tokens, perform actions on behalf of authenticated users, manipulate the user interface, or conduct phishing attacks. Because the XSS is executed client‑side and affects any user interacting with the vulnerable endpoints, it significantly increases the attack surface and potential damage.

Overall, the chaining of these vulnerabilities would allow an attacker to move from initial access to full control over the application and its users, making the impact critical.

---

## Affected Endpoint(s)
- [/ftp/legal.md]
- [#/search]
- [/#/track-result?id=]

---

## Recommendation (Fix)
-Use prepared statements / parameterized queries to prevent SQL Injection.

-Implement strict server-side validation and sanitize all user inputs.

-Apply proper access control on admin routes and sensitive files.

-Store sensitive documents outside the web root and restrict direct access.

-Encode output using HTML escaping to prevent reflected & DOM XSS.

-Add a Content Security Policy (CSP) to limit the execution of inline scripts.

-Regenerate and validate session tokens after login.

-Validate id and other query parameters before rendering results.

-Disable unnecessary endpoints and implement least privilege principles.

---

## Severity
[High]

Reason: The presence of SQL Injection on the login page allows full authentication bypass, granting direct access to the admin panel. Combined with Reflected XSS and DOM-based XSS, an attacker can steal session tokens, perform account takeover, or execute arbitrary JavaScript in users’ browsers.
Additionally, exposed sensitive files and misconfigured access controls further increase the risk, enabling unauthorized access to confidential data.
These issues, when chained together, provide attackers with a clear path to full system compromise.

---

## Reporter
Name: khaled waled 

Handle(github): khaledvvaled  


