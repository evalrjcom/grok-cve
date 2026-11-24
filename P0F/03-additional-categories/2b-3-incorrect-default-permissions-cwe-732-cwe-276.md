<!--
  P0F — Defensive CVE study notes.
  This file is for LEARNING vulnerability classes and remediation ONLY.
  It contains NO exploit code, NO payloads, and NO reproduction steps — by design.
  Code blocks show the *shape* of vulnerable vs fixed code taken from public patches,
  so you can recognize and fix the bug class, not weaponize it.
-->

# 2b.3 Incorrect Default Permissions (CWE-732, CWE-276)

CWE: CWE-732 (https://cwe.mitre.org/data/definitions/732.html) · CWE-276 (https://cwe.mitre.org/data/definitions/276.html)
Root cause: A privileged component is installed into a directory (or with a DACL / Unix mode) that a lower-privileged user can write, so the next time a privileged account runs that helper, it may execute a replacement file.
Source → sink: Installer or service account creates a helper under a writable directory → a standard user replaces the file → a later privileged run (login, service restart, helper invoke) executes the replacement.
Code smells / review questions
- Are ProgramData / Program Files / /usr/bin helpers writable by Users, Everyone, or a non-admin group? Are Unix bits overly permissive or group-writable on SUID/helper paths?
- Smell: writing runtime binaries into a shared, non-ACL-hardened directory; relying on the default DACL of a parent folder.
Fix pattern
- Install privileged helpers only into directories whose write ACL is limited to Administrators/SYSTEM (or root); audit after install; do not run helpers from user-writable locations.
Detection ideas
- On Windows, inspect ACLs on service binaries and ProgramData product directories. On Linux, look for writable parent dirs of SUID/sgid or sudo targets (this complements the §3.18 sudo/SUID class).
Real CVE lessons
- CVE-2019-15752 (Docker Desktop Community Edition before 2.1.0.1): a helper under a ProgramData directory was writable by a low-privilege user. NVD CVSS 3.1 7.8. CWE-732. KEV 2021-11-03. Walked in §3d. -- https://nvd.nist.gov/vuln/detail/CVE-2019-15752
OSCP / report angle
- Permission issues are core OSCP privesc: record the DACL/mode, the privileged consumer, and the vendor fix. Do not replace a binary to "prove" execution.
