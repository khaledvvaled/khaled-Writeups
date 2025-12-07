## Summary:
The login endpoint at https://******.com/login
 does not enforce any rate‑limiting or lockout mechanism. After discovering the page through directory enumeration, I tested the login function using Burp Suite Intruder and confirmed that the application allows unlimited authentication attempts without any throttling, CAPTCHA, or temporary blocking. This behavior enables attackers to perform automated brute‑force attacks against user accounts, potentially leading to unauthorized access and exposure of sensitive affiliate data.

## Steps To Reproduce:
1. **Open the login page:**  
   https://******.com/login

2. **Intercept a normal login request** using Burp Suite (POST request containing `username` and `password`).

3. **Send the captured request** to **Burp Intruder**.

4. **Configure Intruder:**
   - Set the payload position on the `password` field (or both fields).
   - Load a password list such as `rockyou.txt`.

5. **Start the Intruder attack.**

6. **Observe the behavior:**
   - The server accepts **unlimited login attempts**.
   - No rate limiting, CAPTCHA, IP throttling, or account lockout is triggered.
   - The application continues responding normally after hundreds or thousands of requests.

7. This demonstrates the possibility of performing **brute-force** or **credential stuffing** attacks without restrictions.

## Supporting Material/References:
F5063619

## Impact

The lack of rate limiting on the login endpoint allows an attacker to perform automated brute-force and credential-stuffing attacks without restriction, potentially leading to unauthorized access to user accounts.