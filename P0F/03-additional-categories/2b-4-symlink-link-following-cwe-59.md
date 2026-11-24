<!--
  P0F — Defensive CVE study notes.
  This file is for LEARNING vulnerability classes and remediation ONLY.
  It contains NO exploit code, NO payloads, and NO reproduction steps — by design.
  Code blocks show the *shape* of vulnerable vs fixed code taken from public patches,
  so you can recognize and fix the bug class, not weaponize it.
-->

# 2b.4 Symlink / Link Following (CWE-59)

CWE: CWE-59 (https://cwe.mitre.org/data/definitions/59.html)
Root cause: The product follows a symbolic or hard link (often inside an archive, repo, or shared directory) without confirming the resolved target still lies inside an intended base directory.
Source → sink: Untrusted archive, repository, or shared filesystem entry includes a link → extract/clone/write follows the link → a file is created or overwritten outside the destination tree.
Code smells / review questions
- Does extract/clone/copy resolve links and then re-check the canonical path against the destination root?
- Smell: trusting archive member names and link targets; disabling the safe extract filter; following links in a world-writable staging directory.
Fix pattern
- Refuse or carefully re-base links; after resolving, confirm the canonical path stays under the destination; prefer APIs that provide a safe extract/clone mode and keep it on.
Detection ideas
- Review archive and VCS extract paths for link-target checks; inventory Archive_Tar / tarfile / Git versions; prefer the library safe-extract option (see also the §3c tarfile walk, which is CWE-22).
Real CVE lessons
- CVE-2020-36193 (PEAR Archive_Tar through 1.4.11): symlink targets were not constrained to the extract path. NVD CVSS 3.1 7.5. NVD lists both CWE-59 and CWE-22 as Primary. KEV 2022-08-25. Walked in §3d. -- https://nvd.nist.gov/vuln/detail/CVE-2020-36193
- Related (CWE-59, verified, not walked here): Git CVE-2024-32002 (NVD 9.0, hook written via a submodule/link confusion; GHSA-8h77-4q3w-gfgv) and Git CVE-2025-48384 (KEV 2025-08-25). -- https://nvd.nist.gov/vuln/detail/CVE-2024-32002
OSCP / report angle
- Treat extract and clone as write primitives. Report a missing "stay inside dest" check on link targets. Do not include crafted archives or link names in the report.
