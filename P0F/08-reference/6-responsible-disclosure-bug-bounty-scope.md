<!--
  P0F — Defensive CVE study notes.
  This file is for LEARNING vulnerability classes and remediation ONLY.
  It contains NO exploit code, NO payloads, and NO reproduction steps — by design.
  Code blocks show the *shape* of vulnerable vs fixed code taken from public patches,
  so you can recognize and fix the bug class, not weaponize it.
-->

# 6. Responsible Disclosure & Bug Bounty Scope

- Stay in program scope: only listed assets/environments/classes. Out-of-scope findings still go through proper channels.
- Follow each platform's rules (HackerOne, Bugcrowd, Intigriti, vendor PSIRT, CISA VDP). Respect rate limits and prohibited testing.
- Report rather than exploit: minimum proof (code citation + benign request/response). No pivot, bulk exfil, or persistence.
- Good report: summary, version, CWE, source→sink, fix commit or pattern, CVSS/SSVC, KEV relevance. Diffs/screenshots — not weaponized archives.
- Patch bypasses: reference the prior CVE and show why the invariant still fails — high-signal hunting.
- Legal: authorization is the line between research and crime. When unsure, stop and ask the program.
- OSCP labs: attack only lab targets; bring the same reporting discipline to real programs.
— End of cheatsheet v6 —
Defensive education for VP Roju · Prefer fix diffs as ground truth · Hunt smells, report fixes
