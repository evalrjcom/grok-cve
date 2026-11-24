<!--
  P0F — Defensive CVE study notes.
  This file is for LEARNING vulnerability classes and remediation ONLY.
  It contains NO exploit code, NO payloads, and NO reproduction steps — by design.
  Code blocks show the *shape* of vulnerable vs fixed code taken from public patches,
  so you can recognize and fix the bug class, not weaponize it.
-->

# 3.1 Open Redirect (CWE-601)

CWE: CWE-601 (https://cwe.mitre.org/data/definitions/601.html)
Root cause: Application redirects the browser to an attacker-controlled URL using unvalidated user input.
Source → sink: next/returnUrl/redirect param → Location header / meta refresh / JS redirect → external site (phishing/token theft).
Code smells / review questions
- Grep: redirect(, Location:, returnUrl, next=, continue=, RelayState.
- Is the target allow-listed to same-site absolute paths only?
- Smell: a scheme-relative URL that points at an outside host; parsing that treats backslash as path on some platforms.
Fix pattern
- Allow-list destinations; allow only relative paths starting with / but not //; reject absolute external URLs.
Detection ideas
- Hunt auth/login/logout/SSO callback redirect parameters.
Real CVE lessons
- CVE-2021-38000 (Google Chrome (Android Intents)): Untrusted Intent input could open a malicious URL; CWE-601; KEV 2021-11-03. — https://nvd.nist.gov/vuln/detail/CVE-2021-38000
- CVE-2020-11053 (OAuth2 Proxy < 5.1.1): Auth-flow redirect address not validated (CWE-601); not KEV. — https://nvd.nist.gov/vuln/detail/CVE-2020-11053
CVE walk — CVE-2020-11053 (OAuth2 Proxy < 5.1.1 — redirect validator vs browser URL parsing)
Why it matters: An authentication proxy sends the user back to the page they asked for after login. If that return target is not constrained, a trusted login domain becomes a launch pad for phishing, and on OAuth-style flows a wrong redirect target can also send codes or tokens to another host. Alone it is usually a low-severity issue; its value is in the chain.
Source → sink: Source = the redirect target supplied with the login request. Sink = the HTTP redirect issued when authentication finishes. Chokepoint = one validator function, IsValidRedirect() in oauthproxy.go.
Root cause (conceptual): The validator accepted any value that started with a single slash and did not begin with two slashes or a slash-backslash pair. Browsers normalise some whitespace and control characters before parsing a URL, so a string the server judged to be a local path could be read by the browser as an external, scheme-relative address. A parser differential between the checker and the consumer.
Fix: Commit 0d5fa211df8ef2449347a56b22c779eb8d894c43 (merge of the GHSA-j7px-6hwj-hpjg private fix, 6 May 2020), shipped in 5.1.1: a deny regexp is applied to the redirect in addition to the old prefix checks. The regex literal and the test cases are intentionally not reproduced here.
Scoring: NVD CVSS 3.1 6.1 (AV:N/AC:L/PR:N/UI:R/S:C/C:L/I:L/A:N); GitHub (CNA) CVSS 3.1 7.1 — one point higher, same family of vector; CVSS 2 5.8. CWE-601 on both. Not in KEV.
Detection: Grep for redirect-target parameters (rd, next, returnUrl, RelayState, continue) and every place a response Location is built from request data; SCA for oauth2-proxy < 5.1.1.
Remediation: Upgrade to ≥ 5.1.1. Better design: allow-list exact paths or hosts, parse with the platform URL parser and compare scheme and host on the parsed result; never validate with string prefixes.
Takeaway: Validate the representation the browser will act on, not the raw string. Bug bounty: a bare open redirect is often low or informational; it earns severity when chained to OAuth code/token leakage or login phishing on a trusted domain. Show the validator logic and a benign redirect to a domain you control.
- Sources: https://nvd.nist.gov/vuln/detail/CVE-2020-11053 · https://github.com/oauth2-proxy/oauth2-proxy/security/advisories/GHSA-j7px-6hwj-hpjg · https://github.com/oauth2-proxy/oauth2-proxy/commit/0d5fa211df8ef2449347a56b22c779eb8d894c43
OSCP / report angle
- Show the parameter and unconstrained Location. Recommend allow-list. Do not phish real users.

