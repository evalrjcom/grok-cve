<!--
  P0F — Defensive CVE study notes.
  This file is for LEARNING vulnerability classes and remediation ONLY.
  It contains NO exploit code, NO payloads, and NO reproduction steps — by design.
  Code blocks show the *shape* of vulnerable vs fixed code taken from public patches,
  so you can recognize and fix the bug class, not weaponize it.
-->

# 2b.6 CRLF / HTTP Header Injection (CWE-113, CWE-93)

CWE: CWE-113 (https://cwe.mitre.org/data/definitions/113.html) · CWE-93 (https://cwe.mitre.org/data/definitions/93.html)
Root cause: User-influenced data is written into an HTTP header (or another line-delimited protocol field) without stripping or rejecting line-break characters, so extra headers or a response body can be injected.
Source → sink: Request parameter, filename, or URL fragment → copied into a header value (Location, Content-Disposition, a custom header, or a backend protocol line) → additional headers or commands appear on the wire.
Code smells / review questions
- Is every header value validated against an allow-list, or at least stripped of CR/LF and other non-printables, after any URL-decode?
- Smell: user input in Content-Disposition filenames, redirect URLs, or logging/forwarding headers; encode-then-decode gaps.
Fix pattern
- Reject or replace CR, LF and non-printable bytes after decoding; prefer an allow-list of characters for filename and callback parameters; never build a header by string concatenation of unsanitised input.
Detection ideas
- Grep for header-setting APIs that take request parameters; review filename/callback/redirect fields; check language HTTP clients (urllib, and similar) for known CRLF CVEs.
Real CVE lessons
- CVE-2018-18837 (Netdata 1.10.0): a filename (and related) parameter reached a response header. NVD CVSS 3.0 6.1. CWE-113. Not in KEV. Walked in §3d. -- https://nvd.nist.gov/vuln/detail/CVE-2018-18837
- Related: CPython urllib CVE-2019-9740 (CWE-93; NVD 6.1) -- a request URL could introduce extra headers; fixed in the listed 2.7/3.5/3.6/3.7 releases. -- https://nvd.nist.gov/vuln/detail/CVE-2019-9740
- Related (exploited): Zimbra CVE-2022-27924 (KEV 2022-08-04, ransomware Known). NVD primary CWE is 74, KEV lists CWE-93; NVD 7.5 vs ADP 9.8. Command injection into a cache protocol via unsanitised protocol lines. -- https://nvd.nist.gov/vuln/detail/CVE-2022-27924
OSCP / report angle
- Show the header-construction sink and the missing allow-list. Do not include line-break sequences or injected header names in the report.
