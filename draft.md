# Receiver-Owned Authority for AI Agents: Portable Mandates, Receiver Enforcement, and Verifiable Receipts

**Terrynce White** · v1.0 · 2026-10-07 · **FROZEN**

*This version is closed. Later developments — industry protocol releases, external receiver results, new corroboration — belong to a second paper, not to this record.*

*This is experimental research software, not a production platform. Every empirical claim below is traceable to the evidence ledger (`evidence-ledger.md`). Failures are preserved in §10. Unverifiable details are marked [VERIFY].*

---

## 1. Abstract

AI agents are beginning to request actions in outside systems — payments, deployments, purchases, data access — but the question of who validates an agent's authority to act remains unsettled. This paper examines a simple architectural idea: an agent can propose an action, but the system receiving that action should be able to decide whether it will accept the authority behind it. Authority should not have to live inside the model or the model provider. A person or organization should be able to define its own mandate, keep that authority when switching agents or providers, revoke it, and preserve a verifiable history of consequential decisions.

We describe OpenLine, an experimental research system implementing this idea: a user-held wallet carrying signed mandate and revocation history, a receiver-side gate that decides allow-or-refuse before any consequential effect, and signed receipts recording each decision. Across 45 recorded experiments (2026-09-04 to 2026-10-06), the strongest demonstrated results are: a real coding job continued across a Claude-to-Codex provider switch under a byte-identical owner-approved agreement with acceptance decided by a third party; a valid worker credential used to request an out-of-mandate action was refused by the receiver with a signed receipt and zero effects; and one owner-issued STOP prevented protected effects across independent receiver processes. A repaired receipt contract was implemented independently from documents alone.

The experiments establish that authority can be separated from the agent and survive replacement of the underlying provider while remaining subject to receiver-side enforcement. They do not yet establish that an unrelated receiver will accept authority originating outside its own control system. That is the next test — and the subject of a second paper, not a condition for this one.

---

## 2. Introduction: When agents begin acting

Models used to answer questions. Agents request actions. An agent acting for a person may ask a payment network to move money, ask a deployment system to ship code, ask a store to place an order, ask a database for records, ask a job runner to execute work, or hand a task to another agent. Each of these is a request made *to* some outside system — a receiver — that must decide what to do about it.

That decision currently rests on an unstable mixture of mechanisms: the agent's credentials, the provider's session, the receiving system's own access controls, and whatever logging happens afterward. When the agent changes — a different model, a different provider, a different vendor — the authority story usually has to be rebuilt from scratch, because the authority lived inside the provider's session rather than with the person.

This paper asks a narrower question than "how do we make agents safe." It asks: **when an agent asks another system to do something consequential, who gets to decide whether its authority is valid?** Our answer is architectural: the receiver should decide, against authority the owner defined and can carry across providers — and the decision should leave portable evidence.

---

## 3. The architectural problem

Five things are commonly conflated. They are different:

- **Agent capability:** can it perform the task?
- **Agent identity:** what agent is this?
- **Authority:** what has this agent actually been allowed to do?
- **Receiver acceptance:** will the system being asked to act recognize that authority?
- **Evidence:** can the parties later prove what was requested, allowed, refused, revoked, or changed?

Vendor logs alone do not necessarily solve the evidence problem. A log controlled by the agent's vendor is not the receiver's decision record. When a receiver refuses a request, the refusal needs to be the receiver's own signed statement — produced at decision time, checkable later, without calling back to the agent's provider. In our experiments, the receiver's signed STOP receipt for an out-of-mandate request was preserved in the owner's wallet history as the durable record of the refusal [L-B1]. A separate evidence-ledger prototype mechanically enforces honest terminal labeling: a FAIL relabeled as PASS fails validation [L-D4]. These are small, local demonstrations — but they illustrate the structural point: evidence the receiver produces itself is a different object than a vendor's log.

---

## 4. Design principle: the receiver decides

The OpenLine model, in plain language:

- The **principal** (a person or organization) defines the mandate: what is allowed, for whom, within what scope and time.
- The **agent** (the worker) proposes an action. It does not approve its own proposals.
- The **receiver** — the system being asked to act — checks the presented mandate against its own rules.
- The receiver **allows or refuses** the action, before the consequential effect.
- The system creates a **verifiable record** of the decision.

The receiver never has to blindly trust the agent. In one experiment, a worker kept a fully valid transport credential while an owner-side mandate revocation flipped the same consequential request from COMMIT-with-effect to DENY-with-no-effect: the transport stayed authenticated throughout, and the decision changed at the receiver's decision layer [L-B2].

Three invariants hold across the implementation: the wallet never authorizes an effect; the gate never performs one; the worker never judges its own work. The enforcement sequence in the receipt gate runs authority compilation, then a verified commit ledger that consumes a one-use authorization code, blocks replays, and performs the owner-standing final-authority check under a single lock — non-COMMIT decisions raise before the guarded function runs [L-C2].

---

## 5. Portable authority

The wallet is a local store of signed permission records, revocations, and action receipts, exportable as a JSON bundle a receiving service can verify without calling the original AI provider. A mandate carries an identifier, a subject identifier with an Ed25519 subject public key, a scope list (at most 32 entries, narrowable only by strict subset), an expiry, predecessor/successor links, and a status of ACTIVE, SUPERSEDED, REVOKED, or EXPIRED. Revocation is a root-signed event on a signed hash-chained timeline; the gate enforces monotonic heads and quarantines forks.

"Portable" here means three specific things, each demonstrated:

1. **The history is user-held and verifiable without the original provider.** In the provider-switch experiment, the wallet carried the user's authority history across a Claude-to-Codex switch; the receiver made each execution decision from that history [L-A2].
2. **Old authority can be authentic but no longer standing.** The old permission's signature stayed valid; the gate stopped it because newer history said revoked. A record can be genuine and still be too old to use [L-A2].
3. **The successor inherits nothing by default.** In the implementation, cross-subject provider replacement is revoke-old plus a fresh grant. There is no ambient inheritance [L-A3].

What "portable" does **not** yet mean: revocation does not propagate across machines in any tested component; production key custody is explicitly unsolved (file mode 0600 is not a defense against a malicious same-user agent — the record states this plainly [L-A2]); and trust-root succession has only been demonstrated in a bounded in-band fixture [L-B6].

Bundles expire after ten minutes by default, bounding stale-history exposure; receivers may require a shorter age for riskier actions.

---

## 6. Receipts

A receipt is the receiver's signed statement about a decision. Three wire families exist across the components: the wallet gate's `openline.gate.action_receipt.v1` (Ed25519 gate-signed, carrying gate identity, decision, action, and both decision and policy authority references); a lightweight envelope format with payload hashes and proofs; and the receipt gate's hash-chained format where integrity comes from the chain (parent hash to receipt hash) rather than a signature. In every family, the signer is the receiver's gate operator or the owner. The worker signs nothing.

Receipts differ from ordinary logs in two load-bearing ways. First, the producer is the party with the consequence at stake: the receiver that refused the request signs the refusal itself [L-B1]. Second, honest labeling is mechanically enforced: the evidence-ledger prototype's validator confirms that a terminal classification appears verbatim in the bound artifact, so a FAIL relabeled PASS fails validation [L-D4].

What survives provider replacement is the history: the agreement digest stayed byte-identical across the Claude-to-Codex switch [L-A1], and the pre-switch receiver receipt remained in the wallet bundle the successor presented [L-A2]. Old decisions can be reopened when upstream evidence loses standing — an evidence-recall engine identifies which accepted decisions must REOPEN (one small replication: 8/8 reopenings [L-D5], preliminary) — but they are reopened, not rewritten. When a frozen result's self-recorded digest did not reproduce from the present bytes, the discrepancy was preserved in the manifest's known limitations, not repaired: source bytes are sacred [L-D4].

One boundary must be stated: a live GitHub merge was once observed without a signed closure certificate — closure evidence did not survive outside the receiver that produced it [L-D5-negative]. Closure evidence is receiver-local unless the receiver exports it. The paper claims no more.

---

## 7. Implementation

Eight components, all experimental research software:

- **openline-wallet** — user-owned authority: signed mandate/revocation history plus the receiver gate reference implementation. Experimental v0.1 reference.
- **openline-receipt-gate** — the `@authorize` enforcement wrapper: the receiver decides COMMIT, QUARANTINE, or DENY before the guarded function runs. Late experimental; explicitly "not a hosted authorization service."
- **openline-airlock** — operator-owned admission gate for coding agents: model-blind acceptance, anti-self-dealing rules, signed receipts. Experimental, dogfooded research grade. In the APPROVED_JOB_LIVE_001 milestone, "Airlock decided" means the experiment pinned Airlock at `fb02207f` and ran its acceptance evaluation against the final candidate: ELIGIBLE means the candidate satisfied the owner-approved agreement's checks as evaluated by Airlock's own test-discovery machinery, not by the worker's self-report. Airlock's receipts are HMAC-SHA256 under a receiver-local key — they attest that this Airlock instance recorded the decision, not a cross-system identity claim. Airlock holds no worker-succession logic; the handoff itself was the experiment harness's work.
- **openline-claim-graph** — evidence-recall engine for reopening decisions when upstream evidence loses standing. Experimental.
- **openline-lite** — no-server toolkit, the "front door." Experimental.
- **openline-world** — a local developer-preview demo world that vendors the real wallet gate. Demonstration layer only.
- **openline-core** — early protocol reference, frozen and superseded by the wallet.
- **openline-otel** — signed receipts on OpenTelemetry traces plus an MCP `evidence.verify` proxy. MCP is consistently transport-only across the system, never the authority layer.

Each component connects to the central claim the same way: the wallet holds owner-defined authority, the worker proposes, the gate (whoever operates the receiver) decides, and receipts carry the decision as portable evidence.

---

## 8. Experimental method

Every experiment in the record follows the same discipline: a frozen claim, preregistered before contact; explicit success and failure conditions; preserved evidence including frozen receipts; and exact verdict words (PASS, FAIL, INCONCLUSIVE, INCOMPLETE, NO-GO, PARKED) that are never rewritten afterward. Negative and inconclusive results are retained because the method treats them as load-bearing: a qualification gate that catches its first failure and stops is working as designed [EXP-24]; a contract gap found by an independent implementation is more informative than a pass would have been [L-E1].

Two standing rules shaped the record. First, revocation is a race against consequence: the revocation-safety inequality requires the worst credible bound on detection, propagation, receiver, stop, and uncertainty intervals to sit below the irreversibility horizon, or the receiver must fail closed [L-B7]. Second, distinct experiments are never edited into one seamless event: the containment proof (a valid credential stopped at the receiver) and the continuation proof (one job across two providers) are different experiments with different claims [L-B1].

---

## 9. Experimental results

### A. Does authority survive worker/model/provider replacement?

Yes, in the tested configurations. A real Claude Code worker produced a verified partial checkpoint and made zero calls after handoff; a real Codex CLI worker continued from that exact checkpoint under a byte-identical owner-approved agreement; Airlock — not Codex — judged the continuation ELIGIBLE [L-A1]. Across one continuously running gate, Claude acted before a switch, the same Claude authority was stopped after revocation, and Codex continued under a successor mandate — with no provider credential transferred [L-A2]. Agent replacement under one owner-controlled regime completed with history intact and the superseded agent unable to regain authority [L-A4]. A frozen commission moved from GPT-4o-mini to Llama-4-Scout under an owner-signed replacement intent and settled exactly once [L-A5]. Mid-task handoff across staged frozen state succeeded 10/10 with byte-verifiable accepted state [L-A6].

### B. Can an owner revoke authority and stop later actions?

Yes, within the tested boundaries — with one critical precision. A deterministic process holding a worker's valid credential requested an out-of-mandate action; the receiver refused with a signed receipt and zero effects [L-B1]. Revocation flipped a live worker's requests from COMMIT to DENY without touching transport credentials [L-B2]. One owner STOP closed seven post-stop consequence-survival paths including stale receivers and in-flight work, verified by a separate auditor [L-B3], and the property reproduced in a second implementation built from the contract alone [L-B4]. One STOP propagated across independent receiver processes [L-B5]. In-band trust-root succession refused the superseded holder's STOP [L-B6]. No structural bypass of the receiver was found for the covered resource [L-B8].

The precision: the demonstrated rejection occurred when the revoked worker *next attempted a gated action*. Nothing in the record establishes cancellation of actions already underway, and the paper must never describe it as "enforced at revocation time" [L-A1]. Revocation protection is a race, governed by the temporal rule in [L-B7]; a parked benchmark prototype suggests receiver-enforced arming beats ordinary middleware, but all its timing is simulated [L-B8-caveat].

### C. Can the receiving side enforce the decision rather than trusting the worker?

The receiver refused a validly credentialed request because the mandate didn't cover the action [L-B1]; the final authority check runs inside the same lock as permission consumption so it cannot be raced [L-C2]; deferred authority was evaluated at execution time against origin lineage even after traveling through intermediaries [L-C3]. A prompt-injection demonstration shows the architecture's intended behavior — a hostile refund instruction stopped at the receiver — but it is a demonstration under owner review, not an evaluated defense [L-C4].

### D. Does history survive the handoff?

The agreement digest stayed byte-identical across a provider switch [L-A1]; old receipts stayed authentic while losing standing [L-A2]; ten consecutive handoffs produced byte-verifiable accepted state [L-A6]; and honest terminal labeling is mechanically enforced [L-D4]. Bounded by the negative: closure evidence is receiver-local unless exported [L-D5-negative].

### E. Can an independent implementation understand or enforce the contract?

Partially — and the partial result is the most informative in the program. An independent implementation built from documents alone reached matching dispositions on the grant path, but the revocation path exposed a genuine contract gap: the frozen profile had no pinned verdict/decision pairing for terminal DENY. The result was recorded as INCOMPLETE (contract gap), not FAIL — and the gap was then repaired in the public documents, after which a clean-room implementation reached full matching dispositions [L-E1, L-E2]. A probe of an unmodified foreign receiver library honestly reported mixed outcomes: some paths stopped, several ESCAPED — kill-switch semantics do not automatically hold at a foreign boundary [L-E3].

### F. Can a genuinely external receiver recognize authority it did not create?

**Unresolved.** Four substantive replies were received (OpenCodex's maintainer, AgentAdmit's founder, a Microsoft toolkit discussion participant, and a Proof-of-Control working-group member) and several lanes remain open, but no external organization has adopted the system, integrated it, or issued authority under it as of 2026-10-06 [L-F1]. This is the paper's load-bearing gap, and §12 and §14 treat it as such.

---

## 10. Failures and negative results

This section is part of the evidence, not an apology.

The independent-interop gap (E1) forced a repair in the public documents before any pass could be earned — the documents are part of the system, and the failure improved them. The first replace-worker run failed on a real mechanism gap: a revoked session still occupied its participant slot with no public API to retire it; the second run's `replace_worker` path closed it, and the FAIL stands unrewritten. Two containment runs died on provider strict-mode schema defects (apparatus), while the third ran end-to-end and failed scientifically under its frozen usefulness bar — apparatus failure and scientific failure are different verdicts and are recorded as such. The bounded-call qualification stopped at its first failure, which is the gate working as designed; its standing is NO-GO/REPAIR-ONLY. A fixed-authority governance design failed visibly on coordination tax at n=3. An incident-replay run failed on false holds. A warning-window audit found two semantic misses where genuine danger committed — the measured-lead subset is too thin to generalize from, and the record says so. A live GitHub merge was observed without a signed closure certificate, bounding the history claim. One handoff froze on an artifact conflict because the successor correctly refused to invent semantics. A frozen result's self-recorded digest does not reproduce from the present bytes; the discrepancy is preserved, not repaired.

What changed because of these failures: the interop documents were repaired; the worker-replacement mechanism gained an explicit path; the semantic-helper composition lane was ended without a positive terminal; the bound-authority build was stopped at its LOC ceiling and repriced rather than reinterpreted. The full failures table is in `experiment-evidence.md` §12.

---

## 11. Related work

A 2026 survey of current approaches — agent identity and authorization, OAuth-style delegation, MCP and agent interoperability, enterprise agent control, workload identity, policy gateways, verifiable intent, agent passports, notarized records, portable receipts — finds the landscape split into two halves that never meet [B].

**Owner-portable authority** systems (Google's AP2, UCAN, Biscuit, and related capability-token designs) produce signed, attenuable, offline-verifiable, provider-agnostic mandates — but revocation is expiry-based and nobody produces portable receiver decision records. **Receiver-decision** systems (Proof's x401, OPA/Cedar-style policy engines, MCP authorization servers, enterprise privileged-access controls) put a real decision point under the receiver's own policy — but the authority they enforce is the operator's or verifier's own, not an independently originated owner mandate.

The honest overlaps: x401's specification already defines a Verifier-issued Verification Token and a PROOF-RESULT path, so "the receiver verifies owner authority" alone is not the distinguishing claim — x401 is verifier-composed admission, not owner-defined mandates. The LFDT Proof-of-Control effort has the most portable evidence format surveyed, but it standardizes evidence, not authority or enforcement. MCP's authorization specification explicitly forbids token passthrough — servers must not accept tokens issued for other parties — which is the opposite of portable authority. A2A v1.0 deliberately punts authorization semantics: the protocol "does not define the scope, representation, validity, or revocation semantics of the authorization decision."

**Distinctiveness verdict:** no surveyed system combines all six of (a) owner-defined mandates, (b) receiver decision before effect, (c) survival across provider replacement, (d) revocation and history traveling with the authority, (e) portable decision evidence, and (f) the receiver keeping its own policy — as of 2026-10-06. The closest is Google's AP2 v0.2, whose User Credential model has user-signed mandates and independent verifiers returning signed accept/reject receipts — but revocation is explicitly out of scope in its specification. Sierra's Personal Agent Protocol was announced 2026-10-06 with v0.1 planned later in the month; it is recorded as announced-only, and no architecture is inferred from the announcement.

No copying is implied in any direction.

---

## 12. Limits

OpenLine is experimental research software. Nearly all testing occurred in localhost development fixtures. External adoption is zero (§9F). Cryptographic evidence cannot prove facts that were never observed. Receipts do not stop every harmful action — the warning-window audit recorded genuine dangers that committed despite semantic warnings. Receiver enforcement only works at boundaries that actually use it; a foreign-boundary probe showed several paths escaping. Portability has integration costs: cross-machine revocation propagation is undemonstrated, and production key custody is explicitly unsolved. The evidentiary value of signed receipts in a real dispute is untested — no real dispute has occurred. Independent receiver acceptance remains the critical unresolved test.

---

## 13. Discussion

If agents become interchangeable but authority does not, the durable layer of an agentic system is not the model — it is the mandates, the history, and the acceptance decisions. The experiments support a narrow version of this: the worker changed completely across providers while the agreement stayed byte-identical [L-A1], and the successor inherited nothing by default [L-A3].

There is a second-order point worth stating carefully, as framing rather than finding: the portability layer must itself be replaceable. If the system that carries your authority becomes the one place you cannot leave, it has recreated the landlord problem it set out to solve. Nothing in the current experiments tests this — it is a design constraint for whatever comes next, not a demonstrated property.

---

## 14. Falsifiers

The architectural claim would weaken or fail under any of the following, stated as checkable conditions:

1. **A mainstream system accepts independently issued portable authority across providers**, with revocation continuity and independently verifiable receiver decisions. Sharper form: if a future revision of Google's AP2 adds a revocation mechanism with continuity across provider replacement to its existing mandate-and-receipt structure, the distinctiveness claim narrows to non-payment domains or to revocation/history semantics specifically.
2. **Authority cannot practically travel without recreating the original control system** — if independent-implementation attempts at larger scale reproduce the interop-001 pattern (contract gaps on every new path) rather than the interop-002 pattern (document repair suffices).
3. **Receivers refuse outside authority in real deployments** — if the currently open external lanes (Proof-of-Control, the bank-paper follow-up, the RSA probe) resolve negatively: receivers willing to verify identity but unwilling to accept externally issued mandates.
4. **Portability's security or operational costs prove unacceptable** — if production key custody or cross-machine revocation turns out to require trusted infrastructure that defeats the point.
5. **Signed receipts prove to add little beyond normal audit logs** — if, in a real dispute, a receiver's signed decision record carries no more weight than the counterparties' existing logs.

---

## 15. Conclusion

The experiments establish that authority can be separated from the agent and survive replacement of the underlying provider while remaining subject to receiver-side enforcement. They do not yet establish that an unrelated receiver will accept authority originating outside its own control system. That is the next test.

---

## 16. Artifact and reproducibility appendix

| Repository | Experiment | Date | Commit / pin | Result | Paper claim |
|---|---|---|---|---|---|
| terryncew/openline-wallet | APPROVED_JOB_LIVE_001 | 2026-09-09 | head `7d4e8fe5`; Actions run 34413490635; artifact `96a55c65…` | PASS | §9A |
| terryncew/openline-wallet | PLATFORM_EXIT_LIVE_001 | 2026-09-04 | head `711ced88`; Actions run 33841050124; artifact `281ae549…` | PASS | §9A |
| terryncew/openline-wallet | CREDENTIAL_OVERREACH_LIVE_001 | 2026-09-12 | main `c7ac6a24`; Actions run 34725225216 (independently inspected 2026-10-07: success; artifact verdict `CREDENTIAL_OVERREACH_CONTAINMENT_ENFORCED`) | PASS | §9B, §9C |
| terryncew/openline-wallet | AUTHORITY_IN_TIME_001 | 2026-09-09 | commit `a5095ce` | PASS (fixture) | §9B |
| — (study-local: `muse-receiver-live-001/`) | MUSE-RECEIVER-LIVE-001 | 2026-09-19 | `FREEZE.md` terminal `PASS_MUSE_RECEIVER_LIVE_001`; Receipt Gate pin `c903d77eed55bdca06ef231f01ac70a6da38d9af` (read-only) | PASS | §9B |
| — (study-local: `bypass-001/`) | BYPASS-001 | 2026-09-19 | prereg freeze `312df6f80508e0ee472dda51247f271b7a7266d5`; `FREEZE_BYPASS_001_PASS.md` | PASS | §9B |
| terryncew/openline-wallet | TRUST-HANDOFF-001 | 2026-09-18 | wallet `687bf0d8`; gate `9c06dfd6` | PASS | §9C |
| terryncew/openline-receipt-gate | KILL-SWITCH-RECEIPT-INTEGRATION-001B | 2026-09-19 | main `a348930` (PR #92); integration `2061043` merged as PR #93 `d618ce8` | PASS | §9C |
| terryncew/openline-kill-switch | KILL-SWITCH-REFERENCE-001 | 2026-09-19 | evidence `51c69f48dbd2484431d6a62c307f104d00fee77cc6a5ecacac901d473fe51b68` | PASS | §9B |
| terryncew/openline-kill-switch | KILL-SWITCH-PORTABILITY-001 | 2026-09-19 | tag v0.1.0 `ebf2522`; evidence `bace02a6f08f4f15b780096ead4b5e76e3281873492e57924dbfe75ac5622be4` | PASS | §9B |
| — (study-local) | DISTRIBUTED-STOP-001 | 2026-09-19 | [VERIFY] | PASS | §9B |
| — (study-local) | TRUST-ROOT-SUCCESSION-001 / OWNER-CONTROL-002 | 2026-09-19/21 | PR #95 merge `41631b7` | PASS | §9B |
| — (study-local: `openline-receipt-gate/experiments/platform-exit-kill-002/`) | PLATFORM-EXIT-KILL-002 | 2026-09-19/21 | prereg `7e440cf5f6ddeb1e949476aceb8a74ddd3ddbadc86de36f2943b33f1b45ede38`; `TERMINAL.md` PASS | PASS | §9A |
| — (study-local) | INDEPENDENT-INTEROP-002 | 2026-09-19 | contract-repair `2f03346` | PASS | §9E |
| — (study-local) | interop-001 | 2026-09-19/20 | A-side pins in L-E1 | INCOMPLETE | §9E, §10 |
| terryncew/openline-handoff-state | HANDOFF-STATE-001 | 2026-10-04 | v0.1.2; state `777691bf…` | PASS 10/10 | §9A, §9D |
| terryncew/openline-world | WORLD-AUTHORITY-001 | 2026-10-04 | `51b202a` (repair `11c68bd`) | PASS_WITH_APPARATUS_QUALIFICATION | §9 (visualization) |
| terryncew/openline-world | OPENLINE-ATTACK-DEMO-001 | 2026-10-05 | PR #5 `3e8382b`, unmerged | Demo PASS (owner review) | §9C |

*Note: several study-local experiments ran in `~/workspace/` rather than in a public repo; their evidence lives in the workspace experiment directories and the dated memory logs cited in the ledger. Local checkout HEADs differ from GitHub default branches for some repos — citations above use the SHAs recorded in the experiment records.*

---

*End of first draft (verification pass 2026-10-07). Supporting: `evidence-ledger.md` (A), `related-work-map.md` (B), `claim-table.md` (C), `outline.md` (D), `citation-audit.md` (F), `verify-list.md` (G).*
