<!--
  P0F — Defensive CVE study notes.
  This file is for LEARNING vulnerability classes and remediation ONLY.
  It contains NO exploit code, NO payloads, and NO reproduction steps — by design.
  Code blocks show the *shape* of vulnerable vs fixed code taken from public patches,
  so you can recognize and fix the bug class, not weaponize it.
-->

# 3.15 Double Free (CWE-415)

CWE: CWE-415 (https://cwe.mitre.org/data/definitions/415.html)
Root cause: The same memory is freed twice, corrupting allocator metadata / enabling later misuse.
Source → sink: error/retry path frees object → another path frees again → allocator corruption.
Code smells / review questions
- Are all error paths setting the pointer NULL after free? Shared ownership without clear owner?
- Smell: free in both success and cleanup labels; packet ring / buffer teardown races.
Fix pattern
- Single owner; NULL after free; smart pointers/RAII; ASAN in CI.
Detection ideas
- Fuzz teardown/error paths; review every free with multiple exits.
Real CVE lessons
- CVE-2021-22600 (Linux af_packet): Double free in packet_set_ring(); local priv-esc/DoS class; KEV 2022-04-11. — https://nvd.nist.gov/vuln/detail/CVE-2021-22600
CVE walk — CVE-2021-22600 (Linux AF_PACKET — stale union field freed on ring reconfiguration; KEV)
Why it matters: A double free (CWE-415) corrupts allocator state and in kernel code can lead to local privilege escalation or a crash. Per the commit message, packet sockets may switch ring versions whose state fields share a union, so a cleanup path freed rx_owner_map even when no ring was ever allocated and the field held old state from another version. CISA listed it in KEV on 2022-04-11 (ransomware use: Unknown).
Source → sink: Source = a local process reconfiguring its packet-socket ring between versions. Sink = the out_free_pg_vec cleanup label in packet_set_ring() (net/packet/af_packet.c), which freed unconditionally. Details of heap layout are deliberately not covered.
Fix: Commit ec6af094ea28f0f2dda1a6a33b14cd57e36a9755, "net/packet: rx_owner_map depends on pg_vec" (Willem de Bruijn, 15 Dec 2021; Reported-by Syzbot; Fixes 61fad6816fc1): free only when the object it belongs to was allocated. Fixed stable releases per branch: unverified.
Scoring: NVD CVSS 3.1 7.0 (AV:L/AC:H/PR:L/UI:N/S:U/C:H/I:H/A:H); CVSS 2 7.2; Google (CNA) 6.6 (AC:H/PR:L/UI:R/S:C/C:L/I:L/A:H) — interaction and scope differ. CWE-415 on both. KEV 2022-04-11, ransomware Unknown.
Detection: Compare uname -r with distribution advisories; kernel fuzzing (syzkaller found this one) and KASAN in CI; review cleanup labels for frees not guarded by the same condition as the allocation.
Remediation: Apply the distribution kernel update; where AF_PACKET is not needed, limit who can create packet sockets (capabilities, container and seccomp profiles, user-namespace policy).
Takeaway: Pair every free with the condition that guarded its allocation, reset pointers after free, and give unions a discriminator. OSCP and bounty: local-kernel LPE is rarely in bounty scope; document the version check and the diff, not an exploit.
- Sources: https://nvd.nist.gov/vuln/detail/CVE-2021-22600 · https://github.com/torvalds/linux/commit/ec6af094ea28f0f2dda1a6a33b14cd57e36a9755 · KEV feed: https://www.cisa.gov/sites/default/files/feeds/known_exploited_vulnerabilities.json
OSCP / report angle
- Describe the dual-free paths and the ownership fix. No allocator-metadata exploit discussion.

