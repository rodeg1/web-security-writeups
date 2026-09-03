# SQL Injection — UNION Database Version on MySQL

> **Environment:** PortSwigger Web Security Academy — Authorized training lab.

## Lab

PortSwigger Web Security Academy — SQL injection attack, querying the database type and version on MySQL and Microsoft.

## Objective

The objective of this lab is to exploit a SQL injection vulnerability in the product category filter and use a `UNION` attack to retrieve and display the database version string.

## Initial Analysis

The application allows users to filter products by category.

The category parameter was identified as a potential SQL injection point because user-controlled input is processed by the application's SQL query.

The lab specifies that a `UNION` attack can be used to retrieve the results of an injected query.

## Determining the Number of Columns

I first tested the number of columns returned by the original query using the `ORDER BY` technique.

The tests were performed incrementally:

```text
Gifts' ORDER BY 1-- -
Gifts' ORDER BY 2-- -
Gifts' ORDER BY 3-- -
```

The first two positions were accepted, while attempting to reference a third column resulted in an error.

This indicated that the original query returned **2 columns**.

## Identifying the Text Column

Next, I tested which column could display text using a `UNION SELECT` attack.

The following payload was successful:

```text
Corporate'union+select+'test',NULL--+-
```

The value `test` was displayed by the application.

This confirmed that:

- The query returns 2 columns.
- The first column accepts text data.
- The first column is visible in the application response.
- A `UNION SELECT` attack can be used successfully.

## Retrieving the Database Version

After identifying a text-compatible and visible column, I tested a database version function.

The following payload successfully returned the database version:

```text
corporate'union+select+version(),NULL--+-
```

The `version()` function is supported by MySQL-compatible databases and returns information about the database version.

The injected query is conceptually equivalent to:

```sql
SELECT version(), NULL
```

The first column contains the version information, while the second column is populated with `NULL` to maintain the required number of columns.

## Result

The application displayed the database version string.

The lab was successfully completed.

The successful use of the `version()` function indicates that the backend is compatible with a MySQL-style database syntax.

## Vulnerability

**SQL Injection — UNION-based SQL Injection**

The application allows user-controlled input from the product category filter to influence a SQL query without adequate parameterization.

This makes it possible to append a `UNION SELECT` statement and retrieve information from another query.

## Impact

Depending on the application's implementation and database privileges, a UNION-based SQL injection vulnerability may allow an attacker to retrieve sensitive information from the database.

Potential impact can include:

- Database technology and version disclosure
- Access to database metadata
- Retrieval of sensitive application data
- Exposure of user information
- Further database enumeration

The actual impact depends on the privileges available to the application's database account.

## Remediation

The primary mitigation is to use **parameterized queries / prepared statements** instead of concatenating user-controlled input into SQL queries.

Additional security measures include:

- Applying least-privilege database permissions
- Avoiding dynamic SQL construction
- Using secure database APIs or ORM features
- Validating input where appropriate
- Limiting unnecessary database privileges
- Monitoring suspicious database queries

## Key Takeaways

This lab demonstrated a **UNION-based SQL injection** against a MySQL-compatible database.

The main concepts were:

- Identifying a SQL injection point
- Determining the number of columns using `ORDER BY`
- Identifying a visible text-compatible column
- Using `UNION SELECT` to inject an additional query
- Using the `version()` function to retrieve database version information
- Understanding how SQL comments terminate the original query
- Using the database response to identify compatible SQL syntax
