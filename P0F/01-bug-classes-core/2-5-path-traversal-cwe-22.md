<!--
  P0F — Defensive CVE study notes.
  This file is for LEARNING vulnerability classes and remediation ONLY.
  It contains NO exploit code, NO payloads, and NO reproduction steps — by design.
  Code blocks show the *shape* of vulnerable vs fixed code taken from public patches,
  so you can recognize and fix the bug class, not weaponize it.
-->

# 2.5 Path Traversal (CWE-22)

CWE: CWE-22 (https://cwe.mitre.org/data/definitions/22.html)
Root cause: User-controlled path segments escape the intended directory via parent-directory segments or their encoded variants.
Source → sink: Filename/path param → join(base, user) without containment → filesystem read/write.
Code smells / review questions
- Grep: path.join + user, sendFile(user), Zip Slip entry names.
- After canonicalize, does the path still start with the allowed root?
- Smell: ".." blacklist missing encodings; incomplete normalization patches.
Fix pattern
- Resolve absolute; verify prefix against real root; allow-list filenames; validate archive entries.
Detection ideas
- Re-test after path-normalization patches — incomplete fixes are common.
Real CVE lessons
- [supplementary] CVE-2021-41773 (Apache HTTP Server 2.4.49): Normalization mapped URLs outside Alias dirs; KEV 2021-11-03. — https://nvd.nist.gov/vuln/detail/CVE-2021-41773
- [supplementary] CVE-2021-42013 (Apache HTTP Server 2.4.50): Incomplete fix for 41773 — patch-bypass lineage; KEV 2021-11-03. — https://nvd.nist.gov/vuln/detail/CVE-2021-42013
- [supplementary] CVE-2019-11510 (Ivanti Pulse Connect Secure): Unauth crafted URI → arbitrary file read; KEV 2021-11-03. — https://nvd.nist.gov/vuln/detail/CVE-2019-11510
OSCP / report angle
- Emphasize containment after canonicalize. Demonstrate with a harmless in-scope file and stop.

