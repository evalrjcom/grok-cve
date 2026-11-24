<!--
  P0F — Defensive CVE study notes.
  This file is for LEARNING vulnerability classes and remediation ONLY.
  It contains NO exploit code, NO payloads, and NO reproduction steps — by design.
  Code blocks show the *shape* of vulnerable vs fixed code taken from public patches,
  so you can recognize and fix the bug class, not weaponize it.
-->

# 3.8 Mass Assignment (CWE-915)

CWE: CWE-915 (https://cwe.mitre.org/data/definitions/915.html)
Root cause: Framework binds client parameters onto internal objects, letting attackers set privileged fields (role, isAdmin, price).
Source → sink: Request body fields → automatic binder / setattr loop → privileged model attributes updated.
Code smells / review questions
- Grep: update(req.body), Object.assign(model, body), strong params without deny of role/admin fields.
- Which attributes are bindable? Is there an allow-list per action?
- Smell: same DTO for create and admin-update; hidden fields trusted from client.
Fix pattern
- Explicit allow-list (strong parameters); separate DTOs per role; never bind role/price/owner from client.
Detection ideas
- Diff client-submitted fields vs model columns; try adding privileged keys in a test account (in-scope).
Real CVE lessons
- CVE-2026-93364 (Bludit CMS ≤ 3.22.0): Author role can modify admin-reserved page fields via reserved parameters; CWE-915; not KEV (NVD Deferred but record present). — https://nvd.nist.gov/vuln/detail/CVE-2026-93364
CVE walk — CVE-2026-62315 (Frappe ≤ 16.31.0 — field deny-list checked before the dict was parsed)
Why it matters: Mass assignment is a generic update API that accepts a whole object from the client and writes every key. Its defence is a deny- or allow-list of fields, and that list must be applied to the data that is actually written. Here the check ran on the raw value, so a dict-shaped value skipped it (a str-vs-dict type confusion).
Source → sink: Source = the fieldname argument of the whitelisted RPC frappe.client.set_value (a string or a JSON dict). Sink = the document field writes after parsing. The check compared the unparsed argument, not each parsed key.
Fix: Commit 2a04fae9353c02f0a9ce9f40f92c1eba6765c77b, "fix(client): fix checks for fieldnames parsed as dicts." (PR #38951, 28 Apr 2026): normalise to a dict first, then reject any forbidden field in it. The fixed release number is not stated in the NVD text (unverified).
Scoring: NVD status Deferred, so no NVD score. GitHub (CNA) CVSS 4.0 7.1 (AV:N/AC:L/AT:N/PR:L/UI:N/VC:L/VI:H); no CVSS 3.1 vector on the record. CWE-915 (CNA). Not in KEV. The fix predates the record (published 2026-08-20) by about four months.
- Bludit CVE-2026-93364 (≤ 3.22.0, Author role can modify admin-reserved page fields) stays a lesson only: no public fix diff verified.
Detection: Grep for generic update paths (set_value, Model.update(request.json), Mongoose findByIdAndUpdate(req.body), Rails update(params), Spring entities bound directly); test whether a low-privilege role can set privileged fields (role, owner, status, price) through them.
Remediation: Bind to DTOs or serializers with an explicit allow-list; deny-by-default for privileged fields with a per-field server-side permission check; validate after parsing, on the canonical form that will be written.
Takeaway: Check the representation you will write. Bug bounty: use your own accounts and records; report the field and the allow-list gap.
- Sources: https://nvd.nist.gov/vuln/detail/CVE-2026-62315 · https://github.com/frappe/frappe/security/advisories/GHSA-2c6h-wv85-fxvj · https://github.com/frappe/frappe/commit/2a04fae9353c02f0a9ce9f40f92c1eba6765c77b · https://github.com/frappe/frappe/pull/38951
OSCP / report angle
- Show the binder and the privileged field that should be excluded. Recommend strong-params allow-list.

