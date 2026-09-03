# SQL Injection — Category Filter

## Lab

PortSwigger Web Security Academy — SQL injection vulnerability in the product category filter.

## Objective

The objective of this lab is to exploit a SQL injection vulnerability in the product category filter and make the application display one or more unreleased products.

## Initial Analysis

The application allows users to filter products by category.

The application performs a SQL query similar to:

```sql
SELECT * FROM products WHERE category = 'Gifts' AND released = 1
```

The `category` value is controlled by the user, making it a potential point for SQL injection testing.

## Exploitation

I modified the category parameter using the following payload:

```text
Food'or+1=1--+-
```

The payload modifies the logic of the original SQL query.

The `1=1` condition is always true, while the SQL comment causes the remaining part of the original query to be ignored.

Conceptually, the query becomes:

```sql
SELECT * FROM products WHERE category = 'Food' OR 1=1--' AND released = 1
```

As a result, the condition restricting the results to released products is bypassed.

## Result

The application displayed unreleased products, confirming the SQL injection vulnerability and completing the lab.

## Vulnerability

**SQL Injection (SQLi)**

The vulnerability occurs because user-controlled input is incorporated into a SQL query without adequate parameterization.

## Impact

Depending on the application's implementation and database privileges, SQL injection can potentially allow an attacker to:

- Bypass application logic
- Access unauthorized data
- Modify database contents
- Extract sensitive information

The actual impact depends on the database configuration and privileges.

## Remediation

The primary mitigation is to use **parameterized queries / prepared statements** instead of concatenating user input directly into SQL statements.

Additional measures include:

- Applying least-privilege database permissions
- Validating input where appropriate
- Avoiding dynamic SQL construction
- Using secure ORM/query APIs
- Monitoring suspicious database activity

## Key Takeaways

This lab demonstrated how SQL injection can modify the logic of a database query.

The main concepts were:

- Identifying user-controlled SQL input
- Breaking out of the original string context
- Using an always-true condition
- Using SQL comments to remove the remainder of the original query
- Understanding how the final SQL statement is interpreted by the database
