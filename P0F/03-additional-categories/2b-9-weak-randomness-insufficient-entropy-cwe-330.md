<!--
  P0F — Defensive CVE study notes.
  This file is for LEARNING vulnerability classes and remediation ONLY.
  It contains NO exploit code, NO payloads, and NO reproduction steps — by design.
  Code blocks show the *shape* of vulnerable vs fixed code taken from public patches,
  so you can recognize and fix the bug class, not weaponize it.
-->

# 2b.9 Weak Randomness / Insufficient Entropy (CWE-330)

CWE: CWE-330 (https://cwe.mitre.org/data/definitions/330.html)
Root cause: A security-critical value (session key, token, nonce, or credential) is generated or accepted with too little entropy, so it can be predicted, brute-forced, or replaced by a degenerate value.
Source → sink: Authentication or key-exchange protocol → a client- or server-chosen secret with a tiny effective space → the peer accepts it as a valid session or credential.
Code smells / review questions
- Is every nonce, session key and reset token drawn from a CSPRNG with a large enough length? Does the protocol reject all-zero or otherwise degenerate contributions from the client?
- Smell: weak language RNGs or time-based IDs for tokens; accepting a client session key without a uniqueness or entropy check; truncated keys.
Fix pattern
- Use a CSPRNG; fix the length and encoding; reject degenerate (including all-zero) protocol values; do not let a client choose a session key alone.
Detection ideas
- Review token and key generation; on Windows AD, confirm the Netlogon secure-channel enforcement updates are applied (see Zerologon below).
Real CVE lessons
- CVE-2020-1472 (Windows Netlogon, "Zerologon"): an unauthenticated client could establish a Netlogon secure channel to a domain controller because the protocol accepted a client credential with insufficient entropy. NVD CVSS 3.1 10.0 vs Microsoft CNA 5.5. NVD primary CWE is NVD-CWE-noinfo; KEV lists CWE-330. KEV 2021-11-03, ransomware Known. Walked in §3d. -- https://nvd.nist.gov/vuln/detail/CVE-2020-1472
OSCP / report angle
- For this class, record the protocol, the missing entropy check, and the official patch/enforcement state. Do not include cryptographic attack details or test it against a domain you do not own.
