# Security Advisory — Stored Cross-Site Scripting (XSS) in Modified eCommerce Shopsoftware (Currencies "Title" field)


> **Publication date:** 12/11/2026
> **Last updated:** 12/11/2026
> **Severity:** Medium
> **CVSS:** 5.4 — `CVSS:3.1/AV:N/AC:L/PR:L/UI:R/S:C/C:L/I:L/A:N`
> **CWE:** CWE-79 — Improper Neutralization of Input During Web Page Generation ('Cross-site Scripting')
> **Status:** Under investigation / vendor notification in progress

---

## 1. Executive summary

A stored Cross-Site Scripting (XSS) vulnerability was identified in **Modified eCommerce Shopsoftware**, affecting the administration panel's **Currencies** configuration (`admin/currencies.php`). An authenticated back-end user with access to currency management can inject a malicious payload through the **Title** field, which is stored by the application and executed in the browser of any administrator who later views the currencies page.

Successful exploitation can result in **execution of arbitrary JavaScript in the context of the administration panel**, enabling session/cookie theft, actions performed on behalf of the victim administrator, and back-end interface manipulation.

The responsible vendor was notified on **[date — to be completed]** and **[summary of the vendor response — to be completed]**.

---

## 2. Affected products

| Product / Component | Affected versions | Fixed version | Status |
|---|---:|---:|---|
| Modified eCommerce Shopsoftware | v3.3.0-r16825 | [to be confirmed] | Affected |

### Unaffected products

- [Product or version not affected — to be confirmed]

---

## 3. Vulnerability description

The vulnerability occurs because user-supplied input is stored and later rendered in the HTML response **without proper output encoding/sanitization**.

The **Currency "Title"** field, handled by `admin/currencies.php` during the `edit`/`insert` action, does not correctly neutralize HTML/JavaScript metacharacters. The value is persisted in the database and reflected back into the currencies listing, allowing an attacker to inject an HTML element with an event handler (e.g. `<svg onload=...>`) that runs arbitrary JavaScript when the page is rendered.

### Required conditions

- **Authentication:** required (valid back-end/administration account).
- **Privilege:** a user with access to the Currencies configuration (`Configuration → Currencies`).
- **Access vector:** network / remote (HTTP access to the administration panel).
- **User interaction:** required — the payload executes when an administrator opens the currencies page.
- **Specific configuration:** none beyond a standard installation.

---

## 4. Impact

Exploitation may allow:

- Execution of arbitrary JavaScript in the administrator's browser session.
- Theft of session cookies / tokens and potential session hijacking.
- Actions performed in the administrator's context (CSRF-style abuse of authenticated functionality).
- Defacement or manipulation of the back-end interface presented to other administrators.

### Impact on confidentiality

Scripts executing in the admin context can read page content and tokens accessible to the victim's browser session, potentially exposing sensitive back-end data.

### Impact on integrity

An attacker can alter interface content and trigger authenticated operations on behalf of the victim administrator.

### Impact on availability

No direct, intentional service interruption. Availability impact is considered negligible for this issue.

---

## 5. Classification

### CVSS

- **Score:** 5.4
- **Severity:** Medium
- **Vector:** `CVSS:3.1/AV:N/AC:L/PR:L/UI:R/S:C/C:L/I:L/A:N`

### CWE

- **CWE-79 — Improper Neutralization of Input During Web Page Generation ('Cross-site Scripting')**

### CAPEC

- **CAPEC-592 — Stored XSS**

---

## 6. Exploitation scenario

A possible exploitation scenario unfolds as follows:

1. The attacker holds a back-end account with access to the Currencies configuration of a vulnerable **Modified eCommerce Shopsoftware** instance.
2. The attacker edits (or creates) a currency and submits a JavaScript payload in the **Title** field.
3. The application stores the value without sanitization and reflects it in the currencies listing.
4. When another administrator opens `admin/currencies.php`, the stored payload executes in their browser.
5. As a consequence, arbitrary JavaScript runs in the administration context, enabling session theft or unauthorized actions.

---

## 7. Technical evidence

### Affected component

```text
admin/currencies.php  —  parameter: "title"  (action=edit / action=insert)
```

### Example request

```http
POST /admin/currencies.php?page=1&cID=5&action=save HTTP/1.1
Host: [TARGET-HOST]
Content-Type: application/x-www-form-urlencoded
Cookie: [AUTHENTICATED-ADMIN-SESSION]

title=<svg onload=alert('poc-san')>&code=AAA&symbol_left=$&symbol_right=&decimal_point=.&thousands_point=,&decimal_places=2
```

### Observed response

The stored value is returned unencoded within the currencies listing, causing the injected element to be parsed and its event handler to execute.

### Result

The payload `<svg onload=alert('poc-san')>` stored in the **Title** field is rendered as live HTML, and the `onload` handler runs, confirming arbitrary JavaScript execution in the administration panel. The screenshots below show the stored payload and its execution.

**Stored payload in the "Title" field:**

[![Stored XSS payload in the Currencies Title field](images/poc-01-stored-payload.jpg)](https://github.com/marcusvinisantos/Advisory/blob/main/poc-01-stored-payload.jpg)

**Payload execution (JavaScript alert fires):**

[![Stored XSS execution — alert('poc-san')](images/poc-02-execution.jpg)](https://github.com/marcusvinisantos/Advisory/blob/main/poc-02-execution.jpg)

> **Note:** Remove tokens, cookies, credentials, personal data, internal addresses, and other secrets before publication.

---

## 8. Proof of concept

The proof of concept below demonstrates only the vulnerable behavior and must be used exclusively in authorized environments.

```text
1. Log in to the administration panel.
2. Navigate to: Configuration → Currencies.
3. Edit a currency (e.g. admin/currencies.php?page=1&cID=5&action=edit).
4. In the "Title" field, enter the payload:

   <svg onload=alert('poc-san')>

5. Save the change and open the Currencies listing.
```

### Expected result

```text
A JavaScript dialog box displaying "poc-san" is shown, confirming
arbitrary script execution in the context of the administration panel.
```

### PoC limitations

- Does not cause intentional denial of service.
- Does not remove or modify third-party data.
- Does not create persistence or a backdoor.
- Contains no real credentials.
- Does not automate mass exploitation.

---

## 9. Reproduction steps

1. Access a vulnerable **Modified eCommerce Shopsoftware** installation with a valid admin account.
2. Ensure access to the Currencies configuration.
3. Open `Configuration → Currencies` and edit a currency entry.
4. Submit `<svg onload=alert('poc-san')>` in the **Title** field and save.
5. Observe the `alert('poc-san')` dialog when the currencies page renders.
6. Compare with the expected secure behavior: the value should be displayed as inert text, with HTML metacharacters encoded.

---

## 10. Mitigation

Until a definitive fix is applied, it is recommended to:

- Restrict access to the Currencies configuration to trusted administrators only.
- Review which back-end roles can manage currencies and apply least privilege.
- Apply a WAF / reverse-proxy rule to flag HTML/script metacharacters in the `title` parameter.
- Monitor administration requests to `admin/currencies.php` for injection patterns.
- Invalidate sessions and tokens potentially exposed through execution of injected scripts.
- Retain logs and evidence for investigation.

> Mitigations reduce risk but may not fully eliminate the vulnerability.

---

## 11. Fix

The fix is expected to be made available in version **[fixed version — to be confirmed]**.

It is recommended to:

1. Update to **[fixed version]** or later as soon as it is published.
2. Restart affected services when necessary.
3. Invalidate old sessions and credentials.
4. Review logs prior to the update.
5. Confirm the vulnerable behavior can no longer be reproduced.

### Implemented change

[High-level description of the vendor fix — to be completed. Expected: context-aware output encoding (HTML entity encoding) of the currency title when rendered, and/or server-side input validation.]

---

## 12. Detection and indicators

Possible indicators of exploitation:

- Currency records whose **Title** contains HTML tags or event handlers (e.g. `<svg`, `onerror=`, `onload=`, `<script`).
- Requests to `admin/currencies.php` containing encoded or raw HTML/script metacharacters in the `title` parameter.
- Unexpected JavaScript errors or dialogs in the administration panel.
- Anomalous outbound connections from administrator sessions.

### Example log search

```text
admin/currencies\.php.*(title=).*(%3C|<)(svg|script|img|iframe).*(onload|onerror|onfocus)=
```

---

## 13. Disclosure timeline

| Date | Event |
|---|---|
| [DD/MM/YYYY] | Vulnerability identified |
| [DD/MM/YYYY] | First notification sent to the vendor |
| [DD/MM/YYYY] | Acknowledgement of receipt |
| [DD/MM/YYYY] | Vulnerability validated |
| [DD/MM/YYYY] | CVE requested / reserved |
| [DD/MM/YYYY] | Fix made available |
| 12/11/2026 | Advisory published |
| 12/11/2026 | Last update |

---

## 14. Vendor communication

- **Vendor:** Modified eCommerce Shopsoftware
- **Channel used:** [email, form, bug bounty program, CERT, etc. — to be completed]
- **Date of first notification:** [DD/MM/YYYY]
- **Response status:** [Responded / No response / Under analysis]
- **Vendor position:** [Objective summary — to be completed]

---

## 15. Credits

[RedScan Research Team](https://redscanacademy.com/)

The vulnerability was identified and reported by the **RedScan Research Team**:

| Role | Member |
|---|---|
| 🥇 Lead Researcher | Carlos Tuma (bl4dsc4n) |
| 🥇 Lead Researcher | Smith Braz (smithbraz) |
| Researcher (discovery and report) | Marcus Santos (M4r7vini) |

With honors and deep gratitude to **Carlos Tuma** and **Smith Braz**, who lead the RedScan Research Team, for building the lab, mentoring every step of this research and teaching responsible, coordinated disclosure.

A research group of students and instructors from **RedScan Academy**.

---

## 16. References

- Vendor repository: https://github.com/modified-shop/modified-shop
- [Vendor advisory link — to be completed]
- [CVE record link — to be completed]
- [NVD link — to be completed]
- CWE-79: https://cwe.mitre.org/data/definitions/79.html
- CAPEC-592: https://capec.mitre.org/data/definitions/592.html
- [Patch / commit link — to be completed]

---

## 17. Revision history

| Version | Date | Change |
|---|---|---|
| 1.0 | 12/11/2026 | Initial publication |

---

## 18. Legal notice

This advisory is published for educational, defensive, and security-improvement purposes.

The information presented was obtained in an authorized environment and disclosed responsibly or in a coordinated manner. The author does not encourage the use of this information for unauthorized access, service disruption, privacy violations, or any illegal activity.

Use of the information in this document is the sole responsibility of the reader.

---

## 19. Contact

For corrections, updates, or additional information:

- **Email:**marcus.vini.gyn@gmail.com
