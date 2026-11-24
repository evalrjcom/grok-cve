<!--
  P0F — Defensive CVE study notes.
  This file is for LEARNING vulnerability classes and remediation ONLY.
  It contains NO exploit code, NO payloads, and NO reproduction steps — by design.
  Code blocks show the *shape* of vulnerable vs fixed code taken from public patches,
  so you can recognize and fix the bug class, not weaponize it.
-->

# 2.10 SSRF (CWE-918)

CWE: CWE-918 (https://cwe.mitre.org/data/definitions/918.html)
Root cause: Server fetches a URL derived from attacker input, reaching internal/metadata network locations.
Source → sink: URL/webhook/import param → server-side HTTP client → internal IP / cloud metadata.
Code smells / review questions
- Grep: requests.get(user_url), webhook URL fields, PDF renderers, link-unfurlers.
- After redirects, is the final IP still public? Are file:/gopher: blocked?
- Smell: hostname allow-list without re-resolve at connect; only blocking 127.0.0.1.
Fix pattern
- Allow-list schemes+domains; resolve and pin IP; block private ranges; re-validate redirects.
Detection ideas
- Inventory every feature that fetches a user-supplied URL.
Real CVE lessons
- [supplementary] CVE-2021-26855 (Microsoft Exchange): SSRF as part of ProxyLogon; KEV 2021-11-03. — https://nvd.nist.gov/vuln/detail/CVE-2021-26855
OSCP / report angle
- Show the URL parameter and that the server fetches it. Do not pull cloud metadata credentials into the report.

