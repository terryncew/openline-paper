# OpenLine Repository / Component Inventory

**For the technical paper:** *Receiver-Owned Authority for AI Agents: Portable Mandates, Receiver Enforcement, and Verifiable Receipts* (Terrynce White).
**Inspection date:** 2026-10-06/07. Read-only; nothing modified.
**Standard:** primary sources only (code, READMEs, commits). Gaps marked [VERIFY].

---

## 1. openline-wallet

- **URL:** https://github.com/terryncew/openline-wallet
- **Purpose:** Python reference implementation of user-owned authority for AI agents: a wallet that stores signed mandate/revocation history, plus a receiver-side gate that decides each exact action against that history and returns a signed receipt. Replacing the model does not replace the authority history.
- **HEAD:** `b69c6fbaaef603015e6b01c0638559982e729846` (2026-10-03), branch `codex/implement-authority-continuity-experiment`, tree clean.
- **Key docs:**
  - `README.md` — "The model proposes, the receiver decides, and the proof travels." Three public proofs: PLATFORM_EXIT_LIVE_001 (real Claude host acted pre-switch, stopped after revocation, real Codex host continued under successor mandate against one localhost gate), APPROVED_JOB_LIVE_001 (Codex continued Claude's verified checkpoint; Airlock accepted without Claude's chat/credential), AUTHORITY_IN_TIME_001 (receiver admits revocation-protected work only when detect+propagate+receiver+stop+uncertainty bounds beat the consequence horizon). "A record can be genuine and still be too old to use."
  - `VISION.md` — explicitly **unproven** direction ("Status: unproven architectural direction"); self-rule: "Vision may suggest the next experiment. Only evidence moves a statement out of Vision." Has-not-demonstrated list: multi-party federation, public-network discovery/routing, trust between unrelated principals, institutional trust elimination, cross-machine revocation delivery, production key custody.
  - `TEMPORAL_AUTHORITY_PROFILE_v0.md` — draft profile: admission only when `consequence_horizon − (detect+propagation+receiver+stop+uncertainty)` > 0 with supported bounds. Three facts kept separate: revocation issued / observed / effects closed.
- **Mandate model (in code, `src/openline_wallet/wallet.py` + `receiver.py`):**
  - Fields: `mandate_id, subject_id, subject_public_key` (Ed25519 hex), `scopes` (≤32; narrowing requires strict subset), `expires_at` (≤ 366-day epoch cert), `predecessor_mandate_id`, derived `status` (ACTIVE/SUPERSEDED/REVOKED/EXPIRED), `issued_at`, `event_hash`, `revoked_at`, `revocation_reason`, `successor_mandate_id`.
  - Revocation: root-signed `MANDATE_REVOKED` event; events form signed hash-chained timeline (monotonic sequence, no time reversal). Gate enforces strictly monotonic head sequence, rejects stale heads, quarantines forks, refuses bundles older than freshness cap. A revoked mandate → STOPPED `MANDATE_REVOKED`, even if the old signature is still valid.
  - Expiry: mandate `expires_at`; exported bundle TTL ≤ 600 s (default 600 s); gate challenge TTL ≤ 300 s (default 60 s); bundle max age receiver-configurable per action risk.
  - Succession: `narrow()` requires same `subject_id` + same `subject_public_key`, strictly reduced scopes, never-extended expiry; predecessor SUPERSEDED with back-link. Cross-subject/provider replacement in code = revoke-old + fresh grant (B inherits nothing by default).
  - Provider replacement: deterministic demo + tests prove the state machine; real-host evidence (Claude Code 2.1.260 → Codex CLI 0.153.0, 2026-09-04) exists only in frozen proof artifacts, not code.
- **Connection to paper claim:** literal code separation — `WALLET_POLICY_AUTHORITY = "NONE"`, `DECISION_AUTHORITY = "RECEIVER_GATE"` baked into bundles/receipts; wallet never authorizes an effect, gate never performs one. Principal identity = long-lived Ed25519 root key; wallet is the continuity vessel across provider switch.
- **Maturity:** experimental. Self-described "v0.1 reference implementation… development environment." Explicit limits: unencrypted local keys, in-memory gate state (no durable restart recovery), no multi-device sync, no cross-machine revocation delivery, single signing key, exports reveal contained history.
- **[VERIFY]:** test suite not executed in this environment (package not installed; 19/20 test modules error at import — read-only constraint). Live-run hashes (GitHub Actions run 33841050124, evidence SHA-256 `281ae549…b7b`) read as frozen records, not independently re-verified.

## 2. openline-receipt-gate

- **URL:** https://github.com/terryncew/openline-receipt-gate
- **Purpose:** A Python library + CLI (`olp_gate`, v0.6.0rc6) that wraps consequential function calls in a receiver-owned authorization check — the agent proposes the call, the gate decides COMMIT / QUARANTINE / DENY before the function runs, and every decision is recorded in hash-chained / signed receipts.
- **HEAD:** `31070c80ad254284227ce04263138ad52cc04dc9` (2026-09-20), local branch `main`. Note: GitHub `main` has since advanced to `abc610be`; the findings below describe the inspected local HEAD.
- **Key docs:**
  - `README.md` — "Let the agent propose. Make the receiver decide." Before a protected function executes, the gate checks the exact call against: receiver-owned mandate + permission policy, current receiver state, fresh evidence from receiver-controlled providers, trusted receipt keys/source history, and local replay/one-use state. Sufficient evidence → executes; missing evidence → QUARANTINE; hard rule/integrity failure → DENY. The model cannot approve itself with a persuasive explanation. Not a hosted auth service or production certification.
  - `PORTABLE_STANDING_CLAIM.md` — frozen kill-switch claim: the existing owner-signed terminal standing (REVOKED / INACTIVE) can be consumed at the consequence boundary; a current owner STOP makes old worker authority refused at finalize without rewriting historical receipts. Single boundary only; ordering is mechanical (monotonic sequence numbers), not wall-clock.
  - `CLAIM_BOUNDARY.md` — explicit claim discipline: what the repo claims (COMMIT/QUARANTINE/DENY appraisal, Verified Commit, x402 Transaction Airlock on 56 frozen synthetic cases, frozen synthetic calibration) and what it rejects (its own contaminated v0.5.0rc3 calibration result; real-world predictive usefulness; production safety; universal thresholds). External predictive claims held at HOLD pending a preregistered label-blind benchmark.
- **Enforcement point** (`olp_gate/tool_adapter.py::authorize` → `AuthorityCompiler.compile` → `VerifiedCommitLedger.execute_once`):
  - The `@authorize` decorator wraps the guarded function; each call builds a `decision_proposal/v1` from frozen call arguments (rebuilt from hashed values so the caller can't mutate them), with `producer_id` from the mandate and `producer_model` labeled "untrusted-agent".
  - `AuthorityCompiler.compile` checks, in order: (1) producer identity vs mandate (`proposal_producer_not_mandated` → DENY); (2) mandate fit (`olp_gate/mandate.py`: expiry, principal/agent/purpose identity matches, action-type allowlist, target allowlist, disclosure-class lists, settlement/payment value caps, delegation rules) → DENY; (3) permission-policy route authorization → DENY; (4) receiver state + evidence from receiver-controlled providers (failures → QUARANTINE; integrity failures → DENY); (5) authorization TTL window (`authorization_window_closed` → QUARANTINE). A non-COMMIT_ELIGIBLE decision raises `AuthorizationBlocked` before the guarded function runs.
  - On COMMIT_ELIGIBLE, `VerifiedCommitLedger.check_and_consume`/`execute_once` verifies the Ed25519-signed decision receipt against receiver-pinned `trusted_gate_keys`, consumes the one-use code (replay blocked), checks expiry and exact action binding (tool/target/settings/run evidence hashes), runs a receiver preflight (mandate fit re-evaluated), and — where installed — a final-authority check: owner standing (REVOKED/MISSING/expired) is consulted inside the same lock as permission consumption (`stop_standing.py`), so a post-authorization owner STOP refuses the effect; fail-closed if standing can't be established. Only then is the executor invoked exactly once.
- **Receipt wire format:** two families.
  1. ReceiptGate receipts (`openline.receipt_gate.v0.1.1`, JSONL in `receipts/`): `schema, receipt_id, parent_hash, timestamp, action_type, claim, evidence_hash, result_hash, status, decision, policy_flags, next_use_note, metadata, receipt_hash`. Hash-chained (`parent_hash` → prior `receipt_hash`); integrity by chain verification, not signatures.
  2. Signed decision receipts (`proof_to_policy_decision_receipt` v0.4, `gateway.py::evaluate_request`): `kind, receipt_version, algorithm_id, canonicalization_id, spec_uri, issuer{id}, created_at, request_id, action{type, id, risk_level, claim_hash}, source{format, receipt_hashes, primary_hash, source_key_ids}, binding{run_id, session_id, sequence, challenge_nonce, parent_decision_hash, expected_source_hash}, policy{id, version, hash, snapshot}, assessments{per-check}, verdict, decision, commit_authorization, chain_accepted, reason_codes, privacy{…}`, plus `payload_hash` and `signature{algorithm:"Ed25519", public_key, value}` over canonical JSON.
  - **Who signs:** the receiver's gate operator (receiver `signing_key` + `issuer_id`; verification needs the receiver-pinned public key). Owner-signed artifacts: mandate authorizations (`mandate_owner.py`), capability issuance (`issuance.py`). The worker/agent signs nothing — untrusted producer id only.
- **Connection to paper claim:** the most complete enforcement instance in the inventory — proposal profile is literally `decision_proposal/v1`; policy, keys, state, and evidence callbacks are all receiver-supplied; the request cannot add keys or broaden its mandate; model output is forbidden in trust paths. Authority binds to mandate hashes, not the model (`model_swap.py` treats model replacement as the normal case); owner STOP → REVOKED standing refused at finalize under one serialization point while historical receipts are never rewritten.
- **Maturity:** late experimental / reference implementation. Self-described: "Release candidate and reference implementation"; "not a hosted authorization service or a production certification."

## 3. openline-airlock

- **URL:** https://github.com/terryncew/openline-airlock

- **Purpose:** An operator-owned admission gate for autonomous coding agents: any agent/model can propose code changes, but an independent, repo-defined acceptance pipeline (protected paths, the repo's own tests/lint/typecheck, evidence-sufficiency rules) decides what earns human attention — sealed with signed, hash-chained receipts. "The agent can change the code. It cannot change what counts as an improvement."

- **HEAD:** `d4c81e68d3cadb7b65aac95192203968e8fa0303` (2026-09-21), branch `study/rpi-001`. Working tree has untracked local run files; no uncommitted changes to tracked code.

- **Key docs:**
  - `README.md` — "Software that can earn its own improvements." The definition of "better" lives outside the agents — in your tests, protected paths, metrics, limits, promotion rules. No LLM judge: noise, ties, protected-file changes, regressions, or weak evidence stop the loop.
  - `docs/OPENLINE_LINEAGE.md` — three inherited ideas: receiver-owned admission (lineage: `terryncew/openline-receipt-gate` — "candidate capability is evidence, not authority"; the parent receiver decides whether the exact candidate commit earned admission); content-addressed evidence (receipts bind base commit, candidate commit, protected surface, config hash, command exits + output hashes; v0.1 uses receiver-local HMAC explicitly "without pretending the key is a cross-system identity primitive"); flat reverse evidence index (lineage: `terryncew/openline-lite`). Deliberately standalone, not a runtime dependency on the research repos.
  - `docs/CONTINUOUS_IMPROVEMENT.md` — compounding loop: one unique winner per generation compounds on an isolated `airlock/improve/<run-id>` branch; `main` never moves. Honest limits: a scalar is not total product value; self-reported cost preserved as unknown; worktree isolation is not a hostile-code sandbox.

- **Accept/refuse logic** (`run_tournament`, `src/airlock/runner.py`):
  - **Who decides, for whom:** Airlock (the receiver harness) decides, on authority delegated by the operator who authored `.airlock/config.json` (Starter Rules, protected paths, verification commands, objective). It decides for the repository's maintainer/operator which candidate patches earn human review or promotion. The agent is the proposer only.
  - **Inputs per candidate:** branch integrity (worker on assigned `airlock/…/candidate-NN` branch?), changed paths vs base commit, exact commands + exit codes from repo-owned checks, protected-path patterns, sha256 fingerprints of protected files, config hash. Model name is recorded metadata only — plays no role in the decision.
  - **Pipeline:** baseline must be green (`BASELINE_NOT_GREEN` stops before any agent spend) → `BLOCKED` reasons (branch-integrity failure, no patch, protected-paths-changed, target-command failure, lint/typecheck failure, test failure) → `NEEDS_EVIDENCE` sufficiency check (`sieve.py`: repo-owned acceptance evidence resolvable, changed modules have baseline-test references) → `SURVIVED` → exactly 1 survivor = `READY`; >1 = `MULTIPLE_SURVIVORS` (refuses to invent a winner); 0 = `NO_PATCH_READY`.
  - **Anti-self-dealing:** candidate changes to project-control files (`pyproject.toml`, `package.json`, CI workflows) fail closed; judge files restored from base commit before execution (`freeze_judge_files`); uncommitted objective changes can't redefine "better" at runtime; only repo-owned quality commands through a safe-argv allowlist (no shell composition); side-effecting commands (dirty `git status`) fail.
  - **Outputs:** run records (`airlock.run.v1`, swarm, improvement schemas), HMAC-SHA256-signed receipts binding base_commit, candidate_commit, changed_paths, protected_patterns, protected-file fingerprint, config hash, per-command exit codes + stdout/stderr sha256s. `verify_offline` revalidates signature, git-diff boundary, protected paths, base fingerprint, command artifact integrity — no candidate code rerun needed.

- **Handoffs / worker replacement:** no mid-task worker-succession logic exists in this repo (the repo-root `*_HANDOFF.json` files are session run-context artifacts, not agent succession). What exists is **model interchangeability at the proposal layer**: `run_tournament` assigns candidate *i* to `models[i % len(models)]` (adapters: claude-code, codex, aider, opencode, hermes), and admission is model-blind. In the compounding loop only the winning *commit* becomes the next generation's base; the worker behind it is discarded. The paper's succession concept (one worker continuing another's exact checkpoint) is not implemented here.

- **Connection to paper claim:** literal "agent proposes, receiver decides" — agents propose patches; the receiver (Airlock + operator-authored rules + repo's own tests) decides admission. Authority defined by the person/org (objective, Starter Rules, protected paths — enforced against agent modification). Survives provider replacement by construction (model-blind admission; provider credentials scrubbed from worker environments). Revocation: partial — failed/weak candidates disappear into machine history; no explicit mandate-revocation primitive. Receipts are deliberately receiver-local HMAC — they prove what happened at this receiver, not cross-system authority.

- **Maturity:** experimental / research-grade, actively dogfooded. v0.3 "frozen" schemas with backward-readability rule (versioned terminology, not a stability guarantee); heavy dogfood/experiment machinery; README disclaims hostile-code sandboxing, cross-system authority, and production-readiness. Current branch is `study/rpi-001`.

- **[VERIFY]:** memory records "Airlock (not Codex) decided whether the continuation passed" in APPROVED_JOB_LIVE_001, but this repo has no mid-task handoff/continuation logic — could not confirm whether that milestone used this repo's code, a different harness, or an uncommitted flow. Test suite not executed (findings from code + test names).

## 4. openline-claim-graph

- **URL:** https://github.com/terryncew/openline-claim-graph
- **Purpose:** Traces which accepted claims/decisions must be reconsidered when upstream evidence changes — a deterministic "blast radius" engine over receiver-admitted dependency graphs. The model may propose relations; the receiver decides which are HARD/ADVISORY/unadmitted.
- **HEAD:** `30a3ee1316d0b388fd4f20fa49e25556fef1985b` (2026-09-13), branch `main`.
- **Key docs:** `README.md` (stable release 0.5.2; frozen Evidence Recall caught 8/8 warranted reopenings, 42.85% review-load reduction, zero additional misses — "small historical replication, not broad-domain proof"); `PROTOCOL.md` (frozen ERC-001 question with source-time split and anti-hindsight rule); `IDENTITY_BINDING_GATE_001.md` (receiver-owned identity-binding gate: provenance authenticates bytes, not identity; same-name-only evidence quarantines as UNRESOLVED_IDENTITY); `HUMAN_CONTRACT.md` (POINT/BECAUSE/BUT/SO constrained outputs).
- **Connection to paper claim:** the "evidence changes → what must reopen" half of verifiable history. Answers the paper's receipt question "can old decisions be reopened when evidence changes or disappears" — with a demonstrated, narrowly-scoped mechanism (evidence recall), not a general truth engine.
- **Maturity:** experimental with one promoted stable result (0.5.2 PROMOTION verdict under a predeclared rule). Standing Recall explicitly "experimental and has not been promoted into the stable production API."
- **[VERIFY]:** did not re-run the 0.5.2 replication or verify the reported 8/8 and 42.85% figures from raw artifacts.

## 5. openline-lite

- **URL:** https://github.com/terryncew/openline-lite
- **Purpose:** "The front door" — no-server Python toolkit: `openline-check` (may this action proceed under my receiver policy?), `openline-impact` (which standing decisions reopen when evidence is invalidated), `olp-lite` (verify chains, bounded handoffs, benchmark). A producer's `verified` flag is never trusted.
- **HEAD:** `9700413a1399fa610c1253f109986b7baf407e42` (2026-09-20), branch `main`.
- **Key docs:** `README.md`, `START_HERE.md` (connects Lite/Airlock/Wallet/Receipt Gate), `TRUST_BOUNDARY.md` (impact pack format), `MODEL_HANDOFF.md` (verified model handoff: next model receives only receiver-verified facts + file hashes, never the prior model's prose as truth; COMMIT/QUARANTINE/DENY), `ARCHITECTURE.md`.
- **Wire format (in code, `openline_lite/wire.py`):** envelopes `{payload, payload_sha256, proof{alg, key_id, public_key, signature}}`; `olp.source.v1` fields: schema, issuer, issued_at, run_id, sequence, parent_hash, action{type,name,target}, claim, evidence, extensions; `olp.decision.v1` fields: schema, gate_id, issued_at, source_format, source_sha256, policy, verdict, decision, reason_codes, side_effect_observed, assessments. `ReceiptGate.decide()` (in `gate.py`) assesses integrity/provenance/normalization/policy/freshness/evidence/claim_support and emits a receiver-signed decision receipt.
- **Mandate model:** `mandate_ledger.py` — "The conservation namespace is the mandate, never the actor identity." One mandate, one authoritative ledger, fixed capacity; operations violating the invariant are refused before mutation (LedgerRefused), leaving state unchanged.
- **Connection to paper claim:** the minimal receiver-side enforcement point in library form; the decision receipt (`olp.decision.v1`, signed by the gate) is the portable evidence unit; `mapped_adapter.py` maps foreign signed objects into the gate's normalized payload (producer `verified` never trusted; key pinned externally).
- **Maturity:** experimental. 14 test modules present.
- **[VERIFY]:** did not run the test suite; did not verify PyPI publication status beyond `pip index` returning no distribution.

## 6. openline-world

- **URL:** https://github.com/terryncew/openline-world
- **Purpose:** Local developer-preview "shared world" (3D workshop diorama + backend) where AI agents act only through receiver-owned authorization — the demonstration layer for the already-proved handoff authority/continuity property. "Bring your agent. Keep your rules."
- **HEAD:** `05250b5db6c9d4fe40d055b1932a0b5867e2f97c` (2026-10-06), detached HEAD (film branch work).
- **Key docs:** `README.md` (local dev preview; simulated funds; "Nothing here is a hosted service, an audit verdict, or proof that any real system is safe." Latest prerelease v0.1.3 with SHA-256-pinned ZIP); `DECISION.md` (2026-09-25 reuse decision: authorization uses the real `openline_wallet` package — `Wallet.grant/revoke`, `EffectGate.admit_bundle/issue_challenge/evaluate`, Ed25519-signed gate receipts; "The workshop never mints a receipt itself — only the gate's signed decisions become receipts").
- **Enforcement:** `backend/workshop_gate.py` — owner wallet + real receiver gate (`workshop-receiver`); mandate grants, bundle admission, per-action challenge/presentation/evaluation, revocation, signed ALLOWED/STOPPED receipts. Backend never mints receipts.
- **Connection to paper claim:** the visible demonstration of receiver enforcement (e.g. the recorded `deploy:staging` refusal, STOPPED / ACTION_OUTSIDE_MANDATE with zero staging effects). Per standing rule, the world reads authoritative OpenLine/Handoff state; it is not another research program.
- **Maturity:** demo / developer preview. Localhost-only, simulated funds.
- **[VERIFY]:** did not run the world or re-verify the v0.1.3 ZIP SHA-256.

## 7. openline-core (GitHub reference)

- **URL:** https://github.com/terryncew/openline-core
- **Purpose:** Early research reference for the OpenLine Protocol: tiny server that takes "frames" of reasoning and returns a compact digest + telemetry JSON (schema-first, Pydantic). Now superseded — README repoints to openline-wallet as the current stack ("COLE no longer public").
- **HEAD:** `2ddb152f9fff704c33ed606e1dabf6048b651777` (2026-10-01), branch `main`. Public (restored 2026-09-30).
- **Connection to paper claim:** historical — the protocol's earliest public reference point; no current enforcement role.
- **Maturity:** frozen / superseded reference.

## 8. openline-otel (evidence gateway + MCP surface)

- **URL:** https://github.com/terryncew/openline-otel (local copy at `~/workspace/openline-cleanup/openline-otel`)
- **Purpose:** Adds portable, signed OpenLine receipts to OpenTelemetry traces; runs beside existing span processors. v0.2.0 adds an Evidence Gateway that accepts an OLP Canon receipt or Agent Receipts v0.5 wire profile, preserves submitted bytes in a local wallet, and emits a separate signed verdict (integrity, provenance, coverage, freshness, evidence sufficiency, independently witnessed outcome, causal uptake — each `verified`/`rejected`/`undecidable`).
- **HEAD:** `f8c05bf933c725aa1d108aebe7b2030867277dfb` (2026-09-30).
- **SDK/agent integrations found:**
  - `src/openline_otel/mcp_proxy.py` — stdio MCP proxy exposing an `evidence.verify` tool (the verifier as an MCP server). MCP is transport only.
  - openline-wallet: `mcp_bridge.py` (MCP sidecar presenting user-owned authority to a receiver gate; subject private key stays in the user-controlled process), `mcp_egress.py` (receiver-owned MCP egress gate for consequential tool calls — "local/development shim, not a hosted policy service"), `gate_http.py` (development-only localhost receiver).
  - openline-lite: `integrations/living-wiki/` (COMPATIBILITY.md, SKILL.md), `mapped_adapter.py` (foreign signed-object adapter).
  - openline-receipt-gate: `olp_gate/tool_adapter.py`, `olp_gate/integrations/`, `x402_airlock.py` (x402 payments surface).
- **Connection to paper claim:** the interop surfaces — how foreign/agent-side systems present evidence to a receiver without the receiver trusting the agent's self-assertion.
- **Maturity:** experimental (v0.2.0). Installed via git URL, not on PyPI (pip lookup found no distribution) — [VERIFY] whether any release was ever published to PyPI.
- **[VERIFY]:** local copy lives under `openline-cleanup/`; did not confirm it matches the GitHub repo's current HEAD file-for-file.

---

## Component-connection diagram (plain text)

```
Owner / person
  │  defines mandate, holds root Ed25519 key
  ▼
openline-wallet (Wallet)
  │  stores signed mandate / revocation / receipt history
  │  exports signed bundle (TTL ≤ 600 s)
  ▼
Agent / worker (Claude, Codex, …)
  │  proposes exact action; presents bundle + holder proof
  │  (MCP sidecar: mcp_bridge.py / mcp_proxy.py — transport only)
  ▼
Receiver-side gate  ←──  receiver's own policy (allowed actions, freshness, evidence rules)
  │   openline-wallet EffectGate / gate_http.py (dev)
  │   openline-lite ReceiptGate (olp.decision.v1)
  │   openline-receipt-gate olp_gate (openline.receipt_gate.v0.1.1)
  │   openline-world workshop-receiver (vendored wallet gate)
  │
  ├─ ALLOWED ──► effect happens ──► signed receipt (gate-signed, Ed25519)
  └─ STOPPED ──► reason code (e.g. MANDATE_REVOKED, ACTION_OUTSIDE_MANDATE)
                       │
                       ▼
              Receipts: portable evidence
                - wallet gate:   openline.gate.action_receipt.v1 (Ed25519, gate-signed)
                - lite:          olp.decision.v1 (envelope: payload + payload_sha256 + proof)
                - receipt-gate:  openline.receipt_gate.v0.1.1 (hash-chained: parent_hash → receipt_hash)
                - otel:          verdict attached to OTel traces; local wallet copy
                       │
                       ▼
              openline-claim-graph — later: if evidence loses standing,
              compute which accepted decisions must REOPEN (REOPEN/RETAIN/UNDETERMINED)
                       │
                       ▼
              openline-airlock — independent acceptance of worker output:
              worker proposes code, Airlock's rerun tests + promotion rules decide;
              accepted generations committed with signed hash-chained receipts
```

Key invariant across components: **the wallet never authorizes an effect; the gate never performs one; the worker never judges its own work.**

---

## [VERIFY] list (could not confirm from primary sources)

- [VERIFY] openline-wallet: full test suite not executed (package uninstallable in read-only env); live-run hashes read as frozen records only.
- [VERIFY] openline-receipt-gate: test suite not executed (findings from code + test names); README demo outcomes read from documentation, not from an observed run. Remote `main` (`abc610be`) is ahead of the inventoried local HEAD — newer remote commits may supersede these findings.
- [VERIFY] openline-airlock: test suite not executed (findings from code + test names). Untracked local run files not audited. HMAC key lifecycle (rotation, backup, multi-operator) not reviewed.
- [VERIFY] openline-claim-graph: reported 0.5.2 replication figures (8/8, 42.85%) not re-verified from raw artifacts.
- [VERIFY] openline-lite: test suite not run; PyPI publication status unconfirmed (`pip index` found nothing).
- [VERIFY] openline-otel: local copy under `openline-cleanup/` not diffed against GitHub HEAD; no PyPI distribution found — publication status unknown.
- [VERIFY] openline-world: v0.1.3 ZIP SHA-256 not re-verified; world not executed.
- [VERIFY] Local clone HEADs vs GitHub default-branch HEADs (checked 2026-10-07): in sync — openline-claim-graph (`30a3ee13`), openline-otel (`f8c05bf9`). Differ — openline-wallet (local `b69c6fba` on branch `codex/implement-authority-continuity-experiment` vs GitHub main `43997748`), openline-receipt-gate (local `31070c80` vs `abc610be`), openline-airlock (local `d4c81e68` on branch `study/rpi-001` vs `ac7da750`), openline-lite (local `9700413a` vs `a33955b7`), openline-world (local `05250b5d` vs master `706c2e69`). Inventory describes the local checkouts as inspected; GitHub-side newer commits were not reviewed.
- [VERIFY] Trust-root succession: wallet subagent notes AC-09 marks it unproven in that run; a separately frozen TRUST-ROOT-SUCCESSION-001 was referenced but not located in this inventory.
- [VERIFY] Cross-machine revocation delivery: explicitly listed as not demonstrated in wallet VISION.md; no component in this inventory implements it.
