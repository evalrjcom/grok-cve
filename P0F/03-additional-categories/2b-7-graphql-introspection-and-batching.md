<!--
  P0F — Defensive CVE study notes.
  This file is for LEARNING vulnerability classes and remediation ONLY.
  It contains NO exploit code, NO payloads, and NO reproduction steps — by design.
  Code blocks show the *shape* of vulnerable vs fixed code taken from public patches,
  so you can recognize and fix the bug class, not weaponize it.
-->

# 2b.7 GraphQL Introspection and Batching

CWE: CWE-200 (https://cwe.mitre.org/data/definitions/200.html) · CWE-770 (https://cwe.mitre.org/data/definitions/770.html)
Root cause: A GraphQL endpoint exposes the schema (introspection) to unauthenticated or low-privilege clients, or accepts unbounded aliases/batches, so an attacker can map the API or exhaust resolver resources. This is a GraphQL-specific clustering of information exposure and resource limits, not a new CWE number.
Source → sink: Anonymous (or low-priv) GraphQL request → introspection fields or a highly aliased/batched query → schema (including sensitive field names) is returned, or resolvers run far beyond a safe budget.
Code smells / review questions
- Is introspection off in production, or restricted to an authenticated admin role? Are alias, depth, complexity and batch limits enforced server-side (not by a denylist regex)?
- Smell: GraphiQL/Playground reachable without auth in prod; a regex "block introspection" that can be avoided by a semantically equivalent query; no cap on aliases per request.
Fix pattern
- Disable introspection and in-browser IDEs in production (graphql-js DisableIntrospection or the equivalent); require auth for schema docs; enforce depth/complexity/alias/batch limits independently of query shape.
Detection ideas
- From a test account (or unauthenticated, if that is in scope), check whether the production schema can be listed and whether docs/IDE routes are open. Review alias and batch limits. Describe the setting rather than pasting a schema-dump query if the program does not ask for one.
Real CVE lessons
- CVE-2023-47643 (SuiteCRM before 8.4.2): introspection was enabled without authentication. NVD CVSS 3.1 5.3 vs GitHub CNA 3.1. NVD primary CWE is NVD-CWE-noinfo; GitHub lists CWE-200. Not in KEV. Walked in §3d. -- https://nvd.nist.gov/vuln/detail/CVE-2023-47643
OSCP / report angle
- Bug bounty: report prod introspection or missing complexity limits, plus any sensitive field names the schema exposes (for example a password-hash field). Do not dump a full schema or a weaponised batched query in the report.
