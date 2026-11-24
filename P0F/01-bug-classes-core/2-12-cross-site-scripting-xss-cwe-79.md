<!--
  P0F — Defensive CVE study notes.
  This file is for LEARNING vulnerability classes and remediation ONLY.
  It contains NO exploit code, NO payloads, and NO reproduction steps — by design.
  Code blocks show the *shape* of vulnerable vs fixed code taken from public patches,
  so you can recognize and fix the bug class, not weaponize it.
-->

# 2.12 Cross-Site Scripting / XSS (CWE-79)

CWE: CWE-79 (https://cwe.mitre.org/data/definitions/79.html)
Root cause: Untrusted data is written into HTML/JS/CSS contexts without context-appropriate encoding, or a sanitizer is bypassed.
Source → sink: Param/stored field → DOM sink (.innerHTML, .html()) or server template without encoding → victim browser.
Code smells / review questions
- Grep: .html(, .innerHTML, dangerouslySetInnerHTML, incomplete SVG/MathML sanitizers.
- Is every reflected/stored value encoded for its context?
- Smell: stripos/indexOf for 'svg' ignoring whitespace (Roundcube); sanitize-then-transform (jQuery); request parameters written into a template without escaping.
Fix pattern
- Output-encode per context; textContent / auto-escape; maintained sanitizers; CSP as defense in depth.
Vulnerable shape (educational)

Fixed shape

Detection ideas
- Hunt reflected params; re-test sanitizers against whitespace, case, nested tags, SVG.
Real CVE lessons
- CVE-2023-5631 (Roundcube): wash_uri stripos 'svg'; fixed by stripping whitespace (41756cc). KEV 2023-10-26. — https://nvd.nist.gov/vuln/detail/CVE-2023-5631
- CVE-2023-37580 (Zimbra): a request parameter written unescaped into the page → wrapped in the JSTL XML-escape function (874ac8c). KEV 2023-07-27. — https://nvd.nist.gov/vuln/detail/CVE-2023-37580
- CVE-2020-11022 (jQuery): htmlPrefilter regex (1d61fd9). Sanitize-then-transform. — https://nvd.nist.gov/vuln/detail/CVE-2020-11022
- CVE-2020-11023 (jQuery): option-element wrapMap (966a709); KEV 2025-01-23. DOM XSS sinks like .html(). — https://nvd.nist.gov/vuln/detail/CVE-2020-11023
OSCP / report angle
- Show source→sink with missing encoding. Avoid browser-firing PoCs when policy asks for non-weaponized reports.

