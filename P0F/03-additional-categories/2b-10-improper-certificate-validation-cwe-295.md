<!--
  P0F — Defensive CVE study notes.
  This file is for LEARNING vulnerability classes and remediation ONLY.
  It contains NO exploit code, NO payloads, and NO reproduction steps — by design.
  Code blocks show the *shape* of vulnerable vs fixed code taken from public patches,
  so you can recognize and fix the bug class, not weaponize it.
-->

# 2b.10 Improper Certificate Validation (CWE-295)

CWE: CWE-295 (https://cwe.mitre.org/data/definitions/295.html)
Root cause: The product does not correctly verify a certificate (hostname, chain, purpose, or cryptographic group parameters), so a crafted or substituted cert can pass as a trusted issuer or peer.
Source → sink: TLS handshake or code-signing check → validator skips a required comparison (name, chain, or ECC group parameters) → a spoofed cert is treated as trusted.
Code smells / review questions
- Does the validator pin or at least fully check hostname, chain, validity, purpose, and (for ECC) the curve/group parameters against a known-good set?
- Smell: verification disabled "temporarily"; custom ECC parameters accepted; accepting a chain because a leaf parses, without a trusted-root path.
Fix pattern
- Use the platform TLS stack with verification on; reject unknown ECC parameters and require a named, well-known group; check hostname and purpose; pin where the threat model needs it.
Detection ideas
- Inventory TLS clients with verification disabled or custom cert parsing; on Windows, confirm the CryptoAPI update for CVE-2020-0601 is installed.
Real CVE lessons
- CVE-2020-0601 (Windows CryptoAPI, "CurveBall"): ECC certificates were not checked against a known-good group, allowing a spoofed code-signing or TLS cert. NVD CVSS 3.1 8.1. CWE-295. KEV 2021-11-03. Walked in §3d. -- https://nvd.nist.gov/vuln/detail/CVE-2020-0601
OSCP / report angle
- Report a skipped hostname, chain, or group-parameter check and the vendor fix. Do not include a crafted certificate or a proof-of-spoof chain in the report.
