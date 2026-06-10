# Reflected Cross-Site Scripting (XSS) in Blog Search Functionality

## Summary

A Reflected Cross-Site Scripting (XSS) vulnerability was identified in the blog search functionality. User-controlled input supplied through the search parameter is reflected into the HTML response without adequate output encoding, allowing attacker-controlled markup to be interpreted by the browser.

As a result, an attacker can cause arbitrary client-side script execution in the context of the vulnerable application.

---

## Vulnerability Details

| Field                  | Value                                |
| ---------------------- | ------------------------------------ |
| Vulnerability Type     | Reflected Cross-Site Scripting (XSS) |
| Severity               | High                                 |
| Affected Functionality | Blog Search                          |
| Source                 | Search Parameter                     |
| Sink                   | Search Results Page                  |

---

## Description

The application reflects user-supplied search input directly into the search results page.

During testing, it was observed that input filtering was present; however, the filtering mechanism failed to adequately sanitize all browser-interpretable markup before rendering the response.

User-controlled input is inserted into the HTML document and subsequently parsed by the browser, resulting in execution of attacker-controlled content.

---

## Source

### User Input

The search functionality accepts user-controlled input through the search parameter:

```http
GET /?search=<user_input>
```

The supplied value is reflected back to the client within the search results page.

---

## Sink

The user input is rendered within the search results section of the page:

```html
<section class="blog-header">
    <h1>
        0 search results for 'USER_INPUT'
    </h1>
    <hr>
</section>
```

Because the input is reflected into the HTML response without proper context-aware output encoding, browser-interpretable content may be executed.

---

## Impact

Successful exploitation allows execution of arbitrary JavaScript in the victim's browser within the application's origin.

Potential impacts include:

* Session compromise
* Unauthorized actions performed as the victim
* Access to sensitive application data
* DOM manipulation and content modification
* Phishing and social engineering attacks
* Theft of security-sensitive information accessible to client-side scripts

---

## Proof of Concept

### Request

```http
GET /?search=<payload>
```

### Vulnerable Response

```html
<section class="blog-header">
    <h1>
        0 search results for 'USER_INPUT'
    </h1>
    <hr>
</section>
```

The browser interprets the reflected content as active markup rather than rendering it as plain text.

---

## Root Cause

The application performs insufficient sanitization and output encoding of user-controlled input before reflecting it into the HTML response.

The filtering mechanism attempts to restrict certain input patterns but fails to ensure that all browser-interpretable content is safely handled.

---

## Remediation

1. Apply context-aware output encoding before rendering user-controlled data into HTML responses.
2. Treat all user input as untrusted.
3. Avoid rendering user-supplied content as HTML unless explicitly required.
4. If HTML input must be supported, use a robust allowlist-based HTML sanitizer.
5. Deploy a Content Security Policy (CSP) to reduce the impact of client-side injection vulnerabilities.
6. Perform security testing on all reflected input points throughout the application.

---

## References

* OWASP Cross Site Scripting (XSS)
* OWASP XSS Prevention Cheat Sheet
* OWASP Application Security Verification Standard (ASVS)
