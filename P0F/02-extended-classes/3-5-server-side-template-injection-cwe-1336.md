<!--
  P0F — Defensive CVE study notes.
  This file is for LEARNING vulnerability classes and remediation ONLY.
  It contains NO exploit code, NO payloads, and NO reproduction steps — by design.
  Code blocks show the *shape* of vulnerable vs fixed code taken from public patches,
  so you can recognize and fix the bug class, not weaponize it.
-->

# 3.5 Server-Side Template Injection (CWE-1336)

CWE: CWE-1336 (https://cwe.mitre.org/data/definitions/1336.html)
Root cause: Untrusted input is embedded into a template that is then compiled/rendered on the server, so template syntax executes.
Source → sink: User string → template engine render/compile → expression evaluation / RCE-class impact.
Code smells / review questions
- Grep: render_template_string, Template(user), Freemarker/Velocity/Jinja/ERB with user content as the template.
- Is user input data bound into a static template, or is it the template source?
- Smell: email-test / notification preview that renders user HTML as a template.
Fix pattern
- Never compile user strings as templates; sandbox engines; allow-list template names; bind data only.
Detection ideas
- Hunt preview/test-email/report-builder features; review admin-only template editors for sandbox escape.
Real CVE lessons
- CVE-2022-22954 (VMware Workspace ONE Access): SSTI leading to RCE (NVD CWE-94; description confirms SSTI); KEV 2022-04-14, ransomware Known. — https://nvd.nist.gov/vuln/detail/CVE-2022-22954
- CVE-2023-22527 (Confluence Data Center/Server): Template injection → unauth RCE on older versions (NVD CWE-74); KEV 2024-01-24, ransomware Known. — https://nvd.nist.gov/vuln/detail/CVE-2023-22527
CVE walk — CVE-2025-27516 (Jinja < 3.1.6 — sandbox bypass through the |attr filter)
Why it matters: Jinja's SandboxedEnvironment is the standard way to let semi-trusted users author templates (themes, notification or email templates, prompt templates). A sandbox escape turns "can edit a template" into code execution in the application process. Exposure depends on who controls template content.
Source → sink: Source = template text an attacker can author or influence. Sink = the sandboxed environment evaluating it. Chokepoint = the attr filter in src/jinja2/filters.py, which read attributes directly instead of going through the environment's getattr, so the sandbox's safety checks were skipped.
Fix: Commit 90457bbf33b8662926ae65cdde4c4c32e756e403 (merge of the GHSA-cpwx-vrp4-4pq7 private fork; underlying change "attr filter uses env.getattr", 5 Mar 2025), released in 3.1.6: the filter now calls environment.getattr so sandbox checks apply.
Scoring: NVD CVSS 3.1 8.8 (AV:L/AC:L/PR:L/UI:N/S:C/C:H/I:H/A:H) vs GitHub (CNA) CVSS 4.0 5.4 (AV:L/AC:L/AT:P/PR:L/UI:P/VC:H/VI:H/VA:H): a 3.4-point gap from different attack-requirement, user-interaction and scope assumptions. CWE-1336 (CNA); NVD shows no CWE of its own. Not in KEV.
- Closed-source KEV siblings (no public diff; remediation is the vendor patch): VMware Workspace ONE CVE-2022-22954 and Confluence CVE-2023-22527, both KEV with ransomware use Known (see the lessons above).
Detection: Grep for render_template_string, Template(<user data>), from_string on user input, and for sandboxed vs plain Environment; SCA for jinja2 < 3.1.6.
Remediation: Upgrade to ≥ 3.1.6. Never build template source from user input; pass user data as context variables. Treat the sandbox as defence-in-depth: render untrusted templates in a low-privilege, isolated process.
Takeaway: A sandbox is a patchable allow-list, not a boundary — expect more bypasses. Bug bounty: demonstrate evaluation with a harmless arithmetic probe only; no code execution or file reads.
- Sources: https://nvd.nist.gov/vuln/detail/CVE-2025-27516 · https://github.com/pallets/jinja/security/advisories/GHSA-cpwx-vrp4-4pq7 · https://github.com/advisories/GHSA-cpwx-vrp4-4pq7 · https://github.com/pallets/jinja/commit/90457bbf33b8662926ae65cdde4c4c32e756e403
OSCP / report angle
- Show that user input becomes template source. Recommend static templates + data binding. No template expression payloads in the ticket.

