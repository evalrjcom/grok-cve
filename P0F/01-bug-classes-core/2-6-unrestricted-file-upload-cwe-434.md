<!--
  P0F — Defensive CVE study notes.
  This file is for LEARNING vulnerability classes and remediation ONLY.
  It contains NO exploit code, NO payloads, and NO reproduction steps — by design.
  Code blocks show the *shape* of vulnerable vs fixed code taken from public patches,
  so you can recognize and fix the bug class, not weaponize it.
-->

# 2.6 Unrestricted File Upload (CWE-434)

CWE: CWE-434 (https://cwe.mitre.org/data/definitions/434.html)
Root cause: Upload accepts dangerous types or stores them where the server will execute/serve them unsafely.
Source → sink: Multipart file → weak type check → web-executable directory or double-extension confusion.
Code smells / review questions
- Grep: upload libs without MIME+extension allow-list; store under docroot; trust Content-Type.
- Is stored name random? Directory non-executable? Content sniffed server-side?
- Smell: .html/.svg allowed (stored XSS); path traversal inside upload filename.
Fix pattern
- Allow-list + content validation; randomize names; store outside web root; disable script exec.
Detection ideas
- Review avatar/import/attachment; check inline SVG/HTML serving.
Real CVE lessons
- [supplementary] CVE-2023-50164 (Apache Struts): Upload param manipulation enabled path abuse to place a malicious file (NVD CWE-552). — https://nvd.nist.gov/vuln/detail/CVE-2023-50164
OSCP / report angle
- Show accepted-type policy gap and storage location. Do not leave a webshell on the target.

