## Summary:
A reflected Cross‑Site Scripting (XSS) vulnerability was identified on https://********.
The chatbot accepts user-supplied input and reflects it back in the response without proper HTML sanitization or encoding. This allows an attacker to inject malicious JavaScript that executes in the victim’s browser.

By sending a specially crafted payload, the script executed successfully inside the chatbot interface, confirming the presence of XSS.
This issue can be exploited to steal session data, perform actions on behalf of users, redirect victims to malicious websites, or deliver phishing attacks.

## Steps To Reproduce:
1.I accessed the chatbot interface at https://**********.

2.Entered simple HTML (<u>test</u>) and observed that it was rendered without any sanitization.

3.Tested a basic XSS payload:
<img src=x onerror=alert(1)>
The alert executed inside the chatbot, confirming reflected XSS.

4.To verify whether the payload is only reflected on the client or also executed on the backend/staff dashboard, I sent a silent callback payload:
<img src="https://s1ul9x1js94fk70gwmlnjkvuilocc30s.oastify.com/test">
This payload does not show anything visually but triggers an external request if executed on another system.

5.After sending the payload, Burp Collaborator received DNS and HTTP interactions:

DNS lookup from infrastructure IPs (***.***.12.151, ***.***.228.155)
HTTP request from ***.***.63.59

6.These interactions confirm that the payload was executed by a system other than my browser—most likely the backend service (**********.com) or the staff/admin dashboard that processes chatbot messages. This means the XSS is not only reflected but is also executed server-side or when staff view the message.

## Supporting Material/References:

F5088087
F5088093
F5088095


## Impact

-Execute arbitrary JavaScript in users’ browsers.

-Steal session tokens or sensitive data.

-Perform phishing or redirect victims to malicious sites.

-Execute automatically inside the staff dashboard, risking staff account takeover.

-Exposure of internal data handled by staff.