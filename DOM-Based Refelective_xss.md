# DOM-Based Reflected Cross-Site Scripting (XSS)

## Summary

A DOM-based reflected Cross-Site Scripting (XSS) vulnerability exists in the application's search functionality.

User-controlled input from the `search` URL parameter is reflected in the server response and processed by client-side JavaScript using the dangerous `eval()` function. This allows an attacker to execute arbitrary JavaScript in a victim's browser.

---

## Vulnerability Type

* Cross-Site Scripting (XSS)
* DOM-Based Reflected XSS

---

## Source

User-controlled input is supplied through the following URL parameter:

```
?search=<user_input>
```

Example:

```
https://target-site/?search=test
```

---

## Sink

Within `searchResults.js`, the server response is processed using:

```javascript
eval('var searchResultsObj = ' + this.responseText);
```

The `eval()` function interprets the supplied string as JavaScript code and executes it in the browser.

If attacker-controlled data reaches this sink, arbitrary JavaScript execution becomes possible.

---

## Impact

Successful exploitation may allow:

* Session hijacking
* Cookie theft
* Credential theft
* Arbitrary JavaScript execution
* Defacement or modification of page content
* User impersonation

---

## Technical Analysis

When a user performs a search from the application's landing page, the search term is transmitted through the `search` URL parameter.

The server processes this parameter and returns a response containing search-related data.

The JavaScript file `searchResults.js` receives this response and passes it directly to the `eval()` function:

```javascript
eval('var searchResultsObj = ' + this.responseText);
```

Because `eval()` executes JavaScript code contained within the supplied string, an attacker can manipulate the response structure and inject malicious JavaScript that will execute in the victim's browser.

The root cause of the vulnerability is the unsafe use of `eval()` on data that can be influenced by user input.

---

## Proof of Concept

### Payload

```
?search=anything\"-alert(1)}//
```

### Example Request

```
https://target-site/?search=anything%20\%22-alert(1)}//
```

### Result

When the crafted payload is processed by the vulnerable `eval()` statement, the injected JavaScript executes and displays an alert dialog, demonstrating arbitrary script execution.

---

## Remediation

Avoid using `eval()` to process server responses.

Instead:

1. Return properly formatted JSON from the server.
2. Parse the response using `JSON.parse()`.
3. Validate and sanitize all user-controlled data.
4. Implement a strict Content Security Policy (CSP).
5. Use safe DOM APIs instead of dynamic code execution.

Example:

```javascript
var searchResultsObj = JSON.parse(this.responseText);
```

---

## Severity

Medium to High

The final severity depends on the application's context, authentication state, available attack surface, and the sensitivity of exposed user data.
