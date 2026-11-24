<!--
  P0F — Defensive CVE study notes.
  This file is for LEARNING vulnerability classes and remediation ONLY.
  It contains NO exploit code, NO payloads, and NO reproduction steps — by design.
  Code blocks show the *shape* of vulnerable vs fixed code taken from public patches,
  so you can recognize and fix the bug class, not weaponize it.
-->

# 2b.8 Weak Password Reset (CWE-640)

CWE: CWE-640 (https://cwe.mitre.org/data/definitions/640.html)
Root cause: The reset flow lets an attacker choose or influence the destination of the reset token, or matches the victim account using a weak identifier comparison, so the token is delivered to the attacker rather than to the address on file.
Source → sink: Reset request (email or identifier supplied by the client) → lookup and token issue → message is sent to a client-controlled or weakly compared address instead of the verified address on the account.
Code smells / review questions
- Is the token always mailed only to verified address(es) stored on the user, never to the raw request field? Is the identifier comparison the Unicode recommended case-fold, not a database-only lowercasing?
- Smell: request parameter used as the To: header; accepting a list of recipients; host-header used in the reset URL; no proof of email ownership.
Fix pattern
- Send the token only to verified addresses on the account; compare identifiers with a well-defined Unicode case-fold (UTR 36) in the application, not only in SQL; do not honour a client-supplied recipient list; expire tokens quickly and rate-limit resets.
Detection ideas
- Review the reset controller for a To: field taken from the request; confirm the mailer uses the stored, verified email. Check GitLab / Django / similar framework versions against the advisories below.
Real CVE lessons
- CVE-2023-7028 (GitLab CE/EE 16.1 before the listed 16.x patches): reset mail could be delivered to an unverified address. NVD CVSS 3.1 9.8 vs GitLab CNA 10.0. NVD/CNA CWE-640; KEV lists CWE-284. KEV 2024-05-01. Walked in §3d. -- https://nvd.nist.gov/vuln/detail/CVE-2023-7028
- Related (code-level Unicode comparison): Django CVE-2019-19844, NVD 9.8, CWE-640, fixed in 1.11.27 / 2.2.9 / 3.0.1, commit 5b1fbcef7a8bec991ebe7b2a18b5d5a95d72cb70. Token is sent to the address on record after a UTR 36 comparison. -- https://nvd.nist.gov/vuln/detail/CVE-2019-19844
OSCP / report angle
- Show that the mailer uses a request-supplied recipient or a weak comparison. Do not request a reset against someone else's account; use only accounts you own.
