<!--
  P0F — Defensive CVE study notes.
  This file is for LEARNING vulnerability classes and remediation ONLY.
  It contains NO exploit code, NO payloads, and NO reproduction steps — by design.
  Code blocks show the *shape* of vulnerable vs fixed code taken from public patches,
  so you can recognize and fix the bug class, not weaponize it.
-->

# 2.15 Use-After-Free (CWE-416)

CWE: CWE-416 (https://cwe.mitre.org/data/definitions/416.html)
Root cause: Memory is used after it has been freed; often from re-entrancy, incorrect lifetime, or validation that skips lifetime rules.
Source → sink: free/drop object → stale pointer in list/callback/iterator → later dereference.
Code smells / review questions
- While iterating, can a callback free the current node?
- Smell: raw pointers without strong refs; validation bugs that accept illegal states; free then common epilogue.
Fix pattern
- Strong refs for iteration duration; exact-match validation; ASAN; prefer memory-safe languages.
Detection ideas
- Fuzz state machines; review every free paired with list removals and observers.
Real CVE lessons
- CVE-2024-1086 (Linux nf_tables): Verdict validation UAF; f342de4 exact match. KEV 2024-05-30, ransomware Known. Validation→lifetime. — https://nvd.nist.gov/vuln/detail/CVE-2024-1086
- CVE-2024-9680 (Firefox AnimationTimeline::Tick): RefPtr snapshot (0ee07613d050); chained with CVE-2024-49039. KEV 2024-10-15, ransomware Known. Re-entrancy; strong refs. — https://nvd.nist.gov/vuln/detail/CVE-2024-9680
OSCP / report angle
- Describe the lifetime bug and the strong-ref / exact-validation fix. No heap-grooming discussion.

