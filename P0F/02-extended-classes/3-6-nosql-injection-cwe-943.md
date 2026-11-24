<!--
  P0F — Defensive CVE study notes.
  This file is for LEARNING vulnerability classes and remediation ONLY.
  It contains NO exploit code, NO payloads, and NO reproduction steps — by design.
  Code blocks show the *shape* of vulnerable vs fixed code taken from public patches,
  so you can recognize and fix the bug class, not weaponize it.
-->

# 3.6 NoSQL Injection (CWE-943)

CWE: CWE-943 (https://cwe.mitre.org/data/definitions/943.html)
Root cause: Untrusted input alters a NoSQL query document/operators (e.g., MongoDB $gt/$where) instead of being treated as a value.
Source → sink: JSON/query param → driver query object built by merge/concat → database operator injection.
Code smells / review questions
- Grep: find(req.body), $where, raw JSON merged into queries; Elasticsearch/Firestore query builders that interpolate expressions.
- Are operator keys ($...) stripped from user objects before query?
- Smell: parsing JSON from query string directly into filters.
Fix pattern
- Allow-list query fields; strip $-prefixed keys; typed query builders; disable server-side JS ($where).
Detection ideas
- Review every Mongo/ES/Firestore query built from request bodies.
Real CVE lessons
- CVE-2026-85167 (n8n): Query injection in Elasticsearch Get All / Firestore Document Query via expression interpolation; CWE-943; fixed 2.35.4 / 2.36.2; not KEV. — https://nvd.nist.gov/vuln/detail/CVE-2026-85167
- CVE-2021-22911 (Rocket.Chat 3.11–3.13): Unauth NoSQL injection (NVD CWE-75 / NVD-CWE-Other; description confirms NoSQL); not KEV. — https://nvd.nist.gov/vuln/detail/CVE-2021-22911
CVE walk — CVE-2026-63489 (Yeti ≤ 2.5.0 — ArangoDB AQL injection via sort and filter field names)
Why it matters: NoSQL injection is not only MongoDB operators: any query language assembled by string interpolation can be steered. This case is the identifier-position variant: untrusted values placed where field names go (sorting, filter aliases), where bind parameters cannot be used.
Source → sink: Source = sorting fields and filter_aliases in an authenticated API request. Sink = f-strings that build AQL in core/database_arango.py (neighbors() and filter()). Chokepoint = the query-builder module.
Fix: Commit 89d8b61cf97b0fb2d46c2f9ee8b7411444345fc6, "fix: prevent AQL injection via sorting and filter_aliases parameters" (PR #1277, 1 Jul 2026), release 2.5.1 (fixed version taken from the release link in the NVD references): a strict charset validator runs before any field name is interpolated.
Scoring: NVD status Deferred, so no NVD score. GitHub (CNA) CVSS 3.1 7.6 (AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:L/A:L); CWE-943. Not in KEV. The record was published on 2026-11-03, four months after the fix; scores may change.
- Not used as the walk (no public fix diff verified): n8n CVE-2026-85167 and Rocket.Chat CVE-2021-22911, kept as lessons above. Note n8n has two GHSA IDs in circulation: NVD links GHSA-wxwj-8wv6-vpw2, GitHub's advisory database lists GHSA-7mp8-q29w-rc68; both pages load, which one is canonical is unverified.
Detection: Grep for f-strings, format() and concatenation inside query text; on list endpoints, send malformed sort/field keys in a lab and compare 400 vs 500 behaviour.
Remediation: Upgrade to ≥ 2.5.1. Bind all values as parameters; allow-list identifiers. A charset regex like this one is the weakest allow-list — prefer a fixed set of known field names.
Takeaway: Parameters protect values, not identifiers; identifiers need an allow-list. Bug bounty: report the unvalidated field and the engine-level error difference from your own data.
- Sources: https://nvd.nist.gov/vuln/detail/CVE-2026-63489 · https://github.com/yeti-platform/yeti/security/advisories/GHSA-vw33-fwfx-6m9p · https://github.com/yeti-platform/yeti/commit/89d8b61cf97b0fb2d46c2f9ee8b7411444345fc6 · https://github.com/yeti-platform/yeti/pull/1277 · https://github.com/yeti-platform/yeti/releases/tag/2.5.1
OSCP / report angle
- Show the query-construction site and operator-key sanitization fix. Do not dump database contents.

