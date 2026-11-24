<!--
  P0F — Defensive CVE study notes.
  This file is for LEARNING vulnerability classes and remediation ONLY.
  It contains NO exploit code, NO payloads, and NO reproduction steps — by design.
  Code blocks show the *shape* of vulnerable vs fixed code taken from public patches,
  so you can recognize and fix the bug class, not weaponize it.
-->

# 2.16 Integer Overflow / Truncation / OOB from size math (CWE-190, 195, 197, 787)

CWE: CWE-190 (https://cwe.mitre.org/data/definitions/190.html) · CWE-195 (https://cwe.mitre.org/data/definitions/195.html) · CWE-197 (https://cwe.mitre.org/data/definitions/197.html) · CWE-787 (https://cwe.mitre.org/data/definitions/787.html)
Root cause: Arithmetic on lengths wraps or truncates, so a too-small buffer is allocated or an OOB copy runs.
Source → sink: Attacker length fields → add/mul without overflow check → alloc/copy → heap/stack OOB (often CWE-787).
Code smells / review questions
- Grep: size=n*elems; used+n<avail; unsigned subtraction as index.
- Rewrite as avail-used>n with used<=avail precondition (OpenSSL lesson).
- Smell: buffer from fixed worst-case constant (libwebp); pad=align-actual (x_tables); size math above NULL check (zlib).
Fix pattern
- Checked arithmetic; measure-then-alloc (libwebp 902bc91); fuzz length-prefixed codecs; UBSan.
Vulnerable shape (educational)

Fixed shape

Detection ideas
- UINT_MAX boundary unit tests; re-audit size-check patches for NULL/order invariants.
Real CVE lessons
- CVE-2016-2105 (OpenSSL EVP_EncodeUpdate): 5b81448. NVD 7.5 vs OpenSSL Low. Rewrite used+n<avail. — https://nvd.nist.gov/vuln/detail/CVE-2016-2105
- CVE-2022-37434 (zlib): eff308a then 1eb7682 patch²; zlib 1.2.13. NVD 9.8 CWE-787 (CISA-ADP CWE-120). Unsigned offset; vendored libs. — https://nvd.nist.gov/vuln/detail/CVE-2022-37434
- CVE-2023-4863 (libwebp): 902bc91 measure-then-alloc. 1.3.2 / Chrome 116.0.5845.187. KEV 2023-09-13. Size math assumes valid input. — https://nvd.nist.gov/vuln/detail/CVE-2023-4863
- CVE-2021-22555 (Linux x_tables): b29c457 removed pad memsets. NVD 7.8 vs Google CNA 8.3; KEV 2025-10-06. align-actual lengths; unprivileged user ns. — https://nvd.nist.gov/vuln/detail/CVE-2021-22555
OSCP / report angle
- Show the wrapping comparison and the safe rewrite. Mention CVSS disagreements. No exploit primitives.

