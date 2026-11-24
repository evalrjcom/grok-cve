<!--
  P0F — Defensive CVE study notes.
  This file is for LEARNING vulnerability classes and remediation ONLY.
  It contains NO exploit code, NO payloads, and NO reproduction steps — by design.
  Code blocks show the *shape* of vulnerable vs fixed code taken from public patches,
  so you can recognize and fix the bug class, not weaponize it.
-->

# 3.11 Hard-Coded Credentials (CWE-798)

CWE: CWE-798 (https://cwe.mitre.org/data/definitions/798.html)
Root cause: Secrets (passwords, API keys, private keys) are embedded in source, images, or installers and shared across deployments.
Source → sink: Firmware/app binary/repo → hard-coded secret → attacker extracts and authenticates.
Code smells / review questions
- Grep: password=, api_key=, BEGIN RSA PRIVATE KEY in repos/images; default admin/admin.
- Smell: same key for all customers; undocumented static admin users created by plugins.
Fix pattern
- Per-deploy secrets from a vault/secret manager; force rotation on first boot; remove default accounts.
Detection ideas
- Secret scanning in CI; review installer/plugin account creation.
Real CVE lessons
- CVE-2024-20439 (Cisco Smart Licensing Utility): Undocumented static administrative credential; CWE-798; KEV 2025-03-31. — https://nvd.nist.gov/vuln/detail/CVE-2024-20439
- CVE-2022-26138 (Atlassian Questions for Confluence): Hardcoded disabledsystemuser password; KEV 2022-07-29. — https://nvd.nist.gov/vuln/detail/CVE-2022-26138
CVE walk — CVE-2020-8657 (EyesOfNetwork 5.3 — one hard-coded API key on every install; KEV)
Why it matters: A hard-coded secret means everyone who has the software has the key. NVD: the installation uses the same API key, hard-coded as EONAPI_KEY in include/api_functions.php, for all installations, letting an attacker calculate or guess the admin access token. CISA listed it in KEV on 2021-11-03 (ransomware use: Unknown).
Source → sink: Source = a constant shipped in source and identical across installs. Sink = apiKey() in include/api_functions.php, which derived the bearer token from that constant plus a value an outsider can learn.
Fix: Commit 0ed6d8cc901fac30357d438360a13aaee45eb3ba, "#17 APIKEY is now based on machine-id" (6 Feb 2020): the token derivation now mixes in the machine's /etc/machine-id instead of the server address.
Reviewer note: This is a partial mitigation: the shared constant is still the base of the derivation, and machine-id is a per-host value that may be readable elsewhere. Whether later releases removed the constant is unverified. The proper fix is a random per-install secret generated at setup and kept out of source control.
Scoring: NVD CVSS 3.1 9.8 (AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H); CISA-ADP also 9.8; NVD CVSS 2 only 5.0 (partial confidentiality) — the two NVD scores disagree sharply. CWE-798 (NVD and CISA-ADP). KEV 2021-11-03, ransomware Unknown.
- Closed-source KEV sibling: Confluence Questions CVE-2022-26138 (CWE-798, KEV 2022-07-29): the app created an account with a hard-coded password. Remediation per the vendor advisory is to update the app or disable/delete the account; no public diff. https://confluence.atlassian.com/doc/confluence-security-advisory-2022-07-20-1142446709.html
Detection: Secret scanning of source, images and firmware (gitleaks, trufflehog); grep for constants named *_KEY, *_SECRET, password; compare keys and default accounts across two fresh installs.
Remediation: Unique per-install secrets generated at first run, forced credential change on first login, no default accounts, secrets stored outside code, rotation on upgrade.
Takeaway: OSCP: default and shared credentials come first in enumeration. Bug bounty: prove two installs share the value with your own instances; never use a shared key against others' systems.
- Sources: https://nvd.nist.gov/vuln/detail/CVE-2020-8657 · https://github.com/EyesOfNetworkCommunity/eonapi/commit/0ed6d8cc901fac30357d438360a13aaee45eb3ba · KEV feed: https://www.cisa.gov/sites/default/files/feeds/known_exploited_vulnerabilities.json
OSCP / report angle
- Report the presence of a hard-coded secret and recommend rotation/removal. Do not reuse the secret against other tenants.

