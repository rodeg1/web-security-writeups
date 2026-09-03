# SQL Injection — UNION Database Version

> **Environment:** PortSwigger Web Security Academy — Authorized training lab.

## Lab

PortSwigger Web Security Academy — SQL injection vulnerability in the product category filter.

## Objective

The objective of this lab is to use a SQL injection `UNION` attack to retrieve and display the database version string.

The application uses an Oracle database, which requires each `SELECT` statement to specify a table in the `FROM` clause.

## Initial Analysis

The application allows users to filter products by category.

The category parameter was identified as a potential SQL injection point.

The lab indicated that a `UNION` attack could be used to retrieve data from the database.

Because the application uses Oracle, the built-in `dual` table can be used when a `SELECT` statement does not otherwise require a table.

## Determining the Number of Columns

I first tested the number of columns returned by the original query using the `ORDER BY` technique.

The following input was accepted:

```text
Gifts' order by 2-- -
```

However, using a third column resulted in an Internal Server Error:

```text
Gifts' order by 3-- -
```

This indicated that the original query returned **2 columns**.

## Identifying the Text Column

Next, I tested which column could display text using a `UNION SELECT` attack.

The following payload was successful:

```text
Gifts' UNION SELECT NULL,'test' FROM dual-- -
```

The value `test` was displayed by the application.

This confirmed that:

- The query returns 2 columns.
- The second column accepts text data.
- The `dual` table can be used for the Oracle `UNION SELECT` statement.

## Retrieving the Database Version

After identifying a suitable text column, I replaced the test value with the Oracle `banner` value from the `v$version` view.

The final payload was:

```text
Gifts'union+select+NULL,banner+FROM+v$version--+-
```

Conceptually, the injected query uses the second column to retrieve the database version information:

```sql
SELECT NULL, banner FROM v$version
```

The first column is populated with `NULL`, while the second column contains the database version information.

## Result

The application displayed the Oracle database version string.

The lab was successfully completed.

## Vulnerability

**SQL Injection — UNION-based SQL Injection**

The application allows user-controlled input from the category filter to influence a SQL query without adequate parameterization.

This makes it possible to append a `UNION SELECT` statement and retrieve data from another query.

## Impact

Depending on the application's database permissions and implementation, a UNION-based SQL injection vulnerability may allow an attacker to retrieve sensitive information from the database.

Potential impact can include:

- Database version and technology disclosure
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

This lab demonstrated a **UNION-based SQL injection** against an Oracle database.

The main concepts were:

- Identifying a SQL injection point
- Determining the number of columns using `ORDER BY`
- Identifying a text-compatible and visible column
- Using Oracle's `dual` table
- Using `UNION SELECT` to retrieve additional data
- Querying `v$version` to obtain database version information
- Understanding how SQL comments can terminate the original query
