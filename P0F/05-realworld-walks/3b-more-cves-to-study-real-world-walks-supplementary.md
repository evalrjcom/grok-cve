<!--
  P0F — Defensive CVE study notes.
  This file is for LEARNING vulnerability classes and remediation ONLY.
  It contains NO exploit code, NO payloads, and NO reproduction steps — by design.
  Code blocks show the *shape* of vulnerable vs fixed code taken from public patches,
  so you can recognize and fix the bug class, not weaponize it.
-->

# 3b. More CVEs to Study — Real-World Walks (supplementary)

Fourteen further CVE walks in the same format as §3 (plus ExifTool CVE-2021-22204 as the root cause inside the GitLab walk), chosen for patterns the earlier sections lack: supply chain (a release tarball that differs from git, a mutable action tag), signal-handler races, path normalisation, parser defaults (@file expansion, lookups, data binding), trusted internal headers and pre-authentication state. Verified 2026-11-04 against the NVD 2.0 API, the CISA KEV feed (catalog 2026.11.04), vendor advisories and the public fix commits; excerpts are fix source only. Closed-source or unpublished fixes are marked. Numbered 3b so the section numbers used elsewhere do not change. No exploit code, payloads or reproduction steps appear anywhere in this section.
