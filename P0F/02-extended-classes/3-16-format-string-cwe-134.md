<!--
  P0F — Defensive CVE study notes.
  This file is for LEARNING vulnerability classes and remediation ONLY.
  It contains NO exploit code, NO payloads, and NO reproduction steps — by design.
  Code blocks show the *shape* of vulnerable vs fixed code taken from public patches,
  so you can recognize and fix the bug class, not weaponize it.
-->

# 3.16 Format String (CWE-134)

CWE: CWE-134 (https://cwe.mitre.org/data/definitions/134.html)
Root cause: Untrusted input is used as the format string argument to printf-family functions.
Source → sink: User string → printf(user) / syslog(user) / fprintf(log, user) → memory read/write via format specifiers.
Code smells / review questions
- Grep: printf(user), sprintf(buf, user), syslog(PRIORITY, user) — format arg must be literal.
- Smell: logging wrappers that pass client strings as the format.
Fix pattern
- Always use printf("%s", user); compiler flags (-Wformat -Werror=format-security); forbid user-controlled formats.
Detection ideas
- SAST for non-literal format args; review logging of external protocol fields (LLDP, etc.).
Real CVE lessons
- CVE-2019-1579 (Palo Alto PAN-OS GlobalProtect): Format-string RCE class issue on portal/gateway; CWE-134; KEV 2022-01-10, ransomware Known. — https://nvd.nist.gov/vuln/detail/CVE-2019-1579
CVE walk — CVE-2017-12588 (rsyslog < 8.28.0 — zmq3 modules used a config string as the format)
Why it matters: Format-string bug (CWE-134): a string that is not fully trusted is used as the format argument of a printf-style function, so the format directives in it are interpreted instead of printed. The consequences range from information disclosure to memory corruption, depending on the function.
Source → sink: Source = the "description" configuration field of the zmq3 input and output modules. Sink = zsocket_connect()/zsocket_bind(), printf-style variadic functions, called with that field as the format. Code is under contrib/imzmq3 and contrib/omzmq3.
Fix: Merge commit 062d0c671a29f7c6f7dff4a2f1f35df375bbb30b (PR #1565, "Fix format security issue in zmq3 modules"), whose commits are titled "imzmq3/omzmq3: Fix building with -Werror=format-security"; fixed in 8.28.0 per NVD. The flaw was surfaced by a compiler warning.
Scoring: NVD CVSS 3.0 9.8 (AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H); CVSS 2 7.5; CWE-134; GitHub advisory GHSA-f39p-m8r2-p6f6 also Critical 9.8. Not in KEV. Caveat (my assessment): the CVE text says "unspecified impact" and the tainted value is a configuration field, so real exploitability depends on who can set it; the 9.8 vector assumes a network attacker with no privileges.
- Closed-source KEV sibling: PAN-OS CVE-2019-1579 (GlobalProtect; KEV 2022-01-10, ransomware Known; NVD 8.1, CWE-134). Remediation is the vendor patch: https://security.paloaltonetworks.com/CVE-2019-1579
Detection: Build with -Wformat -Wformat-security -Werror=format-security (that is how this was found); grep printf-family calls, logging wrappers and syslog() with a non-literal first argument; static analysis; _FORTIFY_SOURCE.
Remediation: Upgrade; always pass a literal format and the data as an argument; audit wrappers around printf.
Takeaway: Treat compiler warnings as security signals. OSCP and bounty: format-string findings need the call site and the literal-format fix; show reachability from untrusted input, not memory-read output.
- Sources: https://nvd.nist.gov/vuln/detail/CVE-2017-12588 · https://github.com/advisories/GHSA-f39p-m8r2-p6f6 · https://github.com/rsyslog/rsyslog/pull/1565 · https://github.com/rsyslog/rsyslog/commit/062d0c671a29f7c6f7dff4a2f1f35df375bbb30b
OSCP / report angle
- Show the call site where user data is the format. Recommend literal "%s". No format-specifier attack strings.

