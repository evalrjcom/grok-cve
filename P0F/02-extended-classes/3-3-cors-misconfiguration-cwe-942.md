<!--
  P0F — Defensive CVE study notes.
  This file is for LEARNING vulnerability classes and remediation ONLY.
  It contains NO exploit code, NO payloads, and NO reproduction steps — by design.
  Code blocks show the *shape* of vulnerable vs fixed code taken from public patches,
  so you can recognize and fix the bug class, not weaponize it.
-->

# 3.3 CORS Misconfiguration (CWE-942)

CWE: CWE-942 (https://cwe.mitre.org/data/definitions/942.html)
Root cause: Access-Control-Allow-Origin (and related headers) reflect or overly trust origins, exposing authenticated responses to foreign sites.
Source → sink: Origin request header → ACAO reflects Origin / null / * with credentials → foreign JS reads response.
Code smells / review questions
- Grep: Access-Control-Allow-Origin, CORS middleware with origin:true / reflecting Origin.
- Is ACAO a fixed allow-list? Are credentials allowed with * or null?
- Smell: reflecting any Origin; trusting null; CORS on endpoints that also set CSRF tokens (token theft).
Fix pattern
- Explicit origin allow-list; never * with credentials; reject null; prefer CSRF tokens + SameSite over CORS sprawl.
Detection ideas
- Check every API that sets ACAO; especially local/RPC services reachable from browsers.
Real CVE lessons
- CVE-2026-116748 (Transmission 4.0.0–<4.1.3): Permissive CORS in rpc-server.cc exposed X-Transmission-Session-Id anti-CSRF token to any origin; CWE-942; not KEV. — https://nvd.nist.gov/vuln/detail/CVE-2026-116748
CVE walk — CVE-2026-116748 (Transmission 4.0.0–4.1.2 — permissive CORS exposes the RPC anti-CSRF token)
Why it matters: A local service (the BitTorrent client's RPC endpoint) that sends permissive CORS headers lets any web page the user visits read its responses. Here that included the session-id token that is meant to be the CSRF defence, so a page could then drive the RPC API, which NVD says enables script execution on torrent completion, download-directory changes or deletion.
Source → sink: Source = any cross-origin web page. Sink = handle_request() in libtransmission/rpc-server.cc, which added an allow-all origin header and exposed the token header on the handshake response.
Fix: Commit c6a1bd2017a0fea9c956737c213f23e4b0f7929e, "fix: don't expose RPC session id via CORS (#8938)" (25 Jun 2026), released in 4.1.3: the CORS headers are removed, and a startup warning appears when both password and whitelist are off.
Scoring: NVD status is Awaiting Analysis, so there is no NVD score yet. VulnCheck (the CNA) scores CVSS 4.0 8.7 and CVSS 3.1 8.8 (AV:N/AC:L/PR:N/UI:R/S:U/C:H/I:H/A:H); CWE-942. CNA-only scores can change once NVD analyses the record. Not in KEV.
Detection: Look at response headers on local admin and RPC ports for Access-Control-Allow-Origin and Access-Control-Expose-Headers; check whether any token-bearing response is readable cross-origin.
Remediation: Upgrade to ≥ 4.1.3; keep the RPC whitelist or authentication enabled and bind to loopback. For your own services: no wildcard origin on responses that carry tokens, and validate Origin and Host for local services (DNS rebinding).
Takeaway: A CSRF token is worthless if CORS lets other origins read it. Bug bounty: show the header pair and that the token is readable from a page you control; do not run the RPC actions.
- Sources: https://nvd.nist.gov/vuln/detail/CVE-2026-116748 · https://www.vulncheck.com/advisories/transmission-4.0.0-before-4.1.3-permissive-cors-leaks-rpc-anti-csrf-token · https://github.com/transmission/transmission/commit/c6a1bd2017a0fea9c956737c213f23e4b0f7929e · https://github.com/transmission/transmission/pull/8938 · https://github.com/transmission/transmission/releases/tag/4.1.3
OSCP / report angle
- Show Origin reflection and credentialed response readability. Recommend allow-list. No live cross-origin token theft against out-of-scope users.

