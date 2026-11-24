<!--
  P0F — Defensive CVE study notes.
  This file is for LEARNING vulnerability classes and remediation ONLY.
  It contains NO exploit code, NO payloads, and NO reproduction steps — by design.
  Code blocks show the *shape* of vulnerable vs fixed code taken from public patches,
  so you can recognize and fix the bug class, not weaponize it.
-->

# 2b.5 Regular Expression DoS (ReDoS) (CWE-1333)

CWE: CWE-1333 (https://cwe.mitre.org/data/definitions/1333.html)
Root cause: A regular expression has super-linear (often quadratic or exponential) worst-case complexity, and untrusted input can be fed to it in a hot path such as request parsing.
Source → sink: User-supplied string (date, UA, version, URL) → a vulnerable regex in a parser → CPU bound on one request, denying service to others.
Code smells / review questions
- Are untrusted strings length-capped before regex parsing? Can the engine backtrack on nested or overlapping groups?
- Smell: comment/whitespace strippers, email/date/semver validators, and structured-text parsers with nested quantifiers and no timeout.
Fix pattern
- Rewrite the expression to be linear (possessive or non-overlapping); cap input length; use a regex engine with a time or step bound; prefer a real parser over a regex for grammars like dates and versions.
Detection ideas
- Inventory regexes on request paths (date, UA, HTML, versions); run a safe regex-complexity linter; check library versions (moment, semver, markdown, URI parsers) against advisories. Do not use crafted inputs against production.
Real CVE lessons
- CVE-2022-31129 (moment.js 2.18.0 through < 2.29.4): RFC 2822 comment stripping was quadratic. NVD and GHSA both CVSS 3.1 7.5. NVD primary CWE-1333 (GHSA also lists CWE-400 as Secondary). Not in KEV. Walked in §3d. -- https://nvd.nist.gov/vuln/detail/CVE-2022-31129
OSCP / report angle
- For bug bounty, show the regex, the missing length cap, and the patched expression. Do not include a malicious input string in the report.
