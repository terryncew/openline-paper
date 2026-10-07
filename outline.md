# Paper Outline — with evidence attached to each section

**Deliverable D** · 2026-10-06
Working title: **Receiver-Owned Authority for AI Agents: Portable Mandates, Receiver Enforcement, and Verifiable Receipts**
Author: Terrynce White

Evidence keys: L-x = `evidence-ledger.md`, D/I/P/U-n = `claim-table.md`, B = `related-work-map.md`.

---

## 1. Abstract (150–250 words)

- Problem: agents now request actions in outside systems; the question of who validates their authority is unsettled.
- Architecture: the receiver decides — owner-defined mandates, receiver enforcement before effect, portable signed receipts.
- Implemented: wallet (mandates/revocation history), receiver gate (`@authorize` before effect), Airlock (independent acceptance), wire receipt formats, claim-graph recall, OTel evidence adapters. All experimental. [Components table, L-§Components]
- Experiments: 45 recorded; strongest: provider replacement under an unchanged agreement with third-party acceptance [D1/L-A1]; valid-credential out-of-mandate refusal with signed receipt [D2/L-B1]; one STOP across independent receivers [D5/L-B5]; independent clean-room implementation of the repaired contract [D9/L-E2].
- Strongest demonstrated result: D1+D2 composition — the worker changed, the agreement didn't, and the receiver (not the worker) enforced the boundary.
- Major limitation: no external receiver has adopted or recognized the authority; all tests are localhost/development fixtures [U1/L-F1].

## 2. Introduction: When agents begin acting

- Shift from models answering questions to agents requesting actions: payments, code deployment, purchasing, data access, job execution, multi-agent handoffs.
- Central question: when an agent asks another system to do something consequential, who gets to decide whether its authority is valid?
- Evidence: none needed (framing). Keep concrete, no marketing.

## 3. The architectural problem

- Five-way distinction: capability vs identity vs authority vs receiver acceptance vs evidence.
- Why vendor logs don't solve evidence: the party being asked to act needs its own decision record; a log the agent's vendor controls is not the receiver's evidence. [D2/L-B1 — the receiver produced its own signed STOP receipt; D14/L-D4 — honest labeling enforced mechanically.]
- No claim beyond the distinction itself.

## 4. Design principle: the receiver decides

- Plain-language model: principal defines the mandate; agent proposes; receiver checks mandate + its own rules; receiver allows/refuses; system creates a verifiable record.
- The receiver never blindly trusts the agent. [D2/L-B1; D4/L-B2 — transport stayed authenticated while the decision flipped.]
- Code invariant: the wallet never authorizes an effect; the gate never performs one; the worker never judges its own work. [Components table]
- Enforcement sequence: `AuthorityCompiler.compile` → `VerifiedCommitLedger.execute_once` → non-COMMIT raises before the function runs. [L-C2]

## 5. Portable authority

- Wallet model: mandate_id, subject_id, Ed25519 subject key, scopes ≤32 (strict-subset narrowing), expires_at, predecessor/successor links, ACTIVE/SUPERSEDED/REVOKED/EXPIRED; revocation = root-signed event on a signed hash-chained timeline; gate enforces monotonic heads, quarantines forks. [Components table]
- What "portable" means here: the authority history is user-held and verifiable without calling the original provider [D3/L-A2]; the successor inherits nothing by default — cross-subject replacement is revoke-old + fresh grant [D12/L-A3].
- What it does NOT yet mean: cross-machine revocation propagation [P1]; production key custody (0600 explicitly insufficient — stated in L-A2); trust-root succession beyond the in-band fixture [P5].

## 6. Receipts

- What a receipt records: three wire families documented [Components table]; who signs (receiver's gate operator or the owner — the worker signs nothing).
- Receipts vs logs: the receiver produces its own decision evidence [D2]; the bureau validator mechanically rejects relabeled terminals [D14].
- Questions answered: who produces (receiver/owner, never worker); what is signed (decision + action + authority references); what survives provider replacement (the history — D3); independent checkability (independent verifier `verify-node.mjs` in L-E1; clean-room B in L-E2).
- Reopening: claim-graph recall — which accepted decisions must REOPEN when upstream evidence loses standing [L-D5, preliminary]; old decisions are reopenable, not rewritable (bureau anomaly preserved, not repaired — L-D4).
- Bound: PROVIDER-EFFECT-LIVE_001's negative — closure evidence is receiver-local unless the receiver exports it [L-D5-negative].

## 7. Implementation

- Concise component map (8 rows) with maturity labels — all experimental. [Components table]
- How they connect to the claim: wallet (owner-held authority) → worker (proposes) → gate (receiver decides) → receipts (portable evidence) → claim-graph/airlock (recall + independent acceptance).
- Not repository documentation: one paragraph per component max, always tied to the central claim.

## 8. Experimental method

- Frozen claim, preregistration, success/failure conditions, preserved evidence, frozen receipts. [Method visible across L-A1..L-E3]
- Why negatives retained: the bureau validator (D14); "source bytes are sacred" (L-D4 anomaly); the failures table (§10).
- Standing constraints: revocation-safety inequality [L-B7]; experiment-blur rule (containment ≠ continuation — L-B1); one-committed-transaction spirit (frozen IDs not rerun).

## 9. Experimental results — grouped by question

- **A. Authority surviving replacement:** D1/L-A1, D3/L-A2, L-A4, L-A5, D12/L-A3, D11/L-A6. Strongest composition: D1+D2.
- **B. Revocation and stopping:** D2/L-B1, D4/L-B2, D5/L-B5, D6/L-B6, D10/L-B7, D13/L-B8, L-B3, L-B4, L-C2. Caveat: STOP-arrival PARKED (simulated timing only).
- **C. Receiver enforcement:** D2/L-B1 (core), L-C2 (check inside the commit lock), D7/L-C3 (deferred authority at execution time), L-C4 (demo, not evaluated).
- **D. History surviving handoff:** D1 (digest unchanged), D3 (authentic but not current), D11 (byte-verifiable 10/10), D14 (honest labeling), L-D5 (recall, preliminary). Bound: PROVIDER-EFFECT negative.
- **E. Independent implementation:** L-E1 (INCOMPLETE — the contract gap; most informative), D9/L-E2 (repaired contract traveled), L-E3 (foreign boundary probe — mixed, honest).
- **F. External receiver acceptance:** U1/L-F1 — UNRESOLVED. Four substantive replies, zero adoptions. State this plainly and early in the section.

## 10. Failures and negative results (mandatory)

- Use "What changed because of failures" from `claim-table.md` (6 items).
- Full failures table reference (`experiment-evidence.md` §12).
- Apparatus failures distinguished from scientific failures (containment-001/002 vs 003; ACS-001 vs 002).
- This section strengthens credibility: the contract gap (L-E1) improved the documents; the 001 FAIL exposed a real mechanism gap; the bureau anomaly is preserved.

## 11. Related work

- Draw entirely from `related-work-map.md` (B): per-category summaries, the comparison table (8 columns), the distinctiveness verdict.
- Distinctiveness verdict: NO existing system combines all six properties (a)–(f) as of 2026-10-06. Closest: Google AP2 v0.2 (fails revocation — explicitly out of scope).
- Honest overlaps to name: x401's Verifier-issued Verification Token + PROOF-RESULT path (verifier-composed admission, not owner-defined mandates); LFDT Proof-of-Control (most portable evidence format surveyed — standardizes evidence, not authority); MCP Authorization (explicitly no delegation — token passthrough forbidden); A2A §7.6 (punts authorization semantics); obsigna (agent-signed; receiver does not attest).
- Sierra PAP: announced 2026-10-06, v0.1 not shipped — status only, no inference.
- No implication of copying in any direction.

## 12. Limits

- Experimental research software; all tests localhost/development fixtures unless stated.
- External adoption: zero [U1].
- Cryptographic evidence cannot prove facts never observed.
- Receipts don't stop every harmful action [D2's boundary; warning-window misses — EXP-38].
- Receiver enforcement only works at boundaries that actually use it [D13's scope; L-E3's ESCAPED paths].
- Portability has integration costs [P1, P2].
- Independent receiver acceptance is the critical unresolved test [U1].
- Evidentiary value of receipts in a real dispute is untested [U3].

## 13. Discussion

- What changes if agents become interchangeable but authority is not: D1+D12 suggest the durable layer is authority/history/acceptance, not the model.
- Analytical, not prophetic. No "OpenLine will become…".
- The portability/control layer must itself be replaceable — "make the worker replaceable without making the control layer the next landlord" (his thesis; attribute as his framing, not a finding).

## 14. Falsifiers

- If a mainstream system accepts independently issued portable authority across providers with revocation continuity and independently verifiable receiver decisions — the wedge is substantially replicated. (Sharper form from B: if a future AP2 revision adds revocation with continuity across provider replacement to its existing mandate+receipt structure, the distinctiveness claim narrows to non-payment domains or revocation/history semantics.)
- If authority cannot practically travel without recreating the original control system [test: L-E1/L-E2 pattern at larger scale].
- If receivers refuse outside authority in real deployments [U1 resolving negatively].
- If portability's security/operational costs prove unacceptable [P1/P2 resolving badly].
- If signed receipts prove to add little beyond normal audit logs in a real dispute [U3 resolving negatively].

## 15. Conclusion (narrow)

- What the experiments support today: AI capability and authority can be separated; an agent can propose an action while another system retains the right to decide whether that authority is acceptable [D1, D2].
- The remaining question: whether this pattern travels between truly independent systems at useful scale [U1].

## 16. Artifact and reproducibility appendix

- Index: repository, experiment ID, date, commit/tag/hash, result, evidence link, relevant paper claim — built from the ledger's L-entries.
- Note on local-vs-remote HEAD divergence: cite experiment-record SHAs, not checkout HEADs.
- Frozen receipts: artifact SHA-256s where recorded (L-A1: `96a55c65…`; L-A2: `281ae549…`; L-B3: `51c69f48dbd2484431d6a62c307f104d00fee77cc6a5ecacac901d473fe51b68`; L-B4: `bace02a6f08f4f15b780096ead4b5e76e3281873492e57924dbfe75ac5622be4`; L-A6: `777691bf…`).
