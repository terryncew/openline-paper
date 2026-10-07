# Claim Table — OpenLine Technical Paper

**Deliverable C** · 2026-10-06 · Derived from `evidence-ledger.md` (A) and `related-work-map.md` (B).
Every row points to a ledger entry. Nothing here is claimed beyond what the ledger supports.

---

## DEMONSTRATED

| # | Claim | Evidence |
|---|---|---|
| D1 | A worker can be replaced by a different provider's worker mid-job, under an unchanged owner-approved agreement, with acceptance decided by a third party — without transferring the first provider's chat, credentials, or filesystem home. | L-A1 (APPROVED_JOB_LIVE_001, PASS) |
| D2 | A valid worker credential does not widen the worker's mandate: the receiver enforces the exact action boundary and produces a signed refusal receipt. | L-B1 (CREDENTIAL_OVERREACH, PASS) |
| D3 | Old signed authority can remain authentic while losing standing; the receiver owns the execution decision. A genuine record can be too old to use. | L-A2 (PLATFORM_EXIT_LIVE_001, PASS) |
| D4 | Mandate revocation is enforced at the receiver decision layer, independent of transport authentication. | L-B2 (MUSE-RECEIVER-LIVE-001, PASS) |
| D5 | One owner-controlled STOP prevents protected post-STOP effects across independent receiver processes under stale state, delay, restart, and races. | L-B5 (DISTRIBUTED-STOP-001, PASS) |
| D6 | In-band trust-root succession does not return power to the superseded holder; historical receipts stay verifiable without conferring standing. | L-B6 (TRUST-ROOT-SUCCESSION-001, OWNER-CONTROL-002, PASS) |
| D7 | Deferred authority is evaluated at execution time against origin lineage — revoking the origin revokes the artifact's standing even after it travels through intermediaries. | L-C3 (TRUST-HANDOFF-001, PASS) |
| D8 | The kill-switch property traveled once across a materially separate implementation built from the frozen contract alone. | L-B4 (KILL-SWITCH-PORTABILITY-001, PASS) |
| D9 | The repaired receipt contract traveled to an independent clean-room implementation that reached matching dispositions. | L-E2 (INDEPENDENT-INTEROP-002, PASS) |
| D10 | Revocation protection is a race with a precise, falsifiable temporal rule; late revocation is recorded as late, never rewritten as prevention. | L-B7 (AUTHORITY_IN_TIME_001, PASS fixture) |
| D11 | Mid-task handoff across staged frozen state works repeatedly (10/10) with byte-verifiable accepted state. | L-A6 (HANDOFF-STATE-001, PASS) |
| D12 | Cross-subject provider replacement in code = revoke-old + fresh grant; the successor inherits nothing by default. | L-A3 (code fact) |
| D13 | No structural route around the receiver was found for the covered resource and preregistered routes. | L-B8 (BYPASS-001, PASS) |
| D14 | The evidence-ledger prototype mechanically enforces honest terminal labeling (a FAIL relabeled PASS fails validation). | L-D4 (openline-bureau) |

**Scope note (applies to all D-rows):** localhost/development fixtures unless stated otherwise. None of D1–D14 establishes production deployment safety, payment safety, production key custody, cross-machine revocation propagation, or external adoption.

---

## INFERRED

| # | Claim | Basis | Confidence |
|---|---|---|---|
| I1 | The receiver-decides architecture generalizes beyond the tested fixtures to any boundary where the receiver actually mediates the consequence. | D2, D4, D5, D13 share one mechanism (receiver-owned decision before effect). | Moderate — mechanism is consistent, but each new boundary is a new test. |
| I2 | The revocation-safety inequality is the right admission rule for production receivers. | D10 (fixture) + standing methodological constraint. | Low-moderate — the rule is precise; no production timing bounds exist. |
| I3 | Interop-001's contract gap (no pinned verdict/decision pairing for terminal DENY) was the binding constraint on independent implementation, and its repair generalizes. | L-E1 → L-E2 sequence. | Moderate — one repair, one confirmation. |

---

## PROPOSED (not demonstrated)

| # | Claim | Status |
|---|---|---|
| P1 | Cross-machine revocation propagation. | Not demonstrated in any component (wallet VISION.md "has not been demonstrated" list). |
| P2 | Production deployment safety / payment safety / production key custody. | Explicitly disclaimed in every experiment's claim boundary. |
| P3 | The bound-authority narrow profile (EMILIA draft fit). | Design direction only (EXP-25, INFERRED); build stopped at LOC ceiling, repriced, not landed. |
| P4 | STOP-arrival benchmark showing receiver-enforced arming beats middleware. | Prototype PARKED; all timing simulated. |
| P5 | Trust-root succession in production / at scale. | Only the in-band fixture (D6). |
| P6 | External receiver acceptance of OpenLine-issued authority. | See U1. |

---

## UNRESOLVED

| # | Question | Status |
|---|---|---|
| U1 | **Can a genuinely external receiver recognize authority it did not create?** | UNRESOLVED as of 2026-10-06. Substantive replies from four parties, zero adoptions, zero integrations. Proof-of-Control #57 + route-5 PR open; RSA probe window closes 2026-10-23; Sierra PAP v0.1 not shipped. This is the paper's load-bearing gap — see L-F1. |
| U2 | Does the six-property combination (owner-defined mandate + receiver decides before effect + survives provider switch + revocation/history travel + portable evidence + receiver keeps own policy) exist elsewhere? | Related-work map (B) verdict: NO existing system combines all six as of 2026-10-06. Closest: Google AP2 v0.2 (fails revocation — explicitly out of scope). Falsifier stated in the map. |
| U3 | Do signed receiver receipts provide value beyond normal audit logs in a real dispute? | No real dispute has occurred. The bureau prototype (D14) shows mechanical enforceability of honest labeling; evidentiary value in an actual dispute is untested. |

---

## What changed because of failures (for §10 of the paper)

- interop-001's INCOMPLETE forced a contract repair in the public documents before 002 could pass — the documents, not just the code, are part of the system.
- replace-worker-keep-job 001's FAIL exposed a real mechanism gap (`join()` refusing to retire revoked sessions) that 002's `replace_worker` path closed.
- containment-001/002's apparatus failures (provider strict-mode schema defects) ended the semantic-helper composition lane without a positive terminal — the deterministic core survived, the composition claim did not.
- BOUNDED-CALL-001's STOPPED_AT_FIRST_FAILURE is the standing example of the qualification gate working as designed: it caught its failure and stopped.
- PROVIDER-EFFECT-LIVE_001's negative bounds the history claim: closure evidence is receiver-local unless the receiver exports it.
- The openline-bureau SHA-256 anomaly is preserved as a discrepancy, not repaired — "source bytes are sacred."
