<!--
  P0F — Defensive CVE study notes.
  This file is for LEARNING vulnerability classes and remediation ONLY.
  It contains NO exploit code, NO payloads, and NO reproduction steps — by design.
  Code blocks show the *shape* of vulnerable vs fixed code taken from public patches,
  so you can recognize and fix the bug class, not weaponize it.
-->

# 3.13 Information Exposure (CWE-200)

CWE: CWE-200 (https://cwe.mitre.org/data/definitions/200.html)
Root cause: Sensitive data (secrets, PII, internal files, stack traces) is disclosed to unauthorized actors.
Source → sink: Error/debug/feature endpoint → overly detailed response or file read → confidential data leaves trust boundary.
Code smells / review questions
- Are stack traces, env dumps, or internal paths returned to clients?
- Smell: debug mode in prod; verbose auth errors; features that fetch local files for "maps"/"templates".
- Smell: directory listings; backup files under web root.
Fix pattern
- Generic client errors; no debug in prod; strict file allow-lists; classify data and apply need-to-know.
Detection ideas
- Review error handlers and admin import/map features for local file reads.
Real CVE lessons
- CVE-2021-41277 (Metabase): Custom GeoJSON map feature enabled local file inclusion / sensitive exposure (CWE-200, CWE-22); KEV 2024-11-12. — https://nvd.nist.gov/vuln/detail/CVE-2021-41277
CVE walk — CVE-2021-41277 (Metabase < 0.40.5 / 1.40.5 — GeoJSON URL not validated; KEV)
Why it matters: An admin feature that fetches a location on the server's behalf and returns the content becomes a file-read and information-exposure primitive if the location is not validated. NVD: custom GeoJSON map support allowed local file inclusion, including environment variables, because URLs were not validated before loading. CISA listed it in KEV on 2024-11-12 (ransomware use: Unknown), three years after disclosure.
Source → sink: Source = the URL given for a custom GeoJSON map. Sink = the server loading that location (classpath/local resources as well as URLs) in src/metabase/api/geojson.clj and returning it. Two entry points used two different inline checks.
Fix: Commit 042a36e49574c749f944e19cf80360fd3dc322f0, "GeoJSON URL validation fix (#17990)" (21 Sep 2021), in 0.40.5 and 1.40.5: one shared validator for both entry points, plus a superuser requirement on the proxy endpoint.
Scoring: NVD CVSS 3.1 7.5 (AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N); CVSS 2 5.0. GitHub (CNA) CVSS 3.1 10.0 — a 2.5-point gap. CWE also differs: CNA CWE-200, NVD CWE-22. KEV 2024-11-12, ransomware Unknown.
Detection: SCA for Metabase < 0.40.5 / 1.40.5; review every "fetch this URL / read this file" feature for authentication, scheme allow-list and a single shared validator.
Remediation: Upgrade; restrict egress and file access of the service account; keep secrets out of the process environment where possible; allow-list schemes and hosts.
Takeaway: One validator at every entry point; admin-only is a control, not an excuse for no validation. Bug bounty: prove with a non-sensitive file on your own instance and stop; never read another tenant's secrets.
- Sources: https://nvd.nist.gov/vuln/detail/CVE-2021-41277 · https://github.com/metabase/metabase/security/advisories/GHSA-w73v-6p7p-fpfr · https://github.com/metabase/metabase/commit/042a36e49574c749f944e19cf80360fd3dc322f0 · KEV feed: https://www.cisa.gov/sites/default/files/feeds/known_exploited_vulnerabilities.json
OSCP / report angle
- Show the disclosing endpoint and the data class exposed. Redact secrets in the report itself.

