<!--
  P0F — Defensive CVE study notes.
  This file is for LEARNING vulnerability classes and remediation ONLY.
  It contains NO exploit code, NO payloads, and NO reproduction steps — by design.
  Code blocks show the *shape* of vulnerable vs fixed code taken from public patches,
  so you can recognize and fix the bug class, not weaponize it.
-->

# 3.19 Cache Poisoning / Deception (CWE-349, 525)

CWE: CWE-349 (https://cwe.mitre.org/data/definitions/349.html) · CWE-525 (https://cwe.mitre.org/data/definitions/525.html)
Root cause: Attacker influences what a cache stores or keys on, so other users receive attacker-controlled content; or cached sensitive pages are later shown to the wrong party.
Source → sink: Ambiguous request / unkeyed header / query-parse disagreement → cache stores poisoned response → victims served it. Also: browser/back-forward cache exposing private pages (CWE-525).
Code smells / review questions
- Do caches key on Host, route, and auth? Are unkeyed headers (X-Forwarded-Host) reflected into content?
- Smell: front-end and origin disagree on query parsing (duplicate keys, semicolon separators).
- Smell: Cache-Control missing on authenticated HTML; bfcache of banking/settings pages.
Fix pattern
- Key caches on all selective inputs; normalize query parsing across layers; Cache-Control: no-store for private pages; avoid reflecting unkeyed headers.
Detection ideas
- Review CDN/proxy configs; test parser discrepancies between edge and origin with benign probes.
Real CVE lessons
- CVE-2021-23336 (CPython urllib.parse): parse_qsl/parse_qs discrepancy enabled web cache poisoning class issues (NVD primary CWE-444; description is cache poisoning). Fixed in 3.6.13/3.7.10/3.8.8/3.9.2; not KEV. — https://nvd.nist.gov/vuln/detail/CVE-2021-23336
CVE walk — CVE-2024-46982 (Next.js pages router < 13.5.7 / 14.2.10 — invalid default cache lifetime)
Why it matters: Cache poisoning (CWE-349/525 family): a request manipulates what a shared cache stores, so other users receive the result. Per NVD, a crafted request could make Next.js cache a non-dynamic SSR route in the pages router that was meant not to be cached, and send Cache-Control: s-maxage=1, stale-while-revalidate. Impact is availability (CNA vector A:H).
Root cause (from the fix): When rendering returned no revalidate value, the server substituted a default of one second, turning "do not cache" into "cacheable for one second plus stale-while-revalidate". The default was an invalid state papering over a missing invariant. Request-trigger details are not reproduced here.
Fix: Commits 7ed7f125e07ef0517a331009ed7e32691ba403d3 (canary) and bd164d53af259c05f1ab434004bcfdd3837d7cda (backport to the 14.x line, different variable name), both titled "Remove invalid fallback revalidate value (#69990)" (11 Sep 2024). Fixed in 13.5.7 and 14.2.10 per the GitHub advisory.
Scoring: GitHub (CNA) CVSS 3.1 7.5 (AV:N/AC:L/PR:N/UI:N/S:U/C:N/I:N/A:H). NVD status is Analyzed but the record carries only the CNA score and CISA-ADP SSVC, no NVD-assigned score or CWE. CWE: the NVD record shows CWE-639 (CNA); the GitHub advisory lists CWE-349 and CWE-639. Not in KEV.
Detection: Check Cache-Control on SSR and authenticated routes at the origin and the CDN; compare cache keys with inputs the app actually uses; SCA for Next.js versions above.
Remediation: Upgrade; send explicit no-store or private on dynamic and authenticated responses; at the CDN, cache only responses marked cacheable, key on every input that changes the response, and strip unkeyed inputs.
Takeaway: Defaults on missing values are cache-policy decisions — fail closed (no-cache). Bounty: demonstrate on your own cache key or a staging host, never poison shared production caches; report the header and the key discrepancy.
- Sources: https://nvd.nist.gov/vuln/detail/CVE-2024-46982 · https://github.com/vercel/next.js/security/advisories/GHSA-gp8f-8m3g-qvj9 · https://github.com/vercel/next.js/commit/7ed7f125e07ef0517a331009ed7e32691ba403d3 · https://github.com/vercel/next.js/commit/bd164d53af259c05f1ab434004bcfdd3837d7cda
OSCP / report angle
- Walk uses Next.js CVE-2024-46982: NVD has no CVE primarily tagged CWE-349 or CWE-525 (checked 2026-11-04); the NVD record carries CWE-639 from the CNA, while the GitHub advisory also tags CWE-349. Report cache-key / header reflection issues; do not poison shared production caches.
