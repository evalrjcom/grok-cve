<!--
  P0F — Defensive CVE study notes.
  This file is for LEARNING vulnerability classes and remediation ONLY.
  It contains NO exploit code, NO payloads, and NO reproduction steps — by design.
  Code blocks show the *shape* of vulnerable vs fixed code taken from public patches,
  so you can recognize and fix the bug class, not weaponize it.
-->

# 2.17 CSRF (CWE-352)

CWE: CWE-352 (https://cwe.mitre.org/data/definitions/352.html)
Root cause: Victim's browser sends an authenticated state-changing request the app cannot distinguish from intentional action.
Source → sink: Victim session cookie → attacker page triggers GET/POST → state change without anti-CSRF secret.
Code smells / review questions
- Does every state-changing endpoint require a session-bound token and reject GET for mutations?
- Smell: $_REQUEST creds; token only on POST while GET mutates; UA startswith 'Mozilla' as "auth" (Ray); missing sesskey.
Fix pattern
- Anti-CSRF tokens; SameSite cookies; require POST; verify Origin/Sec-Fetch-Site; never treat UA as auth.
Detection ideas
- Crawl for state-changing GET; review unauthenticated local services for DNS-rebinding + CSRF.
Real CVE lessons
- CVE-2025-62593 (Ray): CSRF + DNS rebinding; UA filter replaced (70e7c72) also rejects Sec-Fetch-*; 2.52.0. KEV 2026-08-17. Header filtering ≠ auth. — https://nvd.nist.gov/vuln/detail/CVE-2025-62593
- CVE-2021-43559 (Moodle): Missing sesskey on badges/related_action.php; 3.11.4+. NVD 8.8 vs vendor Serious. — https://nvd.nist.gov/vuln/detail/CVE-2021-43559
- CVE-2019-12616 (phpMyAdmin): Login CSRF: $_REQUEST → $_POST (015c404); 4.9.0. — https://nvd.nist.gov/vuln/detail/CVE-2019-12616
- CVE-2019-12922 (phpMyAdmin): State-changing GET; require POST + PMA_token (427fbed); 4.9.1. — https://nvd.nist.gov/vuln/detail/CVE-2019-12922
OSCP / report angle
- Show missing token or GET mutation. Do not host a live CSRF page against production outside a program.

