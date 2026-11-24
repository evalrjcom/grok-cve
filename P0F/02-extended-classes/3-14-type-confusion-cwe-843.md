<!--
  P0F — Defensive CVE study notes.
  This file is for LEARNING vulnerability classes and remediation ONLY.
  It contains NO exploit code, NO payloads, and NO reproduction steps — by design.
  Code blocks show the *shape* of vulnerable vs fixed code taken from public patches,
  so you can recognize and fix the bug class, not weaponize it.
-->

# 3.14 Type Confusion (CWE-843)

CWE: CWE-843 (https://cwe.mitre.org/data/definitions/843.html)
Root cause: Code accesses a resource using a type incompatible with the actual allocated type, often in dynamic/JIT engines.
Source → sink: Object shape/map invalidated → engine still uses old type assumptions → memory corruption class impact.
Code smells / review questions
- Review JIT/engine code for map transitions without guards; serialization that confuses types.
- Smell: casting without tag checks; array element kind transitions; bytecode verifiers that skip cases.
Fix pattern
- Maintain type tags; re-check after shape changes; fuzz engines; keep browser/runtime patched.
Detection ideas
- Track browser/engine KEV entries; enable sandboxing for untrusted content.
Real CVE lessons
- CVE-2020-6418 (Google Chrome V8): Type confusion in V8 prior to 80.0.3987.122; CWE-843; KEV 2021-11-03. — https://nvd.nist.gov/vuln/detail/CVE-2020-6418
CVE walk — CVE-2023-0286 (OpenSSL 1.1.1 < 1.1.1t, 3.0 < 3.0.8 — X.400 address type confusion)
Why it matters: Type confusion (CWE-843): memory is created as one type and read as another. Here the public header declared a field with a different type than the parser actually stored, so a comparison routine interpreted a string object as a generic ASN.1 value; per the advisory an attacker supplying a certificate chain and CRL could pass arbitrary pointers to memcmp, a possible read primitive or crash. OpenSSL rates it High.
Source → sink: Source = X.509 data containing an X.400 address GeneralName. Sink = GENERAL_NAME_cmp() in crypto/x509v3/v3_genn.c, which used the header's declared type. The advisory says it matters when CRL checking is enabled.
Fix: Commit 2c6c9d439b484e1ba9830d8454a34fa4f80fdfe9 (Hugo Landau, 1.1.1 branch; the 3.0 and 1.0.2 branches have sibling commits of the same title), fixed in 1.1.1t and 3.0.8 (1.0.2zg for premium support, per the advisory): change the declared field type to what the parser stores, and the comparison with it.
Scoring: NVD CVSS 3.1 7.4 (AV:N/AC:H/PR:N/UI:N/S:U/C:H/I:N/A:H), same from CISA-ADP; OpenSSL severity High. CWE-843 (NVD and CISA-ADP). Not in KEV.
- Closed-source KEV sibling: Chrome V8 CVE-2020-6418 (KEV 2021-11-03, NVD 8.8, CWE-843), fixed in Chrome 80.0.3987.122 per NVD; release notes https://chromereleases.googleblog.com/2020/02/stable-channel-update-for-desktop_24.html — fix source diff not verified.
Detection: Fuzzing and sanitizers (ASan/UBSan) in CI; static analysis for union and void* casts; review headers that declare a type different from what the parser builds; SCA for OpenSSL versions above.
Remediation: Upgrade OpenSSL; prefer tagged unions or typed templates; in memory-safe languages use sum types.
Takeaway: Declaration and implementation must agree — a one-word header fix closed it. Bug bounty and OSCP: type-confusion reports need the diff and the affected-version check, not a crash dump; do not weaponise.
- Sources: https://nvd.nist.gov/vuln/detail/CVE-2023-0286 · https://www.openssl.org/news/secadv/20230207.txt · https://github.com/advisories/GHSA-x4qr-2fvf-3mr5 · https://github.com/openssl/openssl/commit/2c6c9d439b484e1ba9830d8454a34fa4f80fdfe9
OSCP / report angle
- For OSCP/reporting: cite the engine advisory and patch level. No heap-type-confusion exploit writeups.

