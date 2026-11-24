<!--
  P0F — Defensive CVE study notes.
  This file is for LEARNING vulnerability classes and remediation ONLY.
  It contains NO exploit code, NO payloads, and NO reproduction steps — by design.
  Code blocks show the *shape* of vulnerable vs fixed code taken from public patches,
  so you can recognize and fix the bug class, not weaponize it.
-->

# 2b.1 Unquoted Windows Service Path (CWE-428)

CWE: CWE-428 (https://cwe.mitre.org/data/definitions/428.html)
Root cause: A Windows service ImagePath (or similar search path) contains spaces and is not wrapped in quotes, so the service control manager may resolve an earlier, unintended path component instead of the intended binary.
Source → sink: Service start as a privileged account → unquoted ImagePath with spaces → the first matching path component is executed instead of the vendor binary.
Code smells / review questions
- Does every service ImagePath that contains spaces use a quoted, fully qualified path?
- Smell: installer writes an unquoted path under a directory whose name has spaces; no post-install audit of ImagePath.
Fix pattern
- Always quote the full ImagePath; prefer a path without spaces; lock down write ACLs on every directory along the path; treat this as an installer bug and ship a patch, not only a manual registry edit.
Detection ideas
- Inventory services and drivers whose ImagePath contains whitespace and is not quoted; confirm write permissions on intermediate directories. Prefer a vendor update over a one-off registry change.
Real CVE lessons
- CVE-2022-2147 (Cloudflare WARP for Windows 2022.2.95.0): unquoted service path; fixed in 2022.3.186.0. NVD CVSS 3.1 7.8 vs Cloudflare CNA 6.5. CWE-428. Not in KEV. Walked in §3d. -- https://nvd.nist.gov/vuln/detail/CVE-2022-2147
OSCP / report angle
- On a Windows host you own, record unquoted ImagePath values and the ACLs of intermediate directories. Report the missing quotes and the writable path; do not demonstrate execution of a substitute binary.
