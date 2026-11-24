<!--
  P0F — Defensive CVE study notes.
  This file is for LEARNING vulnerability classes and remediation ONLY.
  It contains NO exploit code, NO payloads, and NO reproduction steps — by design.
  Code blocks show the *shape* of vulnerable vs fixed code taken from public patches,
  so you can recognize and fix the bug class, not weaponize it.
-->

# 2.1 OS Command Injection (CWE-78)

CWE: CWE-78 (https://cwe.mitre.org/data/definitions/78.html)
Root cause: Untrusted input becomes part of a shell command string that is interpreted by a shell.
Source → sink: HTTP/file/env input → string concat into system()/popen()/Runtime.exec(String)/os.system → OS shell.
Code smells / review questions
- Grep: system(, popen(, exec(, os.system, subprocess shell=True, Runtime.exec(single string), backticks.
- Is user-controlled data ever adjacent to a metacharacter-interpreted string?
- Smell: blacklist "escape" helpers; joining argv then handing to a shell.
Fix pattern
- Prefer argv arrays / ProcessBuilder with shell=False.
- Allow-list arguments if a shell is unavoidable; drop privileges.
Vulnerable shape (educational)

Fixed shape

Detection ideas
- SAST for shell=True / string-built exec; review backup/export/ping-style features.
Real CVE lessons
- [supplementary] CVE-2014-6271 (GNU Bash (Shellshock)): Env/function-definition strings as an unexpected sink. KEV 2022-01-28. — https://nvd.nist.gov/vuln/detail/CVE-2014-6271
- [supplementary] CVE-2024-21887 (Ivanti Connect Secure): Web-component command construction; CWE-77 sibling; KEV 2024-01-10. — https://nvd.nist.gov/vuln/detail/CVE-2024-21887
OSCP / report angle
- Show source, sink, missing allow-list. Recommend argv-array APIs. No working payload in public reports when policy forbids it.

