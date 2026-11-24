<!--
  P0F — Defensive CVE study notes.
  This file is for LEARNING vulnerability classes and remediation ONLY.
  It contains NO exploit code, NO payloads, and NO reproduction steps — by design.
  Code blocks show the *shape* of vulnerable vs fixed code taken from public patches,
  so you can recognize and fix the bug class, not weaponize it.
-->

# 3.18 sudo / SUID / Windows Privilege-Escalation Misconfigurations (CWE-250, 269)

CWE: CWE-250 (https://cwe.mitre.org/data/definitions/250.html) · CWE-269 (https://cwe.mitre.org/data/definitions/269.html)
Root cause: Programs or OS configs grant unnecessary elevated rights, or privileged tools mishandle inputs so a low-priv user gains higher privileges.
Source → sink: sudoers/SUID binary / privileged Windows dialog → weak policy or input handling → root/SYSTEM.
Code smells / review questions
- Review sudoers for NOPASSWD + writable scripts; SUID binaries that shell out or parse env unsafely.
- Windows: service binary permissions, AlwaysInstallElevated, unquoted paths, token privileges, dialog UAC gaps.
- Smell: apache/www-data allowed to run root commands via helper scripts (EyesOfNetwork class).
Fix pattern
- Least privilege; remove unused SUID; sudo allow-lists with fixed args; harden Windows services; patch privileged dialogs.
Detection ideas
- Audit sudo -l equivalents and SUID finds in labs; review Windows service ACLs.
Real CVE lessons
- CVE-2020-8655 (EyesOfNetwork 5.3): sudoers allows apache→root via a user-supplied nmap script; CWE-269; KEV 2021-11-03. — https://nvd.nist.gov/vuln/detail/CVE-2020-8655
- CVE-2019-1388 (Windows Certificate Dialog): UAC/privilege elevation via Certificate Dialog; CWE-269; KEV 2023-04-07, ransomware Known. — https://nvd.nist.gov/vuln/detail/CVE-2019-1388
CVE walk — CVE-2019-13272 (Linux kernel < 5.1.17 — ptrace credentials recorded for the wrong subject; KEV)
Why it matters: A privilege-management bug (CWE-269 family): the kernel recorded the credentials of the wrong process when a ptrace relationship was created, so later access checks used a privileged parent's credentials instead of the requester's. NVD: local users can obtain root in certain parent-child scenarios. CISA listed it in KEV on 2021-12-10 (ransomware use: Unknown).
Source → sink: Source = an unprivileged process requesting a ptrace relationship with its parent. Sink = ptrace_link() in kernel/ptrace.c, which stored the parent's credentials as if the parent were the acting subject; a second flaw used an RCU reference as if it were a stable one. Exploitation chain is not described here.
Fix: Commit 6994eefb0053799d2e07cd140df6c2ea106c41ee, "ptrace: Fix ->ptracer_cred handling for PTRACE_TRACEME" (Jann Horn, 4 Jul 2019), kernel 5.1.17 per NVD: always record the credentials of the process that requests the relationship.
Scoring: NVD CVSS 3.1 7.8 (AV:L/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:H), same from CISA-ADP; CVSS 2 7.2. CWE: NVD has NVD-CWE-noinfo, while the KEV record lists CWE-269. KEV 2021-12-10, ransomware Unknown.
- Contrast, userspace: pkexec "PwnKit" CVE-2021-4034 (KEV 2022-06-27; NVD CWE is memory safety, not 269): fix commit https://gitlab.freedesktop.org/polkit/polkit/-/commit/a2bf5c9c83b6ae46cbd5c779d3055bff81ded683 rejects a missing argv in the setuid helper. EON CVE-2020-8655 (sudoers misconfiguration, KEV) and Windows CVE-2019-1388 (certificate dialog, KEV, closed source) remain lessons above.
Detection: uname -r against distribution advisories; audit SUID/SGID binaries and sudoers (sudo -l); Yama ptrace_scope restricts ptrace relationships.
Remediation: Apply the kernel update; reduce SUID surface; least-privilege sudoers entries without wildcard or writable-path commands.
Takeaway: Access checks must use the acting subject's credentials. OSCP: enumerate kernel version and SUID/sudo, match against fixed versions, document — do not run public exploits outside the lab. Bounty: local LPE usually needs a container/cloud-escape framing.
- Sources: https://nvd.nist.gov/vuln/detail/CVE-2019-13272 · https://github.com/torvalds/linux/commit/6994eefb0053799d2e07cd140df6c2ea106c41ee · KEV feed: https://www.cisa.gov/sites/default/files/feeds/known_exploited_vulnerabilities.json
OSCP / report angle
- OSCP: document the misconfig (sudoers line, SUID path, service ACL) and the least-privilege fix. Related classics often tagged other CWEs (e.g. sudo CVE-2019-14287 CWE-755; pkexec CVE-2021-4034 CWE-125/787) — cite their actual NVD CWE.

