<!--
  P0F — Defensive CVE study notes.
  This file is for LEARNING vulnerability classes and remediation ONLY.
  It contains NO exploit code, NO payloads, and NO reproduction steps — by design.
  Code blocks show the *shape* of vulnerable vs fixed code taken from public patches,
  so you can recognize and fix the bug class, not weaponize it.
-->

# 2b.12 Sensitive Data in Error Messages (CWE-209)

CWE: CWE-209 (https://cwe.mitre.org/data/definitions/209.html)
Root cause: An error path returns or displays secrets that should stay on the server (expected signatures, stack traces with keys, file paths, or SQL fragments), and that message is visible to a less-privileged caller.
Source → sink: Failed verification or exception → error string includes a secret from the check (for example the expected signature) → the caller reads the error and reuses the secret.
Code smells / review questions
- Do API error bodies include expected HMAC/signature values, tokens, connection strings, or full exception traces in production?
- Smell: catch-and-return of the exception message to the client; debug mode left on; a validation-failed message that includes the expected secret.
Fix pattern
- Return a generic error to the client; log details only server-side, and never log the expected secret; disable debug traces in production.
Detection ideas
- Trigger failed authentication and failed signature checks in a test environment and inspect the response body. Search code for error strings that interpolate expected values.
Real CVE lessons
- CVE-2019-7644 (Auth0-WCF-Service-JWT before 1.0.4): a failed JWT signature check leaked the expected signature in the error message, which was enough to forge a token. NVD CVSS 3.0 9.8. CWE-209. Not in KEV. Walked in §3d. -- https://nvd.nist.gov/vuln/detail/CVE-2019-7644
OSCP / report angle
- Show the error body from a test of your own token and the missing genericisation. Do not replay a leaked signature against a production system outside a program.
