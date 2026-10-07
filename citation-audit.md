# Citation Audit — OpenLine Technical Paper

**Deliverable F** · 2026-10-06 · Every cited repository, commit, and external claim checked.

---

## 1. Repositories cited

| Repo | Cited as | Verified |
|---|---|---|
| github.com/terryncew/openline-wallet | public | YES — HEAD `43997748` reachable 2026-10-06 |
| github.com/terryncew/openline-airlock | public | YES — HEAD `ac7da750` reachable 2026-10-06 |
| github.com/terryncew/openline-core | public (restored 2026-09-30) | YES — HEAD `2ddb152f` reachable 2026-10-06 |
| github.com/terryncew/openline-receipt-gate | public | YES — HEAD `abc610be` reachable 2026-10-06 |
| github.com/terryncew/openline-handoff-state | public | YES — refs reachable 2026-10-06 |
| github.com/terryncew/openline-world | public | YES — local clone verified |
| github.com/terryncew/openline-kill-switch | public | YES — HEAD `a3037967`, tag v0.1.0 → `ebf2522f4b` verified 2026-10-06 |

## 2. Commits / SHAs cited

| SHA (as cited) | Where | Status |
|---|---|---|
| `7d4e8fe501528e3e7758921d28b58e2cecd3f18c` | L-A1, draft §16 (APPROVED_JOB_LIVE_001 head) | VERIFIED — commit exists in openline-wallet |
| `653a8067b3d45dde39957f9ad18d90822367cdef` | L-A1 (handoff commit) | CITED FROM REPO RECORD — not independently re-verified; keep [VERIFY] if load-bearing |
| `711ced888befc6ed9f64dd22cc30e9a14f7b5cb1` | L-A2, draft §16 (PLATFORM_EXIT head) | VERIFIED — commit exists in openline-wallet |
| `c7ac6a24aa6995bfb446d0110354cf715b8de464` | L-B1, draft §16 (CREDENTIAL_OVERREACH main) | VERIFIED — commit exists; run 34725225216 independently inspected 2026-10-07 (success; artifact verdict `CREDENTIAL_OVERREACH_CONTAINMENT_ENFORCED`) |
| `a5095ce` | L-B7 (AUTHORITY_IN_TIME_001) | VERIFIED — "proof(wallet): add AUTHORITY-IN-TIME-001", 2026-09-09 |
| `687bf0d8` | L-C3 (TRUST-HANDOFF-001 wallet pin) | VERIFIED — "Merge pull request #47" |
| `9c06dfd6` | L-C3 (receipt-gate pin) | VERIFIED — "Merge pull request #89" |
| `a348930` | L-C2, draft §16 (PR #92 merge) | VERIFIED — "Merge pull request #92". **CORRECTED from `a3489308` (typo in source file)** |
| `2061043` | L-C2 (integration commit) | VERIFIED — ancestor of origin/main via PR #93 |
| `d618ce8` | L-C2 (PR #93 merge) | VERIFIED — "Merge pull request #93", 2026-09-19. **CORRECTED "PR not merged" → merged** |
| `fb02207f` | L-A1 (Airlock pin) | VERIFIED — commit exists in local openline-airlock clone |
| `51b202a` | Draft §16 (WORLD-AUTHORITY-001) | VERIFIED — "WORLD-AUTHORITY-001: preserve repair browser evidence" |
| `3e8382b` | Draft §16 (ATTACK-DEMO-001) | VERIFIED — "Keep refusal card inside the portrait viewport" |
| `777691bf…` | L-A6 (HANDOFF-STATE-001 final state) | CITED FROM RECORD — 64-hex state digest, not a commit; keep as cited |
| `ebf2522` | L-B4 (kill-switch tag v0.1.0) | VERIFIED — tag v0.1.0 → `ebf2522f4b` |
| `2f03346` | L-E2 (contract-repair merge) | CITED FROM RECORD — not independently re-verified |
| `41631b7` | L-B6 (PR #95 merge) | CITED FROM RECORD — not independently re-verified |
| `7e440cf5…` | L-A4 (PLATFORM-EXIT-KILL-002 prereg) | CITED FROM RECORD — not independently re-verified |

**Corrections made during this audit:**
1. `a3489308` → `a348930` (8-char typo; the real PR #92 merge commit).
2. KILL-SWITCH-RECEIPT-INTEGRATION-001B: "PR opened, NOT MERGED" → merged as PR #93 (`d618ce8`) on 2026-09-19. The experiment record captured the pre-merge state.

## 3. Artifact digests cited

| Digest | Where | Status |
|---|---|---|
| `96a55c659059ef4bdfca55b231c2e5664eba774c86dfb22d0b59a27c4af56ec2` | L-A1 (APPROVED_JOB_LIVE_001 artifact) | CITED FROM REPO RECORD |
| `281ae549b31f7a9f4ec940395da488d2b6fa5eb12a9c9bb70471dcc314532203` | L-A2 (PLATFORM_EXIT artifact) | CITED FROM REPO RECORD |
| `51c69f48dbd2484431d6a62c307f104d00fee77cc6a5ecacac901d473fe51b68` | L-B3 (KILL-SWITCH-REFERENCE evidence) | RESOLVED 2026-10-06 — expanded from frozen record: sha256 of `evidence.jsonl` per FREEZE.md frozen sums (`~/workspace/closure-audit/repos/openline-kill-switch/study/FREEZE.md`) |
| `bace02a6f08f4f15b780096ead4b5e76e3281873492e57924dbfe75ac5622be4` | L-B4 (PORTABILITY evidence) | RESOLVED 2026-10-06 — expanded from frozen record: sha256 of `impl-b-evidence.jsonl` per `portable/evidence/SHA256SUMS.txt` in the frozen closure-audit copy of terryncew/openline-kill-switch |

Truncated digests (`…`) must be expanded from the frozen records before publication.

## 4. External claims cited

| Claim in draft | Source | Verification |
|---|---|---|
| AP2 v0.2: revocation explicitly out of scope ("Mandate Management" out of scope) | related-work-map.md (primary: github.com/google-agentic-commerce/AP2, read 2026-10-06) | CORROBORATED by independent third-party research (az-said/interlock docs/09-research-ap2.md, 2026-10-06): "Revocation is not covered… The spec calls 'Mandate Management' out of scope." |
| x401: Verifier-issued Verification Token + PROOF-RESULT path | related-work-map.md | Per map's primary-source reading; not re-verified in this audit — keep map's citations |
| MCP Authorization: token passthrough forbidden | related-work-map.md (primary: modelcontextprotocol.io spec) | Per map's primary-source reading |
| A2A v1.0 §7.6: "does not define the scope, representation, validity, or revocation semantics" | related-work-map.md (primary: a2a-protocol.org) | Per map's primary-source reading |
| Sierra Personal Agent Protocol: announced 2026-10-06, v0.1 not shipped | Direct verification 2026-10-06 (PAP-RECEIVER-001): announcement coverage only; "v0.1 planned for later this month"; no spec repo found | VERIFIED — recheck before publication |
| Six-bank "Building Trust in Agentic Commerce": 2026-09-22 | Memory + clause mapping at workspace/experiments/bank-principles-001/CLAUSE-MAPPING.md | Date per memory; mapping exists |
| EMVCo agentic-payments: draft framework, comment through 2026-09-30 | related-work-map.md | Per map; Board of Advisors meets Oct 13–14 — post-dates this audit |
| LFDT Proof-of-Control: issue #57 filed, public comment closes 2026-10-30 | Memory (verified 2026-10-05: three sources confirm Oct 30) | VERIFIED |
| No external org has adopted OpenLine as of 2026-10-06 | experiment-evidence.md §10 (outreach table) | Per the record; recheck before publication |

## 5. Internal consistency checks

- "45 recorded experiments (2026-09-04 to 2026-10-06)": earliest EXP-02 (2026-09-04), latest entries 2026-10-06 — CONSISTENT.
- Draft §9B caveat references "L-B8-caveat" — the STOP-arrival PARKED caveat lives under L-B8's section in the ledger ("Caveat — STOP-arrival track: PARKED"). Label is slightly off; the content is present. Acceptable.
- Draft abstract says "45 recorded experiments" — matches EXP-01..EXP-45. CONSISTENT.
- Claim-table D-rows all have ledger entries. CONSISTENT.
- Outline §16 SHAs match the audit table above, with the two corrections applied.

## 6. Items that must be resolved before publication (see also G)

- Expand truncated digests (`51c69f48…`, `bace02a6…`) from frozen records. — DONE 2026-10-06, see item 3.
- GitHub Actions run 34725225216 (CREDENTIAL_OVERREACH) — DONE 2026-10-07, independently inspected.
- Re-verify Sierra PAP status (v0.1 may have shipped).
- Re-verify external-lane statuses (RSA probe window closes 2026-10-23; PoC comment closes 2026-10-30).
- Confirm the exact Airlock code exercised by APPROVED_JOB_LIVE_001 beyond the pin.
