# Admin Panel Disclosed in JavaScript

## Lab

**Platform:** PortSwigger Web Security Academy  
**Difficulty:** Apprentice  
**Category:** Access Control  
**Vulnerability:** Broken Access Control  
**Status:** Solved

---

## Objective

The application contains an unprotected administrative panel.

The location of the panel is unpredictable, but the application discloses its location somewhere in the client-side code.

The objective of the lab is to access the administrative panel and use it to delete the user `carlos`.

---

## Reconnaissance

During the initial reconnaissance, I inspected the application's client-side JavaScript.

The following code was identified:

```javascript
var isAdmin = false;

if (isAdmin) {
   var topLinksTag = document.getElementsByClassName("top-links")[0];
   var adminPanelTag = document.createElement('a');
   adminPanelTag.setAttribute('href', '/admin-aai0i1');
   adminPanelTag.innerText = 'Admin panel';
   topLinksTag.append(adminPanelTag);
   var pTag = document.createElement('p');
   pTag.innerText = '|';
   topLinksTag.appendChild(pTag);
}
```

---

## Analysis

The code contains two interesting elements.

First:

```javascript
var isAdmin = false;
```

This indicates that the administrative link is not displayed to the current user.

However, the JavaScript also reveals the actual administrative endpoint:

```javascript
adminPanelTag.setAttribute('href', '/admin-aai0i1');
```

Therefore, the administrative panel is located at:

```text
/admin-aai0i1
```

The fact that the link is hidden from the user does not prevent direct access to the endpoint.

---

## Exploitation

Instead of relying on the application's navigation, I accessed the disclosed endpoint directly:

```http
GET /admin-aai0i1
```

The application returned the administrative panel without requiring the appropriate administrative privileges.

This demonstrated that the endpoint was not properly protected by server-side authorization.

The administrative panel provided functionality to manage users.

I used the available functionality to delete:

```text
carlos
```

The lab was then marked as solved.

---

## Attack Flow

```text
Application
     │
     ▼
Client-side JavaScript
     │
     ▼
isAdmin = false
     │
     ├── Admin link hidden
     │
     ▼
JavaScript reveals endpoint
     │
     ▼
/admin-aai0i1
     │
     ▼
Direct access
     │
     ▼
No effective authorization check
     │
     ▼
Administrative panel
     │
     ▼
Delete carlos
     │
     ▼
Lab solved
```

---

## Vulnerability

The underlying vulnerability is **Broken Access Control**.

The application attempts to control access to the administrative functionality on the client side by conditionally displaying the administrator link:

```javascript
if (isAdmin) {
    // Display admin panel link
}
```

However, hiding a link does not restrict access to the underlying endpoint.

An attacker can inspect the client-side code, discover the endpoint, and request it directly.

Authorization must therefore be enforced on the server side.

---

## Impact

An unauthorized user could potentially access administrative functionality by directly requesting the exposed endpoint.

Depending on the functionality available through the administrative panel, this could potentially allow an attacker to:

- Delete users
- Modify user accounts
- Access administrative information
- Change application settings
- Perform other privileged operations

The exact impact depends on the functionality exposed by the affected application.

---

## Remediation

Administrative functionality should be protected through **server-side authorization controls**.

The application should verify the privileges of every request before allowing access to administrative functionality.

For example:

```text
Request
   │
   ▼
Authentication
   │
   ▼
Server-side authorization
   │
   ├── Administrator → Allow
   │
   └── Regular user → Deny
```

Client-side checks such as:

```javascript
if (isAdmin) {
    // show admin link
}
```

may be useful for the user interface, but they must never be considered a security boundary.

Sensitive endpoints should return an appropriate authorization error when accessed by users without the required privileges.

---

## Key Takeaways

This lab demonstrated several important concepts:

- Client-side JavaScript can disclose sensitive application paths.
- Hidden functionality is not necessarily protected functionality.
- Administrative endpoints should be tested directly.
- Client-side authorization checks are not a substitute for server-side authorization.
- Reconnaissance should include inspection of JavaScript and other client-side resources.
- Access control must be enforced for the actual server-side resource or operation.

---

## Methodology

```text
Reconnaissance
      ↓
Inspect client-side resources
      ↓
Identify interesting JavaScript
      ↓
Find hidden administrative endpoint
      ↓
Request endpoint directly
      ↓
Test authorization
      ↓
Access administrative functionality
      ↓
Validate impact
      ↓
Document vulnerability
```

---

## Tools Used

- Burp Suite Community Edition
- Browser
- JavaScript source inspection
- PortSwigger Web Security Academy

---

## Conclusion

The lab demonstrated a common access control mistake: relying on client-side logic to hide privileged functionality.

Although the administrative link was only created when:

```javascript
isAdmin = true
```

the actual endpoint remained directly accessible.

The vulnerability was therefore not the disclosure of the path itself, but the **lack of effective server-side authorization protecting the administrative endpoint**.
