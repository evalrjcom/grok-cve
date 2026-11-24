<!--
  P0F — Defensive CVE study notes.
  This file is for LEARNING vulnerability classes and remediation ONLY.
  It contains NO exploit code, NO payloads, and NO reproduction steps — by design.
  Code blocks show the *shape* of vulnerable vs fixed code taken from public patches,
  so you can recognize and fix the bug class, not weaponize it.
-->

# 2b.2 DLL Search-Order Hijacking (CWE-427)

CWE: CWE-427 (https://cwe.mitre.org/data/definitions/427.html)
Root cause: The process loads a dynamic library using an underspecified search path (current directory, user-writable folder, or an attacker-influenced name) instead of a validated, fully qualified path to a known-good file.
Source → sink: Privileged process starts or handles an IPC request → library name resolved via the default search order → a library from a writable location is mapped into the privileged process.
Code smells / review questions
- Does every LoadLibrary/dlopen use a full path, preferably with an integrity or signature check?
- Smell: loading by basename only; trusting a path from IPC or a relative installer directory; writing unpack/installer files to a user-writable location and then loading from there.
Fix pattern
- Load only from a known, tightly ACL-protected directory using an absolute path; enable SafeDllSearchMode; do not load libraries whose path comes from unsanitised IPC; prefer delay-load with an explicit full path.
Detection ideas
- Review privileged Windows services and VPN/EDR clients for library loads by basename and for IPC that supplies a path. Compare the installed client version to the vendor advisory.
Real CVE lessons
- CVE-2020-3433 (Cisco AnyConnect for Windows < 4.9.00086): IPC channel allowed a DLL-hijack load. NVD and Cisco both CVSS 3.1 7.8. CWE-427. KEV 2022-10-24, ransomware Known. Walked in §3d. -- https://nvd.nist.gov/vuln/detail/CVE-2020-3433
- Related: CVE-2020-3153 (same product family, also CWE-427, KEV 2022-10-24, ransomware Known) is an installer directory-path issue that can be used to plant a library; NVD 6.5. -- https://nvd.nist.gov/vuln/detail/CVE-2020-3153
OSCP / report angle
- Record the privileged process, the search path, and whether a user can write to it. Do not plant a test library on a system you do not own; describe the load and the missing path validation.
