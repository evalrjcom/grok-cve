<!--
  P0F — Defensive CVE study notes.
  This file is for LEARNING vulnerability classes and remediation ONLY.
  It contains NO exploit code, NO payloads, and NO reproduction steps — by design.
  Code blocks show the *shape* of vulnerable vs fixed code taken from public patches,
  so you can recognize and fix the bug class, not weaponize it.
-->

# 2b.14 CSV / Formula Injection (CWE-1236)

CWE: CWE-1236 (https://cwe.mitre.org/data/definitions/1236.html)
Root cause: Untrusted data is written into a CSV (or similar spreadsheet export) without neutralizing leading characters that spreadsheet software treats as a formula, so a later user who opens the file may execute that formula.
Source → sink: User-controlled or remotely supplied field → exported as a CSV cell → a spreadsheet program interprets the cell as a formula when a person opens the file.
Code smells / review questions
- Does every export path treat cells as data, including values that start with a formula-introducing character?
- Smell: concatenating request or banner fields into CSV with only quote-escaping of double-quotes, no leading-character check.
Fix pattern
- Prefix potentially formula-leading cells so the spreadsheet stores them as text (a common pattern is a leading apostrophe); document that exports are untrusted data; prefer formats that do not auto-execute formulas.
Detection ideas
- Review every CSV/XLSX export. Confirm a sanitiser runs on each cell, not only on one column. Do not include a formula-looking cell in the report.
Real CVE lessons
- CVE-2018-11652 (Nikto 2.1.6 and earlier): the Server banner (and other fields) were written into the CSV report unsanitised. NVD CVSS 3.0 9.8 (this score assumes a high-impact formula open; the practical prerequisite is that someone opens the report in a spreadsheet). CWE-1236. Not in KEV. Walked in §3d. -- https://nvd.nist.gov/vuln/detail/CVE-2018-11652
OSCP / report angle
- Bug bounty: report the export path and the missing sanitiser. Do not include a formula cell or a spreadsheet-command example in the write-up.

