# SQL Injection — Authentication Bypass

> **Environment:** PortSwigger Web Security Academy — Authorized training lab.

## Lab

PortSwigger Web Security Academy — SQL injection vulnerability in the login function.

## Objective

The objective of this lab is to exploit a SQL injection vulnerability in the login function and authenticate as the `administrator` user.

## Initial Analysis

The application contains a login form that accepts a username and password.

The username field was identified as a potential injection point because user-controlled input is processed by the application's authentication query.

The lab specifies that the target account is:

`administrator`

## Exploitation

I tested a SQL injection payload in the username parameter:

```text
administrator'-- -
```

The single quote closes the original username string.

The `-- -` sequence starts a SQL comment, causing the remaining portion of the query to be ignored.

Conceptually, the query becomes similar to:

```sql
SELECT * FROM users
WHERE username = 'administrator'-- -'
AND password = '...'
```

This effectively removes the password verification from the query.

## Result

The application authenticated me as the `administrator` user without requiring the correct password.

The lab was successfully completed.

## Vulnerability

**SQL Injection — Authentication Bypass**

The application incorporates user-controlled input into the authentication SQL query without using adequate parameterization.

## Impact

An SQL injection vulnerability in an authentication function can potentially allow an attacker to bypass authentication and access accounts without knowing their passwords.

The impact depends on the application's implementation and database privileges.

## Remediation

The primary mitigation is to use **parameterized queries / prepared statements** for database operations.

Additional measures include:

- Never concatenate user input directly into SQL queries
- Use parameterized database APIs
- Apply least-privilege database permissions
- Implement appropriate authentication controls
- Monitor and investigate suspicious authentication activity

## Key Takeaways

This lab demonstrated how SQL injection can be used to bypass authentication.

The main concepts were:

- Identifying an injection point in a login function
- Closing the original SQL string
- Using SQL comments to remove the password verification
- Understanding how SQL injection can affect authentication logic
