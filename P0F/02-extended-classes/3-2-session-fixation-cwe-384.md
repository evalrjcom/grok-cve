<!--
  P0F — Defensive CVE study notes.
  This file is for LEARNING vulnerability classes and remediation ONLY.
  It contains NO exploit code, NO payloads, and NO reproduction steps — by design.
  Code blocks show the *shape* of vulnerable vs fixed code taken from public patches,
  so you can recognize and fix the bug class, not weaponize it.
-->

# 3.2 Session Fixation (CWE-384)

CWE: CWE-384 (https://cwe.mitre.org/data/definitions/384.html)
Root cause: Attacker sets (or predicts) a session identifier that the victim then authenticates into, so the attacker inherits the session.
Source → sink: Attacker-chosen session id accepted → victim logs in → same id becomes authenticated → attacker reuses it.
Code smells / review questions
- Does login regenerate the session id? Are session ids accepted from URL/query?
- Smell: session id in URL; no rotate-on-login; long-lived pre-auth cookies reused post-auth.
Fix pattern
- Always issue a new session id on privilege change (login/logout/role elev); HttpOnly Secure SameSite cookies; avoid URL session ids.
Detection ideas
- Review login/SSO handlers for session.regenerate / rotate calls.
Real CVE lessons
- CVE-2026-102489 (Zammad 6.3.0–6.5.4): NVD maps to CWE-384 (session-related); KEV 2026-10-02. Treat as a session-handling critical — confirm with the advisory/fix whether classic fixation or hijack. — https://nvd.nist.gov/vuln/detail/CVE-2026-102489
CVE walk — CVE-2022-25896 (Passport < 0.6.0 — session not regenerated on login)
Why it matters: Passport is the most widely used Node.js authentication middleware. If the pre-login session survives authentication, an identifier an attacker planted or learned before login becomes an authenticated session after it: classic session fixation (CWE-384).
Source → sink: Source = the session identifier (cookie) the browser already holds before login. Sink = SessionManager.logIn writing the authenticated user into that same session object. Chokepoint = this one function, used by every Passport strategy.
Fix: Commit 7e9b9cf4d7be02428e963fc729496a45baeea608, "Regenerate session on login." (PR #900), shipped in 0.6.0. The commit also adds two console.log debug lines; they are omitted below.
Scoring: NVD CVSS 3.1 4.8 (AV:N/AC:H/PR:N/UI:N/S:U/C:L/I:L/A:N); Snyk (CNA) also 4.8 but with a different vector (C:L/I:N/A:L); CVSS 2 5.8. CWE-384. Not in KEV. The one-line CVE text ("regenerated instead of being closed") is confusingly worded — trust the commit message and diff over it.
- Not used as the walk: Zammad CVE-2026-102489 (KEV 2026-10-02, kept as the lesson above) is described as a session hijack and I found no public fix diff to excerpt, so it is not shown as a fixation example.
Detection: Grep login handlers for session regeneration or id rotation; npm ls passport; compare the session cookie value before and after login in a test account.
Remediation: Upgrade to ≥ 0.6.0. Anything stored in the pre-login session is dropped by design, so re-set it after login. Also rotate on logout and on privilege change, and set HttpOnly, Secure and SameSite on the cookie.
Takeaway: Rotate the session on every privilege change. Bug bounty: show the same identifier before and after authenticating in your own accounts; severity depends on whether an attacker can set or learn the cookie (subdomain, XSS, id in URL), so state that precondition.
- Sources: https://nvd.nist.gov/vuln/detail/CVE-2022-25896 · https://github.com/advisories/GHSA-v923-w3x8-wh69 · https://github.com/jaredhanson/passport/commit/7e9b9cf4d7be02428e963fc729496a45baeea608 · https://github.com/jaredhanson/passport/pull/900 · https://github.com/jaredhanson/passport/releases/tag/v0.6.0
OSCP / report angle
- Report missing session rotation on login. Classic fixation demos need program approval; describe the missing regenerate call.

