<!--
  P0F — Defensive CVE study notes.
  This file is for LEARNING vulnerability classes and remediation ONLY.
  It contains NO exploit code, NO payloads, and NO reproduction steps — by design.
  Code blocks show the *shape* of vulnerable vs fixed code taken from public patches,
  so you can recognize and fix the bug class, not weaponize it.
-->

# 2.18 JWT / Crypto Verification (CWE-347, 327)

CWE: CWE-347 (https://cwe.mitre.org/data/definitions/347.html) · CWE-327 (https://cwe.mitre.org/data/definitions/327.html)
Root cause: Signature/MAC not verified correctly, or algorithm confusion lets attacker choose a weak/none/public-key-as-HMAC path.
Source → sink: Token alg/header → verifier that trusts alg or uses deny-list → accept forged token.
Code smells / review questions
- Grep: jwt.decode without algorithms=[...], verify=False, HS256 with public key as secret.
- Is allowed algorithm an allow-list of one family? Are kid/jku attacker URLs?
- Smell: deny-list of PEM headers missing a format (PyJWT ssh-ed25519); accepting alg=none.
Fix pattern
- Pin algorithms=["RS256"]; current libraries; validate iss/aud/exp; explicit key config.
Vulnerable shape (educational)

Fixed shape

Detection ideas
- Unit tests: alg confusion, none alg, wrong aud must fail closed.
Real CVE lessons
- CVE-2022-29217 (PyJWT): Deny-list missed ssh-ed25519; 9c52867 is_pem_format/is_ssh_key; 2.4.0. Pin algorithms=[...]. — https://nvd.nist.gov/vuln/detail/CVE-2022-29217
OSCP / report angle
- Report verifier gap and allow-list fix. Never include a forged production token.

