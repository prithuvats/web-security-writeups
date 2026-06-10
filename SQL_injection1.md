# SQL Injection via Username Parameter

## Summary

A SQL Injection vulnerability exists in the login functionality. User-supplied input is incorporated into a SQL query without proper validation or parameterization, allowing an attacker to manipulate the query logic and bypass authentication.

## Vulnerability Type

**SQL Injection (Authentication Bypass)**

---

## Source

The vulnerable source is the username input field on the login page.

User-controlled input is processed by the backend and used directly within a SQL query.

## Sink

The sink is the database query responsible for validating user credentials.

Example query:

```sql
SELECT * FROM users
WHERE username = 'administrator'
AND password = 'password';
```

## Analysis

The application fails to properly handle user input before constructing the SQL query.

An attacker can inject SQL syntax into the username field. By using the SQL comment operator (`--`), the remainder of the query can be ignored by the database engine.

For example, supplying the following payload:

```text
administrator'--
```

causes the resulting query to become:

```sql
SELECT * FROM users
WHERE username = 'administrator'--
AND password = 'password';
```

Since everything after `--` is treated as a comment, the password verification logic is removed from the query. As a result, authentication is performed solely on the username value, potentially allowing unauthorized access to the administrator account.

## Proof of Concept

### Payload

```text
administrator'--
```

### Resulting Query

```sql
SELECT * FROM users
WHERE username = 'administrator'--
AND password = 'password';
```

### Impact

An attacker can bypass authentication and gain access to accounts without knowing the corresponding password.

## Remediation

1. Use parameterized queries (prepared statements).
2. Avoid constructing SQL queries through string concatenation.
3. Implement server-side input validation.
4. Apply the principle of least privilege to database accounts.
5. Log and monitor authentication anomalies.

## Conclusion

The login functionality is vulnerable to SQL Injection due to improper handling of user input. An attacker can exploit this flaw to bypass authentication and gain unauthorized access to user accounts. Implementing parameterized queries and proper input handling will mitigate this vulnerability.
