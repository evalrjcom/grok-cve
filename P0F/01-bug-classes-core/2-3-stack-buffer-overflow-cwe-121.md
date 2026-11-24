<!--
  P0F — Defensive CVE study notes.
  This file is for LEARNING vulnerability classes and remediation ONLY.
  It contains NO exploit code, NO payloads, and NO reproduction steps — by design.
  Code blocks show the *shape* of vulnerable vs fixed code taken from public patches,
  so you can recognize and fix the bug class, not weaponize it.
-->

# 2.3 Stack Buffer Overflow (CWE-121)

CWE: CWE-121 (https://cwe.mitre.org/data/definitions/121.html)
Root cause: A write exceeds a fixed stack buffer's bounds, typically via unsafe copy/format APIs.
Source → sink: Network/file length → strcpy/sprintf/gets/memcpy(len from packet) → stack buffer.
Code smells / review questions
- Grep: strcpy, strcat, sprintf, gets, unrestricted scanf %s.
- Is destination capacity checked against attacker-controlled length?
- Smell: trusted packet lengths; off-by-one NUL; wchar vs byte size confusion.
Fix pattern
- Bounded APIs (snprintf/strlcpy); ASAN/UBSAN in CI; canaries/NX/ASLR as defense in depth.
Vulnerable shape (educational)

Fixed shape

Detection ideas
- Fuzz length-prefixed parsers; review every memcpy where length is not constant.
OSCP / report angle
- Identify the vulnerable read into a fixed buffer and the missing bounds check. Describe crash signature — not a weaponized exploit chain.

