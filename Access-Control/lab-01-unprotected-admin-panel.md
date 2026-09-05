# Unprotected Admin Panel

## Lab

**Platform:** PortSwigger Web Security Academy  
**Difficulty:** Apprentice  
**Category:** Access Control  
**Vulnerability:** Broken Access Control  
**Status:** Solved

---

## Objective

The application contains an administrative panel that is not properly protected by an authorization mechanism.

The objective of the lab is to access the administrator panel and delete the user `carlos`.

---

## Reconnaissance

During the initial reconnaissance, I checked the application's `robots.txt` file:

```text
/robots.txt
```

The file revealed an interesting path:

```text
/administrator-panel
```

It is important to note that `robots.txt` is **not a security mechanism**. It only provides instructions to web crawlers about which resources should or should not be crawled.

Therefore, discovering a sensitive path in `robots.txt` does not itself represent the vulnerability.

---

## Testing the Administrative Endpoint

I accessed the discovered endpoint directly:

```http
GET /administrator-panel
```

The application returned the administrative panel without requiring authentication or verifying administrative privileges.

This demonstrated that the administrative functionality was exposed to unauthorized users.

---

## Exploitation

The administrative panel contained functionality for managing users.

The target user specified by the lab was:

```text
carlos
```

Using the administrative functionality, I deleted the `carlos` account.

The lab was then marked as solved.

---

## Vulnerability

The underlying vulnerability is **Broken Access Control caused by missing authorization checks**.

The application exposes a privileged administrative endpoint without verifying whether the requesting user has the required permissions.

The attack flow can be summarized as:

```text
Attacker
   │
   ▼
/robots.txt
   │
   ▼
/administrator-panel
   │
   ▼
No authorization check
   │
   ▼
Administrative functionality
   │
   ▼
Delete user
```

---

## Impact

An unauthenticated attacker could potentially access administrative functionality that should only be available to authorized administrators.

Depending on the functionality exposed by the panel, this type of vulnerability could potentially allow:

- User management
- Account deletion
- Modification of application data
- Administrative configuration changes
- Other privileged operations

The exact impact depends on the functionality exposed by the affected application.

---

## Remediation

Administrative endpoints should enforce authorization checks on the server side.

The application should verify that the authenticated user has the appropriate administrative privileges before allowing access to sensitive functionality.

For example:

```text
Request
   │
   ▼
Authentication
   │
   ▼
Authorization check
   │
   ├── Authorized → Allow request
   │
   └── Unauthorized → Deny request
```

Sensitive endpoints should never rely on:

- `robots.txt`
- Hidden links
- Obscure URLs
- Client-side restrictions

Authorization must be enforced server-side for every privileged operation.

---

## Key Takeaways

This lab demonstrated a basic but important access control failure.

The main lessons were:

- Sensitive endpoints should not be considered secure simply because they are difficult to discover.
- `robots.txt` can reveal interesting application paths but does not provide protection.
- Administrative functionality must enforce server-side authorization.
- Directly requesting privileged endpoints is an important part of web application testing.
- Access control should be tested independently from authentication.

---

## Tools Used

- Burp Suite Community Edition
- Browser
- PortSwigger Web Security Academy

---

## Methodology

```text
Reconnaissance
      ↓
Identify interesting paths
      ↓
Direct endpoint access
      ↓
Check authentication
      ↓
Check authorization
      ↓
Identify privileged functionality
      ↓
Validate impact
      ↓
Document vulnerability
```
