<!--
  P0F — Defensive CVE study notes.
  This file is for LEARNING vulnerability classes and remediation ONLY.
  It contains NO exploit code, NO payloads, and NO reproduction steps — by design.
  Code blocks show the *shape* of vulnerable vs fixed code taken from public patches,
  so you can recognize and fix the bug class, not weaponize it.
-->

# 1b. Coverage map

Every bug class in this cheatsheet. "Covered in lessons" = walked with the verified CVE set from the brief. "Walked in doc (supplementary)" = a §3 class with its own CVE walk added later (not part of the live lessons). "[supplementary]" / * CVEs were added from NVD for classes that needed well-known examples and are labeled in the lessons. "Walked in doc (supplementary)" also covers the §3b walks of real-world CVEs. Yellow status cell = concept-only class (no verified fixed CVE). "Walked in doc (supplementary)" also covers the §3c batch-3 walks (verified 2026-11-12, KEV catalog 2026.11.10). "Walked in doc (supplementary)" also covers the §2b/§3d new-category classes and walks (verified 2026-11-12, KEV catalog 2026.11.10).
* = supplementary CVE (added from NVD; not in the walked lesson set). † = NVD primary CWE differs from the class framing; description confirms the pattern. — = no example CVE in lessons. † also covers missing NVD primary CWE or CWE-noinfo in §3c. † also covers missing NVD primary CWE or CWE-noinfo in §3d.
