<!--
  P0F — Defensive CVE study notes.
  This file is for LEARNING vulnerability classes and remediation ONLY.
  It contains NO exploit code, NO payloads, and NO reproduction steps — by design.
  Code blocks show the *shape* of vulnerable vs fixed code taken from public patches,
  so you can recognize and fix the bug class, not weaponize it.
-->

# 3.4 HTTP Request Smuggling (CWE-444)

CWE: CWE-444 (https://cwe.mitre.org/data/definitions/444.html)
Root cause: Front-end and back-end disagree on request framing (Content-Length vs Transfer-Encoding), so one request is parsed as two (or vice versa).
Source → sink: Ambiguous HTTP/1.1 framing → proxy desync → request concatenated/tunnelled into another user's request or internal route.
Code smells / review questions
- Review proxy/CDN/app server CL/TE handling; reject dual CL+TE; normalize HTTP/1.1 hops.
- Smell: RewriteRule/ProxyPassMatch with mod_proxy (httpd 25690 class); concatenation bugs (SAP 22536).
- Prefer HTTP/2 end-to-end where practical to remove CL/TE ambiguity.
Fix pattern
- Single framing authority; drop TE on inbound HTTP/1.1 or reject ambiguous messages; keep proxy and origin versions patched.
Detection ideas
- Inventory reverse-proxy chains; regression-test after proxy upgrades.
Real CVE lessons
- CVE-2022-22536 (SAP NetWeaver / Web Dispatcher): Request smuggling and concatenation; KEV 2022-08-18. — https://nvd.nist.gov/vuln/detail/CVE-2022-22536
- CVE-2023-25690 (Apache HTTP Server mod_proxy): Certain RewriteRule/ProxyPassMatch configs enable smuggling; 2.4.0–2.4.55; not KEV. — https://nvd.nist.gov/vuln/detail/CVE-2023-25690
CVE walk — CVE-2026-48710 (Starlette < 1.0.1 — Host header makes request.url disagree with the routed path; KEV)
Why it matters: Request desync (CWE-444) in the broad sense: two components derive different "paths" from the same bytes. Starlette routes on the raw ASGI path but rebuilds request.url from the Host header, so middleware that makes access decisions from request.url.path can look at a different path than the one the router serves. CISA added it to KEV on 2026-09-02 (ransomware use: Unknown), so it is exploited in the wild. Note: this is a URL/path desync, not classic Content-Length/Transfer-Encoding smuggling; the classic form is the HAProxy comparator below.
Source → sink: Source = the unvalidated Host header. Sink = URL construction in starlette/datastructures.py; consumers = auth or ACL middleware reading request.url.path while routing uses scope["path"].
Fix: Commit 764dab0dcfb9033d75442d7a359645c9f94648c6, "Ignore malformed Host header when constructing request.url" (21 May 2026), released in 1.0.1: the Host header is used only if it matches a strict host[:port] allow-list; otherwise the server tuple is used.
Scoring: NVD CVSS 3.1 6.5 (AV:N/AC:L/PR:N/UI:N/S:U/C:L/I:L/A:N); GitHub (CNA) and Red Hat (SADP) also 6.5. CWE differs: GitHub says CWE-444, Red Hat says CWE-1289; the NVD record shows no CWE of its own. KEV: yes, dateAdded 2026-09-02, ransomware Unknown.
- Classic comparator: HAProxy CVE-2021-40346 (NVD 7.5, CWE-190, not KEV): a missing header-name length check in htx_add_header()/htx_add_trailer() let header-name length bits slip into the value length, which could forge a content-length header after de-duplication and desync the proxy from the back end. Fix: https://github.com/haproxy/haproxy/commit/3b69886f7dcc3cfb3d166309018e6cfec9ce2c95 (the commit message gives a mitigation rule: deny messages with more than one content-length).
Detection: SCA for starlette < 1.0.1 and frameworks built on it; review middleware that decides from request.url instead of the routed path; at the edge, reject or normalise malformed Host headers.
Remediation: Upgrade to 1.0.1; use a trusted-host allow-list; make routing and authorisation read one canonical path value.
Takeaway: Any decision made on a value derived differently from the one used for routing is a desync. Bug bounty: show two layers disagreeing (what was routed vs what the check saw) with a benign request; do not touch protected data. OSCP: classic CL/TE smuggling needs a front-end/back-end pair — describe the parser disagreement, not an attack.
- Sources: https://nvd.nist.gov/vuln/detail/CVE-2026-48710 · https://github.com/Kludex/starlette/security/advisories/GHSA-86qp-5c8j-p5mr · https://github.com/advisories/GHSA-86qp-5c8j-p5mr · https://github.com/Kludex/starlette/commit/764dab0dcfb9033d75442d7a359645c9f94648c6 · https://github.com/Kludex/starlette/releases/tag/1.0.1 · https://ostif.org/disclosing-the-badhost-vulnerability-in-starlette · KEV feed: https://www.cisa.gov/sites/default/files/feeds/known_exploited_vulnerabilities.json
OSCP / report angle
- Describe the desync condition and the config/version fix. Do not include smuggling byte sequences in reports when policy asks for non-weaponized writeups — cite the advisory and affected config.

