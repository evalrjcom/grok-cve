<!--
  P0F — Defensive CVE study notes.
  This file is for LEARNING vulnerability classes and remediation ONLY.
  It contains NO exploit code, NO payloads, and NO reproduction steps — by design.
  Code blocks show the *shape* of vulnerable vs fixed code taken from public patches,
  so you can recognize and fix the bug class, not weaponize it.
-->

# 3.9 Business-Logic Flaws (CWE-840)

No verified fix exists for any CWE-840 CVE, so this class stays concept-only (no CVE walk). The unpatched OpenCart CVE-2026-119558 is noted below as context, not as a walk.
CWE: CWE-840 (https://cwe.mitre.org/data/definitions/840.html)
Root cause: Workflow rules are incomplete: steps can be skipped, replayed, or reordered so invariants (payment, quota, approval) break.
Source → sink: Multi-step process → client-controlled step/state → server accepts illicit transition.
Code smells / review questions
- Can checkout skip payment? Can coupons stack unboundedly? Can approvals be replayed?
- Smell: trust client for price/quantity/step index; missing server-side state machine.
- Smell: negative quantities, race on redeem, reuse of one-time tokens.
Fix pattern
- Server-side state machine with explicit allowed transitions; recompute prices server-side; one-time tokens; idempotency keys.
Detection ideas
- Map every multi-step flow; attempt step reorder/replay in a test account.
Real CVE lessons
- No CVE walk: NVD has zero CVEs tagged CWE-840 (a category CWE, queried 2026-11-04). Its child CWE-841 returns four records: three are Chrome "inappropriate implementation" entries (not business logic) and OpenCart CVE-2026-119558 (NVD status Awaiting Analysis; VulnCheck CVSS 3.1 3.1, CWE-841; not KEV), a checkout step-ordering flaw that lets a logged-in customer place an unpaid order. It affects "through 4.1.0.4" with no fixed version or fix commit, so there is no patch to excerpt. https://nvd.nist.gov/vuln/detail/CVE-2026-119558 · https://www.vulncheck.com/advisories/opencart-through-4.1.0.4-payment-bypass-via-free-checkout-reward-points
OSCP / report angle
- Document the invariant that broke and the state-machine fix. Prefer demo accounts; never cause real financial harm.

