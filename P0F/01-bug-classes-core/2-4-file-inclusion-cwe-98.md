<!--
  P0F — Defensive CVE study notes.
  This file is for LEARNING vulnerability classes and remediation ONLY.
  It contains NO exploit code, NO payloads, and NO reproduction steps — by design.
  Code blocks show the *shape* of vulnerable vs fixed code taken from public patches,
  so you can recognize and fix the bug class, not weaponize it.
-->

# 2.4 File Inclusion (CWE-98)

CWE: CWE-98 (https://cwe.mitre.org/data/definitions/98.html)
Root cause: Application includes/executes a file path derived from attacker input (LFI/RFI patterns).
Source → sink: Param (page/template/lang) → include/require/import → code or sensitive content.
Code smells / review questions
- Grep: include($_GET, require $param, user-fed template loaders.
- Can the resolved path escape the template root? Are wrappers filtered?
- Smell: base dir + raw user input then include.
Fix pattern
- Allow-list map of known templates; disable remote URL includes; open_basedir-style constraints.
Vulnerable shape (educational)

Fixed shape

Detection ideas
- Hunt params named page, template, lang, module, file; taint to include/require.
OSCP / report angle
- Report parameter + include sink + allow-list fix. Do not exfiltrate sensitive file contents into the ticket.

