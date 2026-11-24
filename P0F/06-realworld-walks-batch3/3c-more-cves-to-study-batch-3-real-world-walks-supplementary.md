<!--
  P0F — Defensive CVE study notes.
  This file is for LEARNING vulnerability classes and remediation ONLY.
  It contains NO exploit code, NO payloads, and NO reproduction steps — by design.
  Code blocks show the *shape* of vulnerable vs fixed code taken from public patches,
  so you can recognize and fix the bug class, not weaponize it.
-->

# 3c. More CVEs to Study (batch 3) — Real-World Walks (supplementary)

Fifteen further CVE walks in the same format as §3 and §3b, chosen to fill class gaps that still lacked a full walk: Equifax-era Struts exception-to-expression handling, httpd mod_proxy SSRF, Magento XXE, ActiveMQ OpenWire deserialisation, edge-device auth bypass and broken access control, Exchange pre-auth path confusion, Citrix session-token over-read, glibc tunables overflow, Fortinet SSL VPN path traversal, Python tarfile extraction, Rails Accept-header file disclosure, runc container escape, HTTP/2 Rapid Reset, and PAN-OS GlobalProtect command injection. Verified 2026-11-12 against the NVD 2.0 API, the CISA KEV feed (catalog 2026.11.10), vendor advisories and public fix commits; excerpts are fix source only. Closed-source or unpublished fixes are marked. Numbered 3c so earlier section numbers do not change. No exploit code, payloads or reproduction steps appear anywhere in this section.
