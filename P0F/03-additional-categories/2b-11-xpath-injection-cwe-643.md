<!--
  P0F — Defensive CVE study notes.
  This file is for LEARNING vulnerability classes and remediation ONLY.
  It contains NO exploit code, NO payloads, and NO reproduction steps — by design.
  Code blocks show the *shape* of vulnerable vs fixed code taken from public patches,
  so you can recognize and fix the bug class, not weaponize it.
-->

# 2b.11 XPath Injection (CWE-643)

CWE: CWE-643 (https://cwe.mitre.org/data/definitions/643.html)
Root cause: Untrusted data is concatenated into an XPath expression (often a string literal), so the query structure can be changed and a different node set is selected than the one the code intended.
Source → sink: Document field or request parameter (an XML ID, URI, or search string) → string-built XPath → the query matches a different node, or every node, than the intended ID.
Code smells / review questions
- Is every value interpolated into XPath first validated against a grammar (for IDs, NCName) or bound as a variable, never concatenated into a quoted literal?
- Smell: building an ID query by concatenating document or request input, especially during XML signature verification.
Fix pattern
- Do not concatenate; use the XPath engine variable binding, or allow only values that match the NCName (or tighter) grammar before interpolation; fail closed on a mismatch.
Detection ideas
- Grep for XPath construction from document URIs or request fields, especially in XML-DSig / SAML libraries. Prefer a version that validates IDs before building the query.
Real CVE lessons
- CVE-2026-9390 (Perl XML::Sig before 0.71): a Reference URI was concatenated into an XPath ID lookup. CISA-ADP CVSS 3.1 9.1 (no NVD primary score). CWE-643 is a CNA Secondary; NVD lists no primary CWE (also CWE-1287 Secondary). Not in KEV. Walked in §3d. -- https://nvd.nist.gov/vuln/detail/CVE-2026-9390
OSCP / report angle
- Show the string-built XPath and the missing grammar check. Do not include an injected expression in the report.
