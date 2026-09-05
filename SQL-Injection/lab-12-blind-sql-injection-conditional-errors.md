# Lab 12 — Blind SQL Injection with Conditional Errors

## Overview

**Platform:** PortSwigger Web Security Academy  
**Category:** SQL Injection  
**Type:** Blind SQL Injection — Conditional Errors  
**Difficulty:** Practitioner

### Objective

Exploit a blind SQL injection vulnerability in the `TrackingId` cookie to determine the password of the `administrator` user and use it to authenticate to the application.

---

## Vulnerability

The application uses the `TrackingId` cookie for analytics and incorporates its value into a SQL query.

Unlike a traditional SQL injection, the application does not directly return the results of the SQL query.

Additionally, the application does not behave differently depending on whether the query returns rows.

However, when the injected SQL query generates a database error, the application returns an HTTP 500 response.

This allows the HTTP status code to be used as a side channel.

The application therefore provides a boolean oracle based on SQL errors:

```text
Condition TRUE
      ↓
SQL error
      ↓
HTTP 500

Condition FALSE
      ↓
No SQL error
      ↓
Normal response
