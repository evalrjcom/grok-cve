<!--
  P0F — Defensive CVE study notes.
  This file is for LEARNING vulnerability classes and remediation ONLY.
  It contains NO exploit code, NO payloads, and NO reproduction steps — by design.
  Code blocks show the *shape* of vulnerable vs fixed code taken from public patches,
  so you can recognize and fix the bug class, not weaponize it.
-->

# 2.19 Prototype Pollution (CWE-1321)

Status: covered in lessons — jQuery CVE-2019-11358 and lodash CVE-2019-10744 are both walked below.
CWE: CWE-1321 (https://cwe.mitre.org/data/definitions/1321.html)
Root cause: Attacker-controlled keys that reach the object prototype (the prototype-link and constructor properties) merge into Object.prototype, changing behavior app-wide.
Source → sink: JSON/query merge/defaultsDeep → recursive assign without key deny → polluted base object.
Code smells / review questions
- Grep: lodash merge/defaultsDeep/set, jQuery.extend(true), recursive assign of user JSON.
- Are __proto__, prototype, constructor rejected before merge?
- Smell: deep merge on old lodash (<4.17.12 for defaultsDeep).
Fix pattern
- Upgrade lodash; Object.create(null) maps; block dangerous keys; prefer Map for user-keyed data.
Vulnerable shape (educational)

Fixed shape

Detection ideas
- Unit-test merges reject reserved keys; SCA on known polluted merge functions.
Real CVE lessons
CVE walk — CVE-2019-11358 (jQuery < 3.4.0, deep jQuery.extend(true, {}, ...))
- Affected: GHSA-6c3j-c64m-qhgq lists npm jquery >=1.1.4, <3.4.0. Also shipped in Drupal, Backdrop CMS, Django admin (<2.1.9 / <2.2.2), jquery-rails <4.3.4, and maximebf/debugbar <1.19.0.
Root cause: The merge loop copied every enumerable key and recursed into nested objects without checking for the key __proto__, so a JSON source object an attacker shaped could write onto the shared Object.prototype.
Source → sink: Source = parsed JSON (request body, stored settings, or options built from the URL). Sink = deep $.extend.
Fix: Commit 753d591aea698e57d6db58c9f722cd0808619b1b changed the loop guard and added the comment // Prevent Object.prototype pollution.
- Analysis (note on the fix): it blocks only __proto__. Other merge libraries (e.g., lodash) also had to block constructor / prototype — do not treat a __proto__-only guard as complete in your own code.
Impact: Advisories say it "could extend the native Object.prototype". GHSA also classifies it as XSS (CWE-79). Real impact depends on the gadgets in the app. No RCE is claimed.
Scoring: NVD CVSS 3.1 6.1 (AV:N/AC:L/PR:N/UI:R/S:C/C:L/I:L/A:N); CVSS 2 4.3. No CNA or ADP score. NVD CWE-1321; GHSA CWE-79 + CWE-1321. Not in KEV (the only KEV CWE-1321 entry is closed-source Adobe Acrobat CVE-2026-34621).
Detection: npm audit, Dependabot, Retire.js; grep for deep merge/extend/set-path helpers that take request data; CodeQL js/prototype-pollution* rules (unverified).
Remediation: Upgrade to jQuery 3.4.0+; schema-validate input (Ajv additionalProperties: false); Object.create(null) or Map for user-keyed data; Object.hasOwn checks; Object.freeze(Object.prototype) (test for breakage first); Node --disable-proto (does not apply in browsers).
Bug bounty takeaway: Pollution alone is often triaged low — report the impact of the reachable gadget.
- Sources: https://nvd.nist.gov/vuln/detail/CVE-2019-11358 · https://github.com/advisories/GHSA-6c3j-c64m-qhgq · https://github.com/jquery/jquery/commit/753d591aea698e57d6db58c9f722cd0808619b1b
CVE walk — CVE-2019-10744 (lodash < 4.17.12, defaultsDeep — the constructor route)
Why __proto__ alone isn't enough: An object reaches Object.prototype two ways: directly via __proto__, or via its constructor (the Object function) and then that function's .prototype. jQuery's fix and lodash's pre-4.17.12 safeGet closed only the first.
Source → sink: Source = attacker-controlled request JSON or config. Sink = _.defaultsDeep(target, source). Chokepoint = the internal helper safeGet, which the deep merge uses for nested reads.
Fix: Commit 1f8ea07746963a535385a5befc19fa687a627d2b, "fix: prototype pollution in _.defaultsDeep (#4336)", 24 Jun 2019, +5/-1 in lodash.js.
Why the typeof check: It refuses only a real constructor function; a plain data field named "constructor" still merges. One helper = one chokepoint for every path that uses it.
Scoring: NVD CVSS 3.1 9.1 (AV:N/AC:L/PR:N/UI:N/S:U/C:N/I:H/A:H); CVSS 2 6.4; scored by NVD only. CWE-1321. Not in KEV.
- Fixed versions differ by package (GHSA-jf85-cpcp-j695): lodash 4.17.12, lodash-amd 4.17.13, lodash-es 4.17.14, lodash.defaultsdeep 4.6.1, lodash-rails 4.17.12.
- Later bug in other code paths: CVE-2020-8203 (GHSA-p6mc-m468-83gw), zipObjectDeep/set paths. Fixed in lodash 4.17.19, lodash-es 4.17.20 (NVD says "before 4.17.20"). NVD 7.4. GHSA lists NO patched version for lodash.pick, lodash.set, lodash.setwith, lodash.update and lodash.updatewith — replace them with full lodash >= 4.17.21.
- Related: CVE-2018-16487 (merge/mergeWith/defaultsDeep < 4.17.11), NVD 5.6. HackerOne (the CNA) says CWE-400; NVD says NVD-CWE-noinfo.
Detection: npm audit; npm ls lodash lodash-es lodash.merge lodash.defaultsdeep lodash.set lodash.setwith lodash.update lodash.pick; grep lockfiles for every resolved lodash version; npm overrides / yarn resolutions; npm ci in CI.
Takeaway: Denylist __proto__, constructor and prototype at minimum; better, use Object.create(null) or Map plus schema validation. A fix in one function doesn't cover the whole library. Bug bounty: an old lodash in a lockfile is informational until it's paired with a reachable deep-merge sink plus a gadget.
- Sources: https://nvd.nist.gov/vuln/detail/CVE-2019-10744 · https://github.com/advisories/GHSA-jf85-cpcp-j695 · https://github.com/lodash/lodash/commit/1f8ea07746963a535385a5befc19fa687a627d2b · https://github.com/advisories/GHSA-p6mc-m468-83gw
OSCP / report angle
- Show the merge sink and dangerous key acceptance. No pollution→RCE gadget chains.
