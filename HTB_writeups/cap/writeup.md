## Summary
The machine contains a simple IDOR vulnerability in the capture feature. By changing the ID number in the URL, we can download other users’ PCAP files. One of these PCAPs reveals valid FTP credentials. Using these credentials gives access to the FTP server and the user flag. After logging in as the user, system enumeration shows a Python binary with a special capability that allows running it as root. Abusing this capability lets us execute commands with root privileges and finally obtain the root flag.

---

## Description
The main vulnerability in the CAP machine is an Insecure Direct Object Reference (IDOR) in the /capture functionality. This endpoint is supposed to capture and display network traffic related only to the logged‑in user. However, the application does not properly validate whether the requesting user is allowed to access a specific capture file. As a result, the attacker can freely change the numeric ID in the URL (e.g., /data/0, /data/1, /data/2) and download traffic captures belonging to other users.

This vulnerability is dangerous because it exposes sensitive information contained inside the PCAP files. In this case, one of the captured files includes cleartext FTP credentials, which should never be accessible to unauthorized users. By retrieving these credentials, the attacker can log into the FTP service and gain full access to the victim’s account and files.

From there, the compromise escalates. After logging in as the FTP user and moving to SSH access, local enumeration reveals a Python binary with a misconfigured Linux capability (cap_setuid). This allows the attacker to execute Python with root privileges. Abusing this misconfiguration leads to full privilege escalation and root control over the system.

---

## Steps to Reproduce
1.Scanned the target using Nmap, which revealed port 80 running an HTTP service.

2.Accessed the web application and identified a feature that generates downloadable network captures at the endpoint:
http://10.10.10.245/data/<id>.

3.Tested the parameter manually and discovered that changing the value to 0 or 2 (e.g., /data/0, /data/2) allowed downloading PCAP files belonging to other users, confirming an IDOR vulnerability.

4.Downloaded the exposed PCAP file and analyzed it locally using Wireshark.

5.Extracted clear‑text FTP credentials observed inside the captured traffic.

6.Used the recovered credentials to authenticate to the target’s FTP service and accessed the user's files.

7.Reused the same credentials to establish an SSH connection, successfully obtaining a shell as the user.

8.Enumerated system capabilities using:
getcap -r / 2>/dev/null
and discovered that /usr/bin/python3.8 had the cap_setuid capability.

9.Leveraged this capability to escalate privileges by executing a Python one‑liner to set UID to 0, granting a full root shell:
python3.8 -c 'import os; os.setuid(0); os.system("/bin/bash")'

10.Retrieved the root flag from the root directory.

---

### Request:
```
getcap -r / 2>/dev/null
```
```
python3.8 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```

### Response:
```
/usr/bin/python3.8 cap_setuid+ep
```

---

## Impact
The IDOR vulnerability allowed attackers to download PCAP files belonging to other users simply by changing the ID value. These PCAP files exposed clear‑text FTP credentials, which enabled unauthorized access to internal services. After logging in via FTP and SSH, the attacker abused a misconfigured Linux capability on Python (cap_setuid) to escalate to root instantly. This resulted in full system compromise, allowing complete control over the server.

---

## Affected Endpoint(s)
- [/data/id]
- [/capture]
 


---


## Recommendation (Fix)
-Only allow users to access their own PCAP files.

-Use random IDs (UUIDs) instead of numbers.

-Don’t store passwords in network captures.

-Don’t run system commands with user‑controlled input.

-Remove unnecessary Linux capabilities from programs.

---

## Severity
[High]

Reason: The IDOR allows attackers to access sensitive PCAP files, which may contain usernames, passwords, and network information. Combined with the ability to execute system commands, it can lead to full system compromise.
---

## Reporter
Name: khaled waled

Handle(github): khaledvvaled  


