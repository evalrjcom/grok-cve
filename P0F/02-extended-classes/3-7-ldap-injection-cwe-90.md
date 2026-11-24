<!--
  P0F — Defensive CVE study notes.
  This file is for LEARNING vulnerability classes and remediation ONLY.
  It contains NO exploit code, NO payloads, and NO reproduction steps — by design.
  Code blocks show the *shape* of vulnerable vs fixed code taken from public patches,
  so you can recognize and fix the bug class, not weaponize it.
-->

# 3.7 LDAP Injection (CWE-90)

CWE: CWE-90 (https://cwe.mitre.org/data/definitions/90.html)
Root cause: Untrusted input is concatenated into an LDAP filter and interpreted as filter syntax.
Source → sink: Username/search param → LDAP filter string concat → directory query with altered logic.
Code smells / review questions
- Grep: ldap search filters built with string format; authentication plugins passing raw username into filters.
- Are LDAP special characters (*,),(,),\\,NUL) escaped per RFC 4515?
- Smell: login that reveals whether a DN exists via filter errors.
Fix pattern
- Use parameterized/escaped LDAP APIs; allow-list username charset; least-privilege bind accounts.
Detection ideas
- Review every LDAP auth and search path.
Real CVE lessons
- CVE-2016-8750 (Apache Karaf < 4.0.8): LDAPLoginModule did not encode usernames → LDAP injection DoS; CWE-90; not KEV. — https://nvd.nist.gov/vuln/detail/CVE-2016-8750
- CVE-2017-14596 (Joomla! < 3.8.0): LDAP auth plugin inadequate escaping → username/password disclosure; CWE-90; not KEV. — https://nvd.nist.gov/vuln/detail/CVE-2017-14596
CVE walk — CVE-2016-8750 (Apache Karaf < 4.0.8 — unescaped username in the LDAP login module)
Why it matters: A directory login module builds a search filter from the username. If filter metacharacters are not escaped, the user can change the filter's structure: authentication bypass, data disclosure or denial of service depending on how the result is used. NVD scores availability impact only, and Apache rates it Moderate.
Source → sink: Source = the username from the JAAS NameCallback. Sink = the LDAP search filter assembled in LDAPLoginModule.doLogin() (jaas/modules/.../ldap/LDAPLoginModule.java).
Fix: Commit ac07cb2440ceff94b3001728c1611fc471253d19, "[KARAF-4892] Encode user to avoid LDAP injection" (12 Dec 2016), fixed in 4.0.8: the name is RFC 2254-encoded before use. The Apache advisory links this commit through a retired git-wip host; I resolved it to the GitHub mirror by its abbreviated id, and NVD does not reference it.
Scoring: NVD CVSS 3.0 6.5 (AV:N/AC:L/PR:L/UI:N/S:U/C:N/I:N/A:H); CVSS 2 4.0; CWE-90. Not in KEV.
- Comparator: Joomla CVE-2017-14596 (NVD CVSS 3.0 9.8 vs CVSS 2 5.0; credential disclosure through the LDAP authentication plugin; fixed 3.8.0). Fix commit not verified, so no excerpt.
Detection: Grep for LDAP filters built by concatenation or format strings; test login with empty and over-long names in a lab; SCA for Karaf < 4.0.8.
Remediation: Upgrade; use the library's filter-escape helper (filters and DNs have different escaping rules, RFC 4515 vs RFC 4514); prefer search-then-bind-as-user over comparing attributes; reject empty passwords (anonymous bind); use a least-privilege service account.
Takeaway: Escape for the context where the string lands. Bug bounty: report the unescaped field and the safe-escaped fix; avoid wildcard searches that enumerate other users.
- Sources: https://nvd.nist.gov/vuln/detail/CVE-2016-8750 · https://karaf.apache.org/security/cve-2016-8750.txt · https://github.com/apache/karaf/commit/ac07cb2440ceff94b3001728c1611fc471253d19
OSCP / report angle
- Cite the filter construction and the escaping API. No filter-injection strings in the report.

