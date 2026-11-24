<!--
  P0F — Defensive CVE study notes.
  This file is for LEARNING vulnerability classes and remediation ONLY.
  It contains NO exploit code, NO payloads, and NO reproduction steps — by design.
  Code blocks show the *shape* of vulnerable vs fixed code taken from public patches,
  so you can recognize and fix the bug class, not weaponize it.
-->

# 2.2 SQL Injection (CWE-89)

CWE: CWE-89 (https://cwe.mitre.org/data/definitions/89.html)
Root cause: Untrusted input is concatenated into a SQL statement and parsed as SQL syntax.
Source → sink: Request/param → string concat / ORM raw() → database engine.
Code smells / review questions
- Grep: execute(f"...{, .format(, dynamic SQL, ORDER BY + user field.
- Does any identifier come from the client? Binding does not cover identifiers — need allow-lists.
- Smell: manual quote escaping; second-order SQLi.
Fix pattern
- Parameterized queries for values; allow-list identifiers; least-privilege DB accounts.
Vulnerable shape (educational)

Fixed shape

Detection ideas
- SAST taint to SQL builders; review reporting endpoints with dynamic WHERE.
Real CVE lessons
- [supplementary] CVE-2023-34362 (Progress MOVEit Transfer): SQLi on an internet-facing data-exchange edge; mass exploitation; KEV 2023-06-02. — https://nvd.nist.gov/vuln/detail/CVE-2023-34362
OSCP / report angle
- Quote the query construction and the bound-parameter fix; note impact without dumping data.

