<!--
  P0F — Defensive CVE study notes.
  This file is for LEARNING vulnerability classes and remediation ONLY.
  It contains NO exploit code, NO payloads, and NO reproduction steps — by design.
  Code blocks show the *shape* of vulnerable vs fixed code taken from public patches,
  so you can recognize and fix the bug class, not weaponize it.
-->

# 3.17 NULL Pointer Dereference (CWE-476)

CWE: CWE-476 (https://cwe.mitre.org/data/definitions/476.html)
Root cause: Code dereferences a pointer that may be NULL, typically causing crash/DoS; sometimes a step in a larger bug.
Source → sink: Failed lookup/alloc returns NULL → unchecked use → crash.
Code smells / review questions
- Are all fallible allocs/lookups checked before use?
- Smell: error paths that skip initialization; optional fields assumed present after parse.
Fix pattern
- Explicit NULL checks; defensive coding standards; fuzz renegotiation/optional-extension paths.
Detection ideas
- Crash triage under ASAN; unit tests for omitted optional fields.
Real CVE lessons
- CVE-2021-3449 (OpenSSL TLS): NULL deref on crafted TLSv1.2 renegotiation ClientHello omitting signature_algorithms; CWE-476; not KEV. — https://nvd.nist.gov/vuln/detail/CVE-2021-3449
CVE walk — CVE-2021-3449 (OpenSSL 1.1.1 < 1.1.1k — NULL dereference on TLS 1.2 renegotiation)
Why it matters: A NULL pointer dereference (CWE-476) normally means a crash, which here is a remotely triggered denial of service of any TLS 1.2 server with renegotiation enabled (the default). OpenSSL rates it High; OpenSSL 1.0.2 is not affected and clients are not impacted.
Source → sink: Source = a renegotiation ClientHello whose extensions differ from the first handshake. Sink = tls12_shared_sigalgs() walking the peer signature-algorithm list using a stale length while the array pointer had been freed.
Fix: Commit fb9fa6b51defd48157eeb207f52181f735d96148, "ssl sigalg extension: fix NULL pointer dereference" (Nokia, 15 Mar 2021): reset the length when the list is freed. The commit message traces the bug to c589c34e61, which added TLS 1.3 signature_algorithms_cert support in 2018.
Scoring: NVD CVSS 3.1 5.9 (AV:N/AC:H/PR:N/UI:N/S:U/C:N/I:N/A:H); CVSS 2 4.3; vendor severity High — CVSS and vendor severity disagree. CWE-476. Not in KEV.
Detection: Fuzz renegotiation and state-reuse paths; review code that frees a pointer without resetting its companion length or flag; SCA for 1.1.1 < 1.1.1k.
Remediation: Upgrade to ≥ 1.1.1k; disable TLS renegotiation where it is not needed; free, NULL and zero the length as one idiom.
Takeaway: Pointer and length are one invariant; state that persists across handshakes is a bug source. Bounty: DoS-only issues are often out of scope — check the program rules; OSCP: NULL deref is availability only.
- Sources: https://nvd.nist.gov/vuln/detail/CVE-2021-3449 · https://www.openssl.org/news/secadv/20210325.txt · https://github.com/openssl/openssl/commit/fb9fa6b51defd48157eeb207f52181f735d96148
OSCP / report angle
- Report the unchecked path and the added guard. Usually DoS-class — state impact accurately.

