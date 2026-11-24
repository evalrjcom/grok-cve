<!--
  P0F — Defensive CVE study notes.
  This file is for LEARNING vulnerability classes and remediation ONLY.
  It contains NO exploit code, NO payloads, and NO reproduction steps — by design.
  Code blocks show the *shape* of vulnerable vs fixed code taken from public patches,
  so you can recognize and fix the bug class, not weaponize it.
-->

# 2b.13 Secrets in Log Files (CWE-532)

CWE: CWE-532 (https://cwe.mitre.org/data/definitions/532.html)
Root cause: Logs record credentials, cookies, tokens or other secrets (often as "dump all headers on error"), and those logs are readable by a broader set of operators or users than the secret should be.
Source → sink: Request or error path → logger writes headers, cookies, or tokens at a default level → anyone who can read the log can reuse the secret.
Code smells / review questions
- On 4xx/5xx, does the logger dump the full request, including Authorization and Cookie? Are refresh tokens in structured logs?
- Smell: dumping the entire request header map on error; debug logging left at info in production; no redaction list.
Fix pattern
- Allow-list the fields that may be logged; never log Authorization, Cookie, or token fields by default; redact known secret keys; restrict log ACL and retention.
Detection ideas
- Read a 5xx log line from a test request that carried a session cookie and confirm the cookie is absent. Grep log helpers for full-header dumps.
Real CVE lessons
- CVE-2022-24757 (Jupyter Server before 1.15.4): 5xx logging wrote request headers, including the auth cookie. NVD and GHSA CVSS 3.1 7.5. CWE-532 is GitHub Secondary; NVD lists no primary CWE. Not in KEV. Walked in §3d. -- https://nvd.nist.gov/vuln/detail/CVE-2022-24757
OSCP / report angle
- Show a redacted log sample from your own test request and the missing allow-list. Do not copy live session cookies into a ticket.
