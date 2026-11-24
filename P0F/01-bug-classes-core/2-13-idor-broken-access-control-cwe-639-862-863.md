<!--
  P0F — Defensive CVE study notes.
  This file is for LEARNING vulnerability classes and remediation ONLY.
  It contains NO exploit code, NO payloads, and NO reproduction steps — by design.
  Code blocks show the *shape* of vulnerable vs fixed code taken from public patches,
  so you can recognize and fix the bug class, not weaponize it.
-->

# 2.13 IDOR / Broken Access Control (CWE-639, 862, 863)

CWE: CWE-639 (https://cwe.mitre.org/data/definitions/639.html) · CWE-862 (https://cwe.mitre.org/data/definitions/862.html) · CWE-863 (https://cwe.mitre.org/data/definitions/863.html)
Root cause: Client-supplied object reference used without verifying the caller may access that object.
Source → sink: objectId in URL/body → query by id → response/action without ownership/tenant check.
Code smells / review questions
- Grep: findById(param), TODO authz, missing WHERE owner_id = session.user.
- For every client object id, is there a tenant/owner/role check on the same query?
- Smell: OrgID from request trusted (Grafana); inconsistent create/read/update/delete checks.
Fix pattern
- Server-side authz on every object access; derive tenant from session; ID-swap automated tests.
Detection ideas
- Replace IDs between two low-priv accounts; review GraphQL resolvers and batch endpoints.
Real CVE lessons
- CVE-2024-1313 (Grafana): OrgID check left as TODO enabled (809c1ea); not KEV. Tenant-scoped queries. — https://nvd.nist.gov/vuln/detail/CVE-2024-1313
OSCP / report angle
- Two-account demo of unauthorized object access; no mass data download.

