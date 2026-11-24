<!--
  P0F — Defensive CVE study notes.
  This file is for LEARNING vulnerability classes and remediation ONLY.
  It contains NO exploit code, NO payloads, and NO reproduction steps — by design.
  Code blocks show the *shape* of vulnerable vs fixed code taken from public patches,
  so you can recognize and fix the bug class, not weaponize it.
-->

# 3.10 Clickjacking / UI Redress (CWE-1021)

CWE: CWE-1021 (https://cwe.mitre.org/data/definitions/1021.html)
Root cause: Sensitive UI is embedded in a foreign frame/overlay so the victim clicks actions they did not intend.
Source → sink: Target page framable → attacker overlay → victim click activates privileged control.
Code smells / review questions
- Is X-Frame-Options or CSP frame-ancestors set on sensitive pages?
- Smell: missing headers on admin/settings/consent pages; ALLOWALL framing.
Fix pattern
- CSP frame-ancestors 'self' (or explicit list); X-Frame-Options: DENY/SAMEORIGIN; UI confirmations for high-risk actions.
Detection ideas
- Check admin, password-change, OAuth consent, payment confirmation pages.
Real CVE lessons
- CVE-2019-3794 (Cloud Foundry UAA < v73.4.0): Missing X-FRAME-OPTIONS on various endpoints; CWE-1021; not KEV. — https://nvd.nist.gov/vuln/detail/CVE-2019-3794
CVE walk — CVE-2020-24711 (Gophish < 0.11.0 — admin UI could be framed)
Why it matters: Clickjacking means another origin can embed the application in a frame and trick the user into clicking real controls. Impact equals what one click does: here the Reset button on Account Settings, which NVD describes as a denial-of-service.
Source → sink: Source = a page on another origin framing the admin UI. Sink = a state-changing control reachable with a single click. Missing control = a response header telling the browser not to render the page in a frame.
Fix: Commit 6df62e85fd60f0931d3c8bfdb13b436a961bc9b6, "Added a simple Content-Security-Policy to mitigate clickjacking attempts." (20 Aug 2020), shipped in v0.11.0: a middleware applied to all admin routes sets frame-ancestors and X-Frame-Options.
Scoring: NVD CVSS 3.1 6.5 (AV:N/AC:L/PR:N/UI:R/S:U/C:N/I:N/A:H); CVSS 2 4.3; CWE-1021 (NVD); no CNA score. Not in KEV. NVD also lists a researcher advisory tagged Exploit; I deliberately did not link it.
- Comparator: Cloud Foundry UAA CVE-2019-3794 (NVD CVSS 3.1 5.4 vs CNA CVSS 3.0 6.5; CNA CWE-284 vs NVD CWE-1021; fixed v73.4.0). Fix commit not located, so no excerpt.
Detection: Check response headers (X-Frame-Options or CSP frame-ancestors) on login, settings and every state-changing page, ideally via a global middleware or proxy rule.
Remediation: Set the headers globally (default deny), allow-list only legitimate embedders; SameSite cookies reduce cross-site framing impact; JavaScript frame-busting is not a defence.
Takeaway: Default-deny framing. Bug bounty: many programs exclude clickjacking on pages without a sensitive one-click action — read scope and report only where a state-changing action or auth page is frameable.
- Sources: https://nvd.nist.gov/vuln/detail/CVE-2020-24711 · https://github.com/gophish/gophish/commit/6df62e85fd60f0931d3c8bfdb13b436a961bc9b6 · https://github.com/gophish/gophish/releases/tag/v0.11.0
OSCP / report angle
- Show missing frame-ancestors / XFO. Recommend CSP. Do not trick real users into clicking.

