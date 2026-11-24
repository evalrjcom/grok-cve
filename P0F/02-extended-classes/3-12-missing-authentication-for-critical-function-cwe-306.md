<!--
  P0F — Defensive CVE study notes.
  This file is for LEARNING vulnerability classes and remediation ONLY.
  It contains NO exploit code, NO payloads, and NO reproduction steps — by design.
  Code blocks show the *shape* of vulnerable vs fixed code taken from public patches,
  so you can recognize and fix the bug class, not weaponize it.
-->

# 3.12 Missing Authentication for Critical Function (CWE-306)

CWE: CWE-306 (https://cwe.mitre.org/data/definitions/306.html)
Root cause: A sensitive operation is reachable with no authentication check at all (stronger than "wrong check" — the check is absent).
Source → sink: Request to critical endpoint → no authn middleware → privileged action executes.
Code smells / review questions
- Is the experimental/legacy/setup API still exposed? Default-allow vs default-deny?
- Smell: "temporary" unauthenticated admin wizards left on (SAP LM Configuration Wizard class).
- Smell: management ports assumed to be network-isolated but reachable.
Fix pattern
- Deny-by-default authn on all routes; disable setup endpoints after install; network-isolate management planes.
Detection ideas
- Enumerate all routes unauthenticated; pay special attention to /api/experimental, /setup, /wizard.
Real CVE lessons
- CVE-2020-6287 (SAP NetWeaver AS JAVA LM Config Wizard): No authentication check on critical configuration tasks; KEV 2021-11-03. — https://nvd.nist.gov/vuln/detail/CVE-2020-6287
- CVE-2020-13927 (Apache Airflow Experimental API): Default allowed unauthenticated API; default changed to deny in 1.10.11; KEV 2022-01-18. — https://nvd.nist.gov/vuln/detail/CVE-2020-13927
CVE walk — CVE-2020-13927 (Apache Airflow < 1.10.11 — experimental API open by default; KEV)
Why it matters: Missing authentication on a critical function (CWE-306). Airflow's Experimental REST API shipped with an auth backend that accepted every request, so anyone who could reach the web server could use an API that controls a workflow engine. CISA listed it in KEV on 2022-01-18 (ransomware use: Unknown).
Source → sink: Source = any network client. Sink = the Experimental API endpoints. Missing control = the [api] auth_backend setting, whose default was the allow-all backend.
Fix: Commit 9e305d6b810a2a21e2591a80a80ec41acb3afed0, "Change default auth for experimental backend to deny_all" (1 Jul 2020; PR #9611), released in 1.10.11. It changes the default for new installs only; existing installs keep their airflow.cfg and must set the deny-all backend explicitly. The commit is not an NVD reference (NVD points to the Apache dev-list thread); I found it via the git history and PR #9611.
Scoring: NVD CVSS 3.1 9.8 (AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H) = CISA-ADP 9.8; CVSS 2 7.5. CWE: NVD CWE-306 and CWE-1188; CISA-ADP adds CWE-1056; the KEV record lists CWE-1188 and CWE-306. KEV 2022-01-18, ransomware Unknown.
- Closed-source KEV sibling: SAP CVE-2020-6287 (unauthenticated configuration wizard); remediation is the SAP patch, no public diff.
Detection: Unauthenticated requests against every management or experimental route in a lab; review framework defaults for "allow all" backends; SCA for Airflow < 1.10.11.
Remediation: Upgrade; on existing installs set [api] auth_backend = airflow.api.auth.backend.deny_all (or a real authentication backend); never expose the web server to untrusted networks without auth.
Takeaway: Secure by default — and flipping a default does not fix existing installs. Bug bounty: unauthenticated access to a management API is high severity, but prove it with a read-only endpoint; never trigger jobs.
- Sources: https://nvd.nist.gov/vuln/detail/CVE-2020-13927 · https://github.com/apache/airflow/commit/9e305d6b810a2a21e2591a80a80ec41acb3afed0 · https://github.com/apache/airflow/pull/9611 · https://lists.apache.org/thread.html/r23a81b247aa346ff193670be565b2b8ea4b17ddbc7a35fc099c1aadd%40%3Cdev.airflow.apache.org%3E · KEV feed: https://www.cisa.gov/sites/default/files/feeds/known_exploited_vulnerabilities.json
OSCP / report angle
- Show the unauthenticated critical sink. Recommend deny-by-default. Related: BIG-IP CVE-2022-1388 also in §2.7.

