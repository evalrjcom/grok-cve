<!--
  P0F — Defensive CVE study notes.
  This file is for LEARNING vulnerability classes and remediation ONLY.
  It contains NO exploit code, NO payloads, and NO reproduction steps — by design.
  Code blocks show the *shape* of vulnerable vs fixed code taken from public patches,
  so you can recognize and fix the bug class, not weaponize it.
-->

# 2.14 Race Conditions / TOCTOU (CWE-362, 367)

CWE: CWE-362 (https://cwe.mitre.org/data/definitions/362.html) · CWE-367 (https://cwe.mitre.org/data/definitions/367.html)
Root cause: A check and a later use are not atomic, so state can change in between.
Source → sink: check(resource) → [concurrent change] → use(resource); non-atomic balance/flag updates.
Code smells / review questions
- Is check-then-act under one lock / transaction / conditional UPDATE?
- Smell: read row, decide in app, write without version/lock; access()+open(); COW write races.
Fix pattern
- Transactions with row locks; conditional UPDATE WHERE version=?; unique constraints; FOLL_COW-style atomicity.
Detection ideas
- Concurrent load-test state-changing endpoints; review money/inventory/invite/privilege-grant paths.
Real CVE lessons
- CVE-2016-5195 (Linux Dirty COW): Race on mm mapping; 19be0ea (FOLL_COW). KEV 2022-03-03. — https://nvd.nist.gov/vuln/detail/CVE-2016-5195
- CVE-2023-37904 (Discourse): Commits 62a609e, 25d481f. Check-then-act atomicity. — https://nvd.nist.gov/vuln/detail/CVE-2023-37904
OSCP / report angle
- Explain the non-atomic window and the locking fix. Discuss kernel races conceptually — not a priv-esc walkthrough.

