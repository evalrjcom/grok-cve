<!--
  P0F — Defensive CVE study notes.
  This file is for LEARNING vulnerability classes and remediation ONLY.
  It contains NO exploit code, NO payloads, and NO reproduction steps — by design.
  Code blocks show the *shape* of vulnerable vs fixed code taken from public patches,
  so you can recognize and fix the bug class, not weaponize it.
-->

# 2.7 Authentication Bypass (CWE-287)

CWE: CWE-287 (https://cwe.mitre.org/data/definitions/287.html)
Root cause: A request reaches a privileged action without proving the required identity/authenticator.
Source → sink: Request to sensitive route → missing/incorrect authn check → privileged handler.
Code smells / review questions
- Is auth enforced in middleware for every route, or only in some controllers?
- Smell: TODO near OrgID/role checks; weaker alternate entry points; spoofable headers.
- Smell: route authorization vs rendered-resource identity mismatch.
Fix pattern
- Central deny-by-default middleware; session-bound identity; regression tests on every admin route.
Detection ideas
- Map all routes; re-test after auth patches for bypass paths.
Real CVE lessons
- CVE-2024-38856 (Apache OFBiz): Route vs resource authz; 18.12.15 missed controller — 18.12.16 follow-up. KEV 2024-08-27. — https://nvd.nist.gov/vuln/detail/CVE-2024-38856
- [supplementary] CVE-2024-27198 (JetBrains TeamCity): Auth bypass enabling admin actions before 2023.11.4; KEV 2024-03-07. — https://nvd.nist.gov/vuln/detail/CVE-2024-27198
- [supplementary] CVE-2022-1388 (F5 BIG-IP): Missing authentication on undisclosed requests; KEV 2022-05-10. — https://nvd.nist.gov/vuln/detail/CVE-2022-1388
OSCP / report angle
- Document the unauthenticated request reaching a privileged sink. Never alter production data out of scope.

