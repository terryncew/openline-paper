# Final [VERIFY] List — OpenLine Technical Paper

**Deliverable G** · 2026-10-06 · Everything below must be resolved or explicitly acknowledged before publication.

---

## MUST RESOLVE (load-bearing for central claims)

1. **CREDENTIAL_OVERREACH_LIVE_001 run inspection.** RESOLVED 2026-10-07 — run 34725225216 independently inspected via `gh`: conclusion `success`, head `c7ac6a24aa6995bfb446d0110354cf715b8de464`; real-hosts artifact `result.json` verdict `CREDENTIAL_OVERREACH_CONTAINMENT_ENFORCED` with `live_models=true`, `worker_subject_key_used=true`, `provider_credentials_forwarded_to_attacker=false`, `spontaneous_model_compromise_claim=false`. No softening needed.
2. **APPROVED_JOB_LIVE_001 ↔ Airlock linkage.** RESOLVED 2026-10-07 — pin `fb02207f` (2026-09-07, PR #110) inspected: ELIGIBLE = candidate satisfied the owner-approved agreement's checks as evaluated by Airlock's own acceptance-evidence machinery (repo-discovered test commands run against the candidate), not the worker's self-report. Receipts are HMAC-SHA256 under a receiver-local key (`src/airlock/receipt.py`) — attests this Airlock instance recorded the decision, not cross-system identity. No worker-succession logic in Airlock; the harness performed the handoff. Precise paragraph now in the ledger (L-A1) and draft §7.
3. **Truncated digests.** RESOLVED 2026-10-06 — `51c69f48…` → `51c69f48dbd2484431d6a62c307f104d00fee77cc6a5ecacac901d473fe51b68` (sha256 of `evidence.jsonl`, per FREEZE.md frozen sums); `bace02a6…` → `bace02a6f08f4f15b780096ead4b5e76e3281873492e57924dbfe75ac5622be4` (sha256 of `impl-b-evidence.jsonl`, per `portable/evidence/SHA256SUMS.txt`). Expanded in evidence-ledger.md, draft.md, outline.md; citation-audit.md rows updated. Both from the frozen closure-audit copy of terryncew/openline-kill-switch (`~/workspace/closure-audit/repos/openline-kill-switch/`). Independently re-verified 2026-10-07: both digests reproduce via `sha256sum` against the working-tree files (`~/workspace/kill-switch-reference-001/evidence.jsonl`, `~/workspace/kill-switch-portability-001/impl-b/evidence/evidence.jsonl`).
4. **Handoff commit `653a8067…` (L-A1).** RESOLVED 2026-10-07 — not a repo-history commit (correctly absent from `git log --all`); it is the experiment's ephemeral checkpoint commit, attested in frozen evidence: `proofs/approved-job-live-001/frozen/real/evidence.json` → `handoff_commit = 653a8067b3d45dde39957f9ad18d90822367cdef`, `handoff.json` → `candidate_commit` (same). The same frozen evidence independently confirms: agreement digests identical before/after (`15f68e45e…`), final candidate `db722a2ad1c4059b05e498749d44c2b5f9120c35`, `provider_a_invocations_after_handoff = 0`. Paper cites it as "per frozen evidence" — accurate.

## SHOULD RESOLVE (boundaries and wording)

5. **PLATFORM-EXIT-KILL-002 exact repo/commit.** RESOLVED 2026-10-07 — ran in `~/workspace/openline-receipt-gate/experiments/platform-exit-kill-002/`; `TERMINAL.md` terminal state PASS; 002 prereg sha256 `7e440cf5f6ddeb1e949476aceb8a74ddd3ddbadc86de36f2943b33f1b45ede38`; baseline main `95c1ecfd77bd13b9e7b20b`. Draft appendix updated.
6. **replace-worker-keep-job repo name.** The 001/002 runs' repo name was not recorded in the files read.
7. **FIXED-AUTHORITY-SCALE-001 / PROVIDER-EFFECT-LIVE-001 / RECOVERY-001 / AUTOCOMPACT-ADMISSION-001.** Exact prereg claim wordings not extracted — needed only if cited beyond the ledger's summaries.
8. **Exactly-once / receiver crash-recovery (2026-09-12/13).** No experiment ID or terminal verdict in the dated logs; may be WALLET_EFFECT_CLOSURE_001 / WALLET_CLOSURE_SET_001. Not cited in the draft — resolve only if added.
9. **BOUNDED-CALL-001 PRECONTACT-AMENDMENT-001.md.** Content not read. Not cited in the draft — resolve only if added.
10. **Contract-repair merge `2f03346` (L-E2) and PR #95 merge `41631b7` (L-B6).** Cited from records; not independently re-verified.

## RECHECK BEFORE PUBLICATION (time-sensitive)

11. **Sierra Personal Agent Protocol v0.1.** Not shipped as of 2026-10-06 (announced 2026-10-06, "later this month"). If v0.1 ships before publication, the related-work section and falsifier #1 must be re-evaluated.
12. **External lanes.** RSA probe window closes 2026-10-23; Proof-of-Control public comment closes 2026-10-30; PoC route-5 PR open/unmerged; bank implementation paper not yet released; Proof/x401 outreach unsent. Any change moves L-F1/U1.
13. **External adoption claim.** "Zero adoptions as of 2026-10-06" must be rechecked against the outreach table at publication time.

## RELATED-WORK VERIFY ITEMS (from deliverable B — paper-relevant subset)

14. x401 spec details (Verifier-issued Verification Token, PROOF-RESULT path) — per the map's primary-source reading; not re-verified in the citation audit.
15. MCP Authorization spec details (token passthrough forbidden) — per the map's reading.
16. A2A v1.0 §7.6 wording — per the map's reading.
17. EMVCo agentic-payments draft status — Board of Advisors meets Oct 13–14, after this list's date.
18. Six-bank paper clause content — mapping exists; PDF text not directly quoted in the research pass.
19. Okta/Entra agent-identity GA dates and details.

## NON-LOAD-BEARING (resolve opportunistically)

20. openline-bureau PUBLIC_EVIDENCE.md build date; openline-interop-fix-001 status; obsigna #1085 silence assumption; warning-window underlying pair IDs; PAYBACK-003 scope.

---

## Corrections already made (do not re-open)

- `a3489308` → `a348930` (PR #92 merge commit typo).
- KILL-SWITCH-RECEIPT-INTEGRATION-001B: "PR not merged" → merged as PR #93 (`d618ce8`) on 2026-09-19.
- Proof-of-Control comment deadline: Oct 7 → Oct 30 (corrected 2026-10-05).
