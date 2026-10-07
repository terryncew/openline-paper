# Related-Work Map — Receiver-Owned Authority for AI Agents

**Deliverable B** · 2026-10-06 · Research pass over current (2026) primary sources.

Method: read-only survey of specs, official docs, GitHub repos, official announcements, and original papers. Secondary reporting used only where noted. Capabilities are marked **SHIPPED** (spec/code/docs publicly available) or **ANNOUNCED** (publicly committed, not yet available). Items that could not be confirmed from a primary source are marked **[VERIFY]**. This map does not favor any system and implies no copying in any direction.

The distinctiveness question under test: does any existing system already combine ALL of —
(a) the person/organization defines the mandate,
(b) the receiving system makes the final decision before the consequential effect,
(c) authority survives switching the underlying agent/model/provider,
(d) revocation and history travel with that authority,
(e) decisions create portable evidence,
(f) the receiving system does not surrender its own policy control?

---

## 1. MCP / agent interoperability — auth stories

**MCP Authorization** (SHIPPED; spec draft candidate 2026-07-28, stable line 2025-11-25). Authorization is OPTIONAL for MCP implementations. For HTTP transports that implement it, the client is an OAuth 2.1 client and the MCP server an OAuth 2.1 resource server: servers publish RFC 9728 Protected Resource Metadata, validate audience-bound tokens (RFC 8707 `resource`), and answer 401/403 with `WWW-Authenticate`. Client registration prefers CIMD (draft-ietf-oauth-client-id-metadata-document). There is **no delegation mechanism**: the spec forbids token passthrough — servers MUST NOT accept tokens issued for other parties or forward the client's token upstream. Scope step-up (`insufficient_scope`) narrows privilege per session; it is not delegation.
Primary: https://modelcontextprotocol.io/specification/draft/basic/authorization (read 2026-10-06). Stable-line text [VERIFY].

**MCP Elicitation** (SHIPPED, draft). Lets a server request structured user input mid-request (form mode) or redirect the user to an external URL (URL mode, for auth/payment flows where data must not transit the client). Responses are accept/decline/cancel; elicitations MUST be bound to client+user identity. A consent channel, not a delegation protocol: no mandate, no chained authority, no records beyond ephemeral request state.
Primary: https://modelcontextprotocol.io/specification/draft/client/elicitation

**MCP Enterprise-Managed Authorization (EMA)** (SHIPPED; promoted preview→stable 2026-06-18). Opt-in extension. The enterprise IdP issues an Identity Assertion JWT Authorization Grant (ID-JAG, draft-ietf-oauth-identity-assertion-authz-grant) via RFC 8693 token exchange of the user's IdP token; the client presents the ID-JAG to the MCP server's authorization server (RFC 7523 JWT bearer) for an audience-bound MCP access token. First shipped use of RFC 8693 token exchange inside the MCP ecosystem — but delegation happens at token issuance, not at the tool-call layer. Revocation behavior in the normative doc [VERIFY].
Primary: https://modelcontextprotocol.io/extensions/auth/enterprise-managed-authorization · https://github.com/modelcontextprotocol/ext-auth/blob/main/specification/stable/enterprise-managed-authorization.mdx

**A2A v1.0.0** (SHIPPED; Linux Foundation, released 2026-03-12). Agents are treated as ordinary enterprise applications. Auth schemes declared in the AgentCard `securitySchemes`; credentials obtained out-of-band; v1.0 modernized OAuth 2.0 flows and added multi-tenancy. Agent Cards MAY be JWS-signed (JCS/RFC 8785) — a signature, not a delegation artifact. Authorization is deliberately punted: §7.5 has the server authorize "based on the authenticated identity and its own policies"; §7.6 `TASK_STATE_AUTH_REQUIRED` allows in-task authorization requests chainable across agents, but §7.6.4 states the protocol "does not define the scope, representation, validity, or revocation semantics of the authorization decision or credential." No portable authority object, no decision receipts.
Primary: https://a2a-protocol.org/latest/specification/ · https://github.com/a2aproject/A2A/releases/tag/v1.0.0

Net: MCP and A2A both authenticate the client application and leave authorization to the receiving server's own policy. Neither defines portable, provider-independent authority records or revocation semantics for agent actions. MCP's spec actively prohibits the naive delegation pattern (token passthrough).

---

## 2. OAuth-style delegation applied to agents

**RFC 8693 — OAuth 2.0 Token Exchange** (IETF Proposed Standard, 2020-01). STS endpoint: present `subject_token` (+ optional `actor_token`, audience, scope) to get a new audience-bound token. Defines the `act` (actor) claim for delegation chains, nested `act` for earlier actors, `may_act` for pre-authorization. The workhorse composed into 2026 agent stacks (MCP EMA, KYAPay, transaction-tokens-for-agents). Defines delegation semantics for tokens, not agent identity.
Primary: https://datatracker.ietf.org/doc/html/rfc8693

**draft-araut-oauth-transaction-tokens-for-agents-02** (ACTIVE; 2026-05-22, expires 2026-11-23). Extends OAuth Transaction Tokens with an `agentic_ctx` claim: immutable identity layer (`sub` = principal, `act` = agent per RFC 8693) + mutable context layer (`current_actor`, `originator`, `hop_count`, workload identifiers). A Transaction Token Service replaces the token at each agent transition; monotonic trust attenuation, loop prevention, per-hop chain metadata. Short-lived signed JWTs; each service makes its own access decisions from claims. Delegation-by-token-replacement with auditable chains. Revocation semantics not addressed in text reviewed [VERIFY].
Primary: https://datatracker.ietf.org/doc/draft-araut-oauth-transaction-tokens-for-agents/ (full text read 2026-10-06)

**draft-klrc-aiagent-auth-03 (AIMS)** (ACTIVE individual draft; authors span Okta, AWS, Zscaler, Ping, OpenAI). Profiles WIMSE + OAuth for agents: agents get WIMSE workload identities (SPIFFE SVIDs, short-lived X.509/JWT); delegation via authorization code (user-delegated), client credentials (autonomous), JWT bearer; access-token `client_id` = agent, `sub` = delegated user; Transaction Tokens for downscoped hops; CIBA for human confirmation; Shared Signals Framework for revocation. Individual draft, not WG-adopted; expires 2027-01-07. Framework only — no shipped implementation defined.
Primary: https://datatracker.ietf.org/doc/draft-klrc-aiagent-auth/

**AAuth — draft-hardt-oauth-aauth-protocol** (individual draft, exploratory). Agent Authorization Grant (device-flow-inspired: polling/SSE/WebSocket) for confidential agent clients, five resource-access modes, HTTP Message Signatures for key discovery, agent governance as an orthogonal layer. Working demo (Keycloak, Agentgateway, A2A, MCP) claimed in secondary sources [VERIFY].
Primary: https://dickhardt.github.io/AAuth/draft-hardt-oauth-aauth-protocol.html · https://github.com/dickhardt/AAuth

**draft-oauth-ai-agents-on-behalf-of-user** (EXPIRED 2025-11-09, never adopted). Introduced `requested_actor` and `actor_token` for user→client→agent delegation chains. Historical only.
Primary: https://datatracker.ietf.org/doc/draft-oauth-ai-agents-on-behalf-of-user/

**Skyfire KYAPay drafts** (2026-07, expire 2027-01-20; draft-skyfire-oauth-kyapay-token + draft-skyfire-oauth-kyapay-token-exchange). KYAPay tokens identify agent, principal, payment capabilities, authorized scope/mission, audience; an STS/IdP exchanges them for standard OAuth access tokens so a service can distinguish agentic from human-present sessions. "Early production deployments" claimed in the draft, not independently verified [VERIFY].
Primary: https://datatracker.ietf.org/doc/draft-skyfire-oauth-kyapay-token-exchange/

Net: the dominant 2026 delegation primitive is token exchange + chained claims (`act`/`may_act`, `agentic_ctx`), not credential sharing. These produce audience-bound, short-lived tokens whose chains are inspectable — but revocation is expiry/AS-side, and there is no receiver-issued decision record.

---

## 3. Workload identity systems

**SPIFFE/SPIRE** (SHIPPED; SPIRE v1.15.3 published 2026-08-21; spec at spiffe.io). SPIFFE IDs (`spiffe://trust-domain/path`), SVIDs (X.509 or JWT, exactly one SPIFFE ID each), trust bundles, cross-domain federation via a standard HTTPS bundle endpoint. Keys/certificates are "short lived, rotated frequently and automatically." Answers *who this workload is*; encodes nothing about *what it may do*. Independent verification is built in: any relying party holding the trust bundle verifies an SVID without contacting the issuer. Revocation is by expiry/rotation and registration-entry deletion; no per-credential CRL/OCSP mechanism found in the spec pages read [VERIFY].
Agent-specific 2026 usage is all vendor/third-party, none in the SPIFFE spec: Red Hat Kagenti (SVIDs exchanged for OAuth2 tokens for an MCP gateway), Vijil Dome, NetFoundry (SVIDs authenticating agents to MCP servers over mTLS), agentrust-io ADR-0009 (SPIFFE URIs as canonical agent/issuer identifiers).
Primary: https://spiffe.io/docs/latest/spiffe-about/spiffe-concepts/ (accessed 2026-10-06) · https://github.com/spiffe

**Cloud workload identity** (SHIPPED): AWS IAM Roles Anywhere (X.509 + trust anchor + profile → 15-min–12-h role credentials; CRL import; docs published 2026-10-01), GCP Workload Identity Federation (external IdP token → STS exchange → short-lived Google token; attribute-mapped), Azure workload identity federation (federated identity credentials on app registrations/managed identities). All three establish who the workload is *for that provider's own APIs*; authority is defined by the account/project/tenant owner via IAM and enforced at that provider's boundary. None encodes an agent mandate or a grant a third-party receiver can verify.
Primary: https://docs.aws.amazon.com/rolesanywhere/latest/APIReference/Welcome.html · https://docs.cloud.google.com/iam/docs/workload-identities · https://learn.microsoft.com/entra/workload-id/workload-identity-federation

Net: workload identity is cryptographically solid, independently verifiable identity — but identity is not authority. Every system surveyed answers "who" and leaves "what may it do on whose behalf" to a separate, operator-owned policy layer.

---

## 4. Enterprise agent identity / control systems

**Microsoft Agent Governance Toolkit (AGT)** (SHIPPED; open-sourced 2026-04-02, MIT; v5.0.0 ~June 2026). Seven-package runtime-security stack: per-call policy enforcement (YAML/OPA-Rego/Cedar) between agent and tool execution, Ed25519 + ML-DSA-65 agent identity (SPIFFE-compatible), trust scores with decay, four-tier execution rings with kill switches, append-only hash-chained audit logs, adapters for 20+ frameworks. Shipped OPA/Cedar policy backends (commit 2026-03-16; repo self-marks shipped). Enforcement is inside the operator's deployment; audit logs are operator-held, not a wire protocol.
Primary: Microsoft Open Source Blog 2026-04-02 · https://github.com/microsoft/agent-governance-toolkit

**Agent Control Standard (ACS)** (SHIPPED spec + early reference implementations; launched 2026-05-27; MIT; donated to OWASP GenAI Security Project ~Sept 2026). Vendor-agnostic runtime governance: standardized middleware hooks (input, tool call, plan→execution, memory write, code exec, sub-agent spawn) with inline allow/deny/modify verdicts; workstreams for OpenTelemetry conventions, Agent Bill of Materials, MCP/A2A integrations, agent identity (ephemeral credentials, JIT access). Model/framework-agnostic by design; no external-verifier protocol; broad multi-vendor adoption not yet demonstrated.
Primary: BusinessWire 2026-05-27 · https://agentcontrolstandard.ai · https://github.com/Agent-Control-Standard/ACS

**CSAI Foundation: AARM + Agentic Trust Framework** (SHIPPED specs; 2026-04-29). AARM (contributed by Vanta) is "an open system specification for securing AI-driven actions at runtime across context, policy, intent, and behavior." ATF (v0.9.1, CC BY 4.0) applies Zero Trust (NIST 800-207 lineage) to agents across identity, behavior, data governance, segmentation, incident response. Explicit Proof-of-Control relationship: "Enforcement (what an agent may do) is CSA AARM's half; Proof-of-Control is the evidence half."
Primary: CSAI Foundation announcement 2026-04-29 · https://github.com/massivescale-ai/agentic-trust-framework

**Delinea × StrongDM** (SHIPPED product; acquisition closed 2026-03-05). Combined PAM platform: unified identity security control plane (Iris AI) covering admins, developers, non-human identities, AI agents; just-in-time runtime authorization "at the moment of action"; zero-standing-privilege model; ephemeral credentials. Authority evaluated per action at runtime — but inside the platform; no external-verifier protocol, no portable receipts.
Primary: GlobeNewswire 2026-03-05

**Okta / Auth0 for AI Agents** (SHIPPED; Auth0 GA 2025-11-19; Okta for AI Agents GA reported 2026-04-30 [VERIFY]; Agent SSO GA reported 2026-08-24 [VERIFY]). User Authentication for agents, Token Vault (agents never see refresh tokens), async human approval via CIBA, fine-grained authorization for RAG; agent registry, governance workflows, kill switch. Cross App Access (XAA), an open OAuth extension for agent-to-app authorization — reported adopted as MCP's Enterprise-Managed Authorization extension [VERIFY from primary]. Agent SSO registers agents as first-class identities with short-lived governed tokens instead of static API keys. GA dates from secondary research [VERIFY against Okta primary docs].

**Microsoft Entra Agent ID** (GA reported 2026-04; details [VERIFY]). Agent identities as special service principals, OAuth 2.0 + MCP + A2A, Conditional Access templates for autonomous vs on-behalf-of agents, sponsor lifecycle workflows. Details from secondary research [VERIFY from MS Learn].

**Skyfire KYA / KYAPay** (SHIPPED; docs updated 2026-07-14). Signed JWTs (types `kya`, `pay`, `kya-pay`) binding human identity (`hid`), agent platform (`apd`), agent identity (`aid`), `scope`; seller verifies via JWKS at `https://app.skyfire.xyz/.well-known/jwks.json`; presented in `kyapay-token` header; `pay` tokens carry a committed payment value charged post-delivery. Verification requires a human-started paid subscription; token creation fails if the seller's required assurance level is unmet. Any seller verifies the JWT locally — issuer pinned to app.skyfire.xyz. Identity "track record" is Skyfire-hosted, not portable. KYAPay submitted as an IETF draft.
Primary: Skyfire docs (2026-07-14)

**Visa Trusted Agent Protocol (TAP)** (SHIPPED spec; announced 2025-10-14; commercial launch Q1 2026). Three-signature trust model: (1) RFC 9421 HTTP Message Signatures for agent recognition (`created`/`expires` ≤ 8 min, `keyid`, Ed25519/PS256, nonce, tag `agent-browser-auth`/`agent-payer-auth`); (2) linked signed consumer/device identity (OIDC JWT, nonce-bound); (3) linked signed payment container. Merchant verification: tag check, field check, timestamp window, nonce replay cache, public key via JWKS Key Store. Payment Instructions API (consumer sets preferences/spending limits) and Payment Signals API (agent sends purchase signal; token unlocked only on match against pre-approved instructions). Sept 2026: Visa, Mastercard, Ant International agreed to develop shared Know Your Agent interoperability — ANNOUNCED, no timeline, no merchant pilots.
Primary: https://developer.visa.com/capabilities/trusted-agent-protocol/trusted-agent-protocol-specifications

**RSA Agent ID** (ANNOUNCED 2026-09-29; Discover+Secure GA 2026-11-16; Govern H1 2027). Agent/MCP-server inventory as first-class identities with named owner, risk tier, lifecycle state; per-call policy enforcement at an RSA-hosted or self-deployed AI/MCP gateway; high-risk actions approved out-of-band by a named, authenticated operator with a phishing-resistant credential. Enforcement records stream to customer SIEM; no portable receipt format specified.
Primary: BusinessWire 2026-09-29 — pre-GA, announced only.

**CyberArk → Idira (Palo Alto Networks)** (acquisition closed 2026-02-11; "Secure AI Agents Solution" announced 2025-04). Incumbent-PAM agent play: identity-first security, privileged access, lifecycles, orchestration of agents. Less protocol detail than Delinea/StrongDM. Rebrand to Idira (May 2026) [VERIFY].
Primary: BusinessWire via newshub.medianet.com.au (2025-04)

**Cisco MCP proxy in Secure Access** (RSAC 2026; secondary only [VERIFY]). Network-layer enforcement of agent→tool calls; agent identities via extended Duo IAM, each linked to an accountable human owner.

**Radware Agent Trust Management** (announced 2026-09-29; secondary [VERIFY]). Site-operator control over visiting outside agents: captures declared intent, behavioral consistency checks, fine-grained control (view but not purchase).

Net: the enterprise lane converges on per-call, identity-bound, operator-owned enforcement (PAM lineage + OAuth + gateways). All of it is receiver/operator-side: the receiver keeps its policy, but the authority being enforced is the operator's own policy over its own agents — not portable owner-defined mandates verifiable by third parties.

---

## 5. Policy gateways / policy enforcement points

**OPA** (SHIPPED; CNCF graduated; v1.14.0 2026-02 per secondary [VERIFY]). A policy *decision* engine that "decouples policy decision-making from policy enforcement" — the calling software (the PEP) enforces. Policies in Rego, distributed as signed versioned bundles. Decision Logs plugin emits structured records (decision_id, input, result) to a configurable endpoint — portable JSON, but unsigned and operator-emitted, not a verifiable receipt. No revocation primitive (OPA issues no credentials); authority changes by updating bundles. Policy external to the model, so model swaps change nothing — by design.
Primary: https://www.openpolicyagent.org/docs/latest/

**Cedar / Amazon Verified Permissions** (SHIPPED). Cedar: typed permit/forbid language over principal/action/resource/context with a formally specified evaluator; policy templates with `?principal`/`?resource` placeholders — template-linked policies instantiate grants for principal/resource pairs, template updates propagate automatically. AVP (Cedar-as-a-service): `IsAuthorized` API returns Allow/Deny plus the list of policies that produced the decision; the calling application must enforce. CloudTrail logs management events and (with a configured trail) `IsAuthorized` calls — AWS-scoped, unsigned, operator-owned. (The OCSF 99001 per-evaluation audit events in AWS's agent sample are emitted by the sample's own evaluator Lambda, not by AVP/Cedar — do not attribute that format to Cedar.)
Primary: https://docs.cedarpolicy.com/policies/templates.html · https://docs.aws.amazon.com/verifiedpermissions/latest/apireference/API_IsAuthorized.html

**Agent-specific PEPs** (2026): microsoft/agent-governance-toolkit ships OPA/Cedar backends (SHIPPED, commit 2026-03-16); Strands harness-sdk Cedar design (team/designs/0006) — thorough design doc for a Cedar plugin intercepting tool calls with user identity propagated through multi-agent delegation (Graph/Swarm); status is **design/proposal**, shipped implementation [VERIFY]; aws-samples/sample-cedar-agentic-ai-authorization (~2026-07, SHIPPED reference implementation) — three Cedar policy layers (agent→tool, agent→agent delegation with ≤5-hop limit, originating-user authorization), HMAC-SHA256 context-integrity envelopes + RFC 8693 on-behalf-of delegation, per-evaluation OCSF 99001 events emitted by the sample's evaluator; MCP-layer gateways positioned as enforcement points (Kong AI MCP Proxy 3.12+, Traefik Hub MCP Gateway v3.20 EA 2026-03-16 with Task-Based Access Control — GA ANNOUNCED not verified, Docker MCP Gateway v0.43.3, Azure APIM + Entra patterns).
Primary: https://github.com/microsoft/agent-governance-toolkit/commit/ff7af1896ba809f9910229b6c1dc17175aa5a7d1 · https://github.com/strands-agents/harness-sdk/blob/HEAD/team/designs/0006-cedar-authorization.md · https://github.com/aws-samples/sample-cedar-agentic-ai-authorization

Net: policy gateways give the receiver a clean decision point with its own policy — but decisions are local, records are operator-owned and unsigned, and no system surveyed produces a portable, independently verifiable decision receipt. Re-evaluation, not verification.

---

## 6. Agent control planes and verifiable-control efforts

**Proof x401 (HTTP Proof Requirement Protocol)** (SHIPPED draft; spec 0.2.0 latest, 0.1.0 versioned; Status: Draft). A Verifier protecting a resource returns a `PROOF-REQUEST` header carrying a Verifier-composed W3C Digital Credentials API request (OpenID4VP + DCQL, recommended signed so the Verifier is the relying party). The agent relays it via a Credential Manager (wallet), presents the Result Artifact in `PROOF-RESPONSE`, and gets authorization feedback in `PROOF-RESULT` (including error objects). On success the Verifier MAY issue a **Verification Token**: short-lived (SHOULD), scoped to Verifier audience + route/policy/action/resource (MUST), MUST expire, SHOULD support unique ID + replay detection + revocation; JWT claims deployment-specific (`iss` = Verifier, `sub` = Agent, `aud`, `exp`, `iat`, `jti`, `x401_request_id`, `x401_satisfied_requirements`). The Verifier composes the proof requirement with `trusted_authorities` issuer constraints and "remains authoritative for issuer trust enforcement"; the credential issuer (human's wallet/bank) asserts the underlying claims. Authority does not freely transfer: token reuse across routes only under explicit conditions. Payment explicitly out of scope (HTTP 402 stays separate). Editors: Daniel Buchner (Proof), Bhushit Agarwal (Circle); reviewers from Lightspark, MATTR, Okta, OpenAI, Proof, Visa.
Primary: https://x401.proof.com/spec/latest/ (0.2.0, Draft) · https://github.com/proof/x401

**Proof-of-Control (Advanced AI Society / LF Decentralized Trust)** (SHIPPED working draft; v1.0, 127 requirements across 10 chapters, Apache 2.0). Core mechanism: an Action Interception Gateway + evidence pipeline producing signed, hash-chained, anchored evidence; capability-bound dispatch; four Verifiability Tiers (Assertion / Attestation / Trust-minimized / Self-Enforcing) with a binary procurement threshold at Tier 3+. Ships a working reference implementation (22 correctness tests, 11 attacks, benchmarks, 16 signed test vectors, CDDL/JSON-Schema evidence profiles) measured on Intel TDX. Public comment open through **2026-10-30**. Explicitly the *evidence* half, not enforcement — "Not runtime enforcement" (that's CSA AARM's half). Standardizes what evidence must be producible and how it is graded — not who may act. Evidence designed for independent verification ("without trusting the operator") — the most portable record format in this survey.
Primary: https://github.com/LFDT-ProofOfControl/ov-poc-standard · https://advancedaisociety.org/proof-of-control

**EMVCo Agentic Payments — Framework for Specifications** (SHIPPED draft; released 2026-09-01; public comment closed 2026-09-30). Card-based payments where a consumer delegates purchase authority to an AI agent. Central proposal: **Intent Services** — a shared, interoperable layer for registering, referencing, retrieving, and managing consumer-authorized intent before, during, and after transactions (recurring purchases, cumulative budgets, post-transaction activity), complementing cryptographic assurance like Verifiable Intent. Also explores Know Your Agent checks and Agentic Transaction Indicators. A framework-for-specifications, not a protocol spec; developed with FIDO Alliance, OpenID Foundation, OpenWallet Foundation, W3C input.
Primary: emvco.com announcement 2026-09-01

**Six-bank "Building Trust in Agentic Commerce"** (2026-09-22; ASB Bank, Bank of America, Capital One, Commonwealth Bank of Australia, ING, NatWest). Five **voluntary, non-binding** principles: transparency, safety, privacy and data, choice, interoperability. Asks providers to preserve evidence of consumer instructions, authentication, intent, transaction decisions and outcomes — including warnings and interventions — as an auditable record for scam investigation, money recovery, and dispute resolution. Explicitly does not settle liability ("when an agent's transaction goes wrong, the allocation of liability is unclear"). Banks plan a second paper on implementation; paper PDF not retrieved [VERIFY clause detail].
Primary: joint paper 2026-09-22 · https://www.pymnts.com coverage 2026-09-22

**CSAI "agentic control plane"** (four-phase rollout 2026-06–2027-12). Frames AARM as the runtime-enforcement standard for the "agentic control plane." Open spec, not a product.

Net: x401 is the closest serious external-receiver seam in this lane — an independent Verifier composes requirements, validates presentations cryptographically, returns structured authorization feedback, and can issue scoped revocable tokens — but the authority is verifier-composed (admission), not an owner-defined mandate, and tokens are narrowly scoped to the verifier's routes. Proof-of-Control is the strongest portable-evidence format — but it standardizes evidence, not authority or enforcement.

---

## 7. Verifiable intent / mandate systems

**Google Agent Payments Protocol (AP2) v0.2** (SHIPPED; spec + Python/Go SDKs + samples; unveiled 2025-09-16 with 60+ partners; v0.2 released and donated to FIDO Alliance 2026-04-28 — FIDO Agentic Authentication TWG and Payments TWG ongoing; Apache-2.0; repo read at main commit e1ea56d 2026-10-06). Two mandate types in open and closed forms: **Checkout Mandate** (`mandate.checkout.open.1` / `mandate.checkout.1`, verified by Merchant — merchant cryptographic proof the agent may buy the assembled checkout) and **Payment Mandate** (`mandate.payment.open.1` / `mandate.payment.1`, verified by Credential Provider, Network, Merchant Payment Processor). Format: SD-JWT VCs with `vct` claims; ES256; selective disclosure; `cnf`-bound agent keys; open mandates carry constraints (amount ranges, allowed payees/instruments, execution dates, budgets, recurrence); closed mandates bind to the open mandate via `sd_hash` and to the checkout via `transaction_id` (hash of a merchant-signed Checkout JWT). Five roles: Shopping Agent, Credential Provider, Merchant, Merchant Payment Processor, Trusted Surface (non-agentic UI that gets informed user consent and signs mandates). Modes: Human Present (user signs closed mandates on a Trusted Surface) and Human Not Present (user signs open mandates including the agent's public key as `cnf`; RECOMMENDED minimal `exp`; agent closes mandates itself with its agent key). Two trust models: (a) **User Credential** (OpenID4VP with `transaction_data` delegate payloads; the Verifier trusts the credential *Issuer* — "a single User Credential [can] delegate Mandates to many different Agents, without the Verifier needing to have an explicit trust relationship with each Agent"); (b) **Trusted Agent Provider** (the agent provider's trusted surface signs mandates; the Verifier trusts the provider directly). "The Mandate Content itself functions independent of the means used to establish trust in its integrity." **Agent-to-agent delegation is explicitly out of scope of v0.2.** **Revocation: no revocation protocol in the spec** (confirmed in implementation_considerations.md — agent SHOULD manage active mandates and limit duration; management of delegated mandates by external Trusted Surfaces "is outside the scope of this specification"). Decision records: Merchant MUST return a signed **Checkout Receipt** (accept or reject); MPP MUST return a signed **Payment Receipt** to SA, CP, and possibly networks — mandates chain into a "non-repudiable picture of the transaction" usable as dispute evidence (dispute mechanics out of scope). Under the User Credential model, mandates are self-contained signed SD-JWTs verifiable by any party holding the issuer's keys — portable across agents and providers; mandates name amounts, payees, constraints, and agent keys, never a model or provider. No production-deployment evidence found in primary sources; PyPI package "will be published at a later time."
Primary: https://github.com/google-agentic-commerce/AP2 (docs/ap2/ all read 2026-10-06) · https://blog.google/products-and-platforms/platforms/google-pay/agent-payments-protocol-fido-alliance/ (2026-04-28) · https://ap2-protocol.org/

**Mastercard Verifiable Intent** (SHIPPED draft; v0.1 dated 2026-02-18; donated to FIDO Alliance 2026-04-28). Co-developed with Google. SD-JWT layered delegation chain: identity → intent → action; ES256; issuer keys via JWKS; selective disclosure; built on FIDO/EMVCo/IETF/W3C. Primary spec at verifiableintent.dev not directly read [VERIFY details].
Primary: https://verifiableintent.dev [unread] · https://www.paymentsjournal.com coverage 2026-04

**Sierra / Meta Personal Agent Protocol** (ANNOUNCED 2026-10-06 ONLY). Open standard "that defines how personal agents interact with businesses," built with Meta, Genesys, Instinct, Rocket, Shopify, Stripe, Walmart. Third-party summary: an open OAuth-based standard letting personal agents authenticate with businesses, carry context across channels, operate via website, MCP/OpenAPI, or a company-owned agent; **v0.1 spec planned for later in October 2026; payments and granular permissions listed as future extensions.** Nothing else is public. Do not infer architecture from the announcement.
Primary: sierra.ai/blog "Introducing Personal Agent Protocol" (2026-10-06)

**Stripe/OpenAI Agentic Commerce Protocol (ACP)** (SHIPPED beta; latest stable spec 2026-04-17; Apache-2.0). Merchant checkout API. Proof of authorization = **Shared Payment Token**: scoped, merchant-and-cart-total-bound single-use token; agent pays without holding buyer credentials. `rfc.delegate_payment` defines a delegated vault token with Allowance constraints (`reason: one_time`, `max_amount`, currency, `checkout_session_id`, `merchant_id`, `expires_at`). Bearer + optional `Signature` over canonical JSON, TLS 1.3. Delegation-as-scoped-credential-minting: authority defined and enforced by the merchant/PSP that mints the token; expiry-based, no revocation protocol; auditability "remains on the merchant's rails."
Primary: https://github.com/agentic-commerce-protocol/agentic-commerce-protocol (rfcs read; status Draft; repo badge beta)

Net: AP2 is the only system reviewed whose mandate objects are (a) signed by the authorizing party rather than the agent, (b) independently verifiable by receivers without issuer contact, and (c) portable across agents/providers under its User Credential model — while explicitly lacking revocation and explicitly scoping out agent-to-agent delegation in v0.2.

---

## 8. Agent passports / delegation records

**DigiCert AI Trust Manager** (SHIPPED; GA 2026-09-15, DigiCert ONE). **AI Passport**: portable, cryptographically signed credential verifying agent identity and binding it to an accountable owner. **Policy-based "visas"**: define which systems, data, and actions the agent may access, and for how long. Credentials "travel with the agent, allowing another organisation to verify its identity and permissions **without using the same identity provider**"; the receiving org **can still deny access** in its own environment; automated **kill switch** blocks prohibited actions and revokes authority when trust changes. Wire format of passport/visa not disclosed in sources read [VERIFY]; cross-org verification demonstrated vs documented [VERIFY].
Primary: GlobeNewswire 2026-09-15 · https://www.engineering.com/digicert-adds-ai-agent-identity-to-nvidia-safety-platform/ (2026-09-30)

**Cubitrek Agent Passport** (SHIPPED draft; v0.1, 2026-04-28, MIT). Business-issued signed JSON at `/.well-known/agent-passport.json`; Ed25519; DNS-bound trust (DNS-over-HTTPS verifier); TS reference verifier; three signed worked examples. Claims: who is contacting, what the agent is authorized to do, spend ceiling, when a human takes over, how the conversation is audited. Receiving operator verifies issuer, checks spend ceiling, routes to human at threshold, logs signed conversation. Authority source = issuing business. Revocation story not seen [VERIFY].
Primary: https://github.com/Cubitrek/agent-passport (spec/agent-passport-v0.1.md read 2026-10-06)

**ERC-8004 "Trustless Agents"** (draft EIP; registries reported live Jan 2026). Three on-chain registries: Identity (ERC-721 NFT, tokenId = agentId, tokenURI → registration file), Reputation (structured feedback), Validation (independent checks: re-execution, stake-secured, TEE). Issuer = whoever registers (owner controls token; transferable). Anyone verifies on-chain. Not an authorization format: identity + trust signals only; receivers enforce their own policy.
Primary: https://github.com/moltline/moltwiki/blob/HEAD/pages/ERC-8004.md · https://www.allium.so/blog/onchain-ai-identity-what-erc-8004-unlocks-for-agent-infrastructure/ (2026-09)

**ZeroID (Highflame, 2026-04)** (secondary only [VERIFY primary]). Reported: OAuth 2.1 + RFC 8693 token exchange + SPIFFE + OpenID Shared Signals; issues VCs; per-agent persistent identity; delegation chains; time-scoped credentials + real-time revocation (claimed).
Primary: third-party GitHub deep-dive 2026-06 only.

**AIP — Agent Identity Protocol** (paper + IETF draft; reference implementations SHIPPED). Sunil Prakash, "AIP: Agent Identity Protocol for Verifiable Delegation Across MCP and A2A," arXiv:2603.24775 (2026-03); IETF draft-prakash-aip-00 (2026-03-27). **Invocation-Bound Capability Tokens (IBCTs)** fuse identity, attenuated authorization, provenance into one append-only token chain. Two wire formats: compact (signed JWT, single-hop) and chained (Biscuit token with Datalog policies, multi-hop delegation). Holder-side attenuation only; transport bindings across MCP/A2A/HTTP; provenance-oriented completion records; Python + Rust reference implementations with cross-language interop; 340–380 bytes per delegation block, sub-ms verification. Expiry + holder-side attenuation; explicit revocation [VERIFY].
Primary: https://arxiv.org/pdf/2603.24775v1.pdf · IETF draft-prakash-aip-00

---

## 9. Notarized / receiver-attested agent action records

**obsigna Agent Receipts** (SHIPPED; four pinned spec versions; Go/TS/Python SDKs with CI-enforced cross-verification). An Agent Receipt = **W3C Verifiable Credential** (vc-data-model-2.0), Ed25519-signed, recording one agent action: action (standardized taxonomy), principal (who authorized), issuer (which agent performed), outcome (success/failure, reversibility, undo method), SHA-256 hash chain to previous receipt, parameters hashed (never plaintext). **Who signs: the issuer — the agent side** (optionally via an out-of-process signing daemon so a compromised agent can't forge past receipts). Verifier = any consumer, offline-capable. **The receiver does NOT attest**: no receiver countersignature; external "Anchor" (commit chain state to a content-addressed store) is an optional extension, not in the base protocol. This is agent-signed tamper-evidence, not receiver notarization.
Primary: https://github.com/agent-receipts/obsigna (spec/README.md read 2026-10-06)

**hraedon/agent-provenance** (SHIPPED skeleton; FROZEN 2026-10-04, maintenance only). "CloudTrail for agent actions, vendor-independent": agent harnesses emit signed tool-call events (HMAC-SHA256 over RFC 8785 canonical JSON; Ed25519 via regista BC-196), with first-class delegation-chain fields (`on_behalf_of`: principal_id, session_id, authenticated_at, scope, expires_at). Receiver/third-party attestation was planned as **witness federation** — periodic Merkle-root co-signatures by N independent parties (auditor, customer, third party); infra available, integration pending at freeze. Notably, **RFC 3161 trusted timestamping was implemented then deleted outright** (Merkle tree witnessed uuid.bytes, not content) — no trusted-time guarantee offered; OpenTimestamps anchoring never implemented. Explicit scope: audit layer, not enforcement layer; missing events undefended by design.
Primary: https://github.com/hraedon/agent-provenance (README read 2026-10-06)

**NVIDIA Open Agent Safety Platform / OpenShell** (ANNOUNCED ~2026-09-30; secondary only). OpenShell places infrastructure-level controls outside the agent's process, permits nothing by default, and **records every decision to allow or deny an action** — enforcement-side decision logging (receiver-side record). Paired with DigiCert AI Trust Manager. Treat as announced, not verified shipped.
Primary: https://www.engineering.com/digicert-adds-ai-agent-identity-to-nvidia-safety-platform/ (2026-09-30; secondary)

**No dedicated 2026 notarization service for agent actions found.** Generic RFC 3161 timestamping authorities exist (DigiCert, GlobalSign, Actalis) but are not agent-targeted. The closest agent-specific mechanisms are optional external anchors (obsigna) and unimplemented witness federation (agent-provenance).

Net: the "notarized record" lane is thin. Existing receipt systems are agent-signed (self-attested), not receiver-attested. No 2026 system was found where the receiving side countersigns the agent's action record as a matter of protocol.

---

## 10. Portable receipt / delegation-evidence formats

**UCAN (User Controlled Authorization Networks)** (ucan.xyz; 1.0-rc — not yet a settled standard). Delegated capability tokens: user (issuer DID) delegates attenuated capability to agent DID; chains (each hop restates/narrows); attenuation-only. Verification is self-contained: any party verifies offline with the token + proof chain — no issuer callback. DID-native. **No in-protocol revocation mechanism** — pure signed claims; deployments rely on short expiry + issuer-side revocation lists (known revocation-freshness pitfall) [VERIFY any 2026 revocation profile]. Chains bind to DIDs/keys, not to any model or provider — survives model/provider replacement by construction. Active 2026 agent use: gitlawb (owner→agent delegation), AgentBnB ADR-020, Covia, Vers.
Primary: https://github.com/covia-ai/covia/blob/HEAD/venue/docs/UCAN.md (2026-03) · https://github.com/xiaoher-c/agentbnb/blob/HEAD/docs/adr/020-ucan-token.md (2026-04-01). ucan.xyz not directly read [VERIFY].

**Biscuit** (v3; Eclipse Foundation; biscuitsec.org). Ed25519 chained signed blocks; authority block carries grant as Datalog facts; attenuation blocks can only add checks, never widen. Verifiable by anyone holding the issuer's public key — works across process/machine boundaries (subagents, A2A). Each block carries a revocation ID intended for receiver-maintained blocklists (revocation-by-indirection, not in-token); no receiver-side blocklist standard confirmed [VERIFY]. Receiver adds its own Datalog policies at authorization time (token facts + receiver policy evaluated together) — the receiver keeps its policy by construction. Agent-runtime use: gravital-steward RFC-0005 (2026-06-15).
Primary: https://github.com/gravital-cloud/gravital-steward/blob/HEAD/docs/rfc/RFC-0005-steward-auth.md · https://github.com/senthil1216/attenuate-agent. biscuitsec.org not directly read [VERIFY].

**Macaroons** (Birgisson et al., NDSS 2014; Google). Bearer credentials built on nested chained MACs (HMAC); caveats attenuate and contextually confine use; any holder appends caveats offline (append-only). Third-party caveats + discharge macaroons = canonical revocation-by-indirection / third-party approval mechanism. In production ~a decade (all of LND's RPC auth). Verification is symmetric-key: outsiders verify only if they share/trust the root secret (unlike Biscuit's public-key verifiability). No 2026 agent-specific deployment primary source found [VERIFY].

**x402** (v2; Linux Foundation x402 Foundation, operational 2026-07-14). HTTP 402 payment handshake + on-chain settlement (EIP-3009 signatures); ~40 org members. Payment-request/settlement format, not a mandate or delegation-evidence format. Primary spec not read [VERIFY].

Net: UCAN and Biscuit are the cleanest general-purpose portable delegation formats — issuer-signed, attenuable, offline-verifiable, provider-agnostic. Both lack in-protocol revocation and neither carries receiver decision records.

---

## Comparison table

Legend: Y = yes (primary-source supported) · P = partial · N = no · ? = could not verify · "ann" = announced-only.

### MCP / interop and OAuth delegation

| System | Who defines authority? | Who enforces? | Survives model/provider switch? | Outside receiver can verify? | Decision records portable? | Revocable? | Receiver keeps own policy? |
|---|---|---|---|---|---|---|---|
| MCP Authorization (OAuth 2.1 profile) | External OAuth AS | MCP server, per request | Y (client app, not model) | Token only; no records | N — none exist | OAuth-standard only | Y |
| MCP Elicitation | Human user (consent) | MCP server | N/A (consent channel) | N/A (ephemeral) | N | Y (decline/cancel) | Y |
| MCP EMA extension (stable 2026-06-18) | Enterprise IdP | MCP server + IdP | Y | Token only | N | ? (normative text unread) | Y |
| A2A v1.0.0 (2026-03-12) | Receiving agent's own model (§7.5) | A2A server | Y | AgentCard JWS only | N — §7.6.4 leaves undefined | Implementation-defined | Y |
| RFC 8693 Token Exchange | AS/STS minting token | Resource server (`act`/`may_act`) | Y | Y (JWT + `act` chain) | P — chain travels in token | Expiry / AS-side | Y |
| draft-araut transaction-tokens-for-agents-02 | Transaction Token Service | Each service in call graph | Y | Y (chain metadata inspectable) | P — chain in token | ? (short-lived; no protocol) | Y |
| draft-klrc-aiagent-auth-03 (AIMS) | WIMSE attestation + OAuth AS | Resource servers | Y (SPIFFE SVID) | Y | P — claims in tokens | Y (short-lived + Shared Signals) | Y |
| AAuth (draft-hardt) | AS after agent-authenticated consent | Resource servers | Designed Y | Would be | ? | ? | Y |

### Workload identity and policy gateways

| System | Who defines authority? | Who enforces? | Survives model/provider switch? | Outside receiver can verify? | Decision records portable? | Revocable? | Receiver keeps own policy? |
|---|---|---|---|---|---|---|---|
| SPIFFE/SPIRE (v1.15.3) | SPIRE operator (IDs only — not authority) | Relying party + its own policy | Model Y; provider P (re-federation) | Y (trust bundle) | N — identity only | Expiry/rotation/entry deletion | Y |
| AWS IAM Roles Anywhere | AWS account owner (IAM) | AWS service APIs | Model Y; provider N | AWS-only | N (CloudTrail, AWS-scoped) | Y (CRL, deletion, expiry) | N/A (receiver is AWS) |
| GCP Workload Identity Federation | GCP project IAM admins | Google Cloud APIs | Model Y; provider N | GCP-only | N (audit logs, GCP-scoped) | Y | N/A |
| Azure workload identity federation | Entra tenant admin | Entra/Azure RBAC | Model Y; provider N | Entra-only | N (tenant logs) | Y | N/A |
| OPA (v1.14.0) | Policy author (Rego bundles) | Calling software (PEP) | Y by design | Recompute only | P — unsigned JSON decision logs | Bundle updates (no credentials) | Y |
| Cedar / Amazon Verified Permissions | Policy-store owner | Calling application | Y by design | Re-evaluate only | P — unsigned, operator-owned | Policy updates | Y |
| MS agent-governance-toolkit (OPA/Cedar) | Agent operator | PolicyEvaluator/PolicyEngine (fail-closed) | Y by design | N | P — operator logs | Policy updates | Y |
| MCP gateways as PEPs (Kong, Traefik, Docker, APIM) | Gateway operator | Gateway on HTTP path | Y | N | P — operator logs | Policy/token changes | Y (gateway = receiver policy) |

### Enterprise control and verifiable-control efforts

| System | Who defines authority? | Who enforces? | Survives model/provider switch? | Outside receiver can verify? | Decision records portable? | Revocable? | Receiver keeps own policy? |
|---|---|---|---|---|---|---|---|
| MS Agent Governance Toolkit | Enterprise operator policy | Runtime interception layer | Y (agent-level identity, per-action policy) | Logs inspectable, not wire-verifiable | P — hash-chained, operator-held | Y (kill switches, rings) | Y (receiver = operator) |
| Agent Control Standard (2026-05-27) | Operator policy via hooks | Platform implementing ACS | Designed Y | Local only | N (not standardized) | Operator-implemented | Y |
| CSAI AARM + ATF | Deployer policy | AARM enforcement point | Designed Y | Conformance only | N | Kill switches | Y |
| Delinea × StrongDM | Delinea Platform policy (Iris AI) | JIT runtime authorization | ? (not addressed) | N | N (platform audit) | Y (ephemeral creds) | Y |
| Okta/Auth0 + XAA | Enterprise admin + user consent (CIBA) | Okta/Auth0 identity layer | ? (directory-bound) | P (XAA open standard) [?] | N (no agent receipt format) | Y (kill switch, short-lived) | Y |
| Skyfire KYA/KYAPay | Human principal (paid subscription); seller sets assurance level | Seller (verifies JWT via JWKS) | ? (agent identity platform-scoped) | Y (local JWKS verification) | P — JWTs portable; track record hosted | Y (24h default expiry) | Y (seller sets required level) |
| Visa TAP | Visa directory (recognition); consumer (spending limits) | Merchant (signature verify) + network/issuer | N/A (recognition, not delegation) | Y (merchant-side verify) | ? (records Visa-internal) | Y (key directory, 8-min expiry) | Y |
| RSA Agent ID (ann) | Named owner + enterprise policy | RSA AI/MCP Gateway + operator approval | ? | N (operator path) | N (SIEM stream) | Y (decommission) | Y |
| Proof x401 (draft 0.2.0) | Verifier composes requirement; issuer asserts claims | Verifier at protected route | P (tokens narrowly scoped to verifier routes) | Y (core design point) | P — PROOF-RESULT carries decision info | Y (expiry, revocation, replay) | Y (verifier authoritative for issuer trust) |
| LFDT Proof-of-Control (draft v1.0) | Deployer defines control boundaries | N/A — evidence, not enforcement | N/A (evidence schema) | Y ("without trusting the operator") | Y — signed, hash-chained, anchored | N/A (control, not evidence) | Y (deployers disclose trust assumptions) |
| EMVCo Agentic Payments draft (2026-09-01) | Consumer (intent) | Each payment participant | ? | Via shared Intent Services | P — intent state fields (format TBD) | Intent lifecycle (details ?) | Y |
| Six-bank paper (2026-09-22) | Consumer (instructions/intent) | Each participant (own controls) | ? | Demanded, not shipped | Proposed, no format | Principle only | Y |

### Mandates, passports, notarized records, portable evidence

| System | Who defines authority? | Who enforces? | Survives model/provider switch? | Outside receiver can verify? | Decision records portable? | Revocable? | Receiver keeps own policy? |
|---|---|---|---|---|---|---|---|
| Google AP2 v0.2 (2026-04-28) | User via Trusted Surface | Verifiers per mandate (Merchant, CP, MPP) | Y under User Credential model; conditional under Trusted Agent Provider model | Y (offline signature/constraint verify) | Y — signed mandates + signed receipts (dispute evidence) | N — explicitly out of scope | Y |
| Mastercard Verifiable Intent (draft v0.1) | User/issuer (SD-JWT chain) | Consuming verifiers | ? | Y (public-key SD-JWT) | Y (tamper-proof log) | ? | ? |
| Sierra/Meta PAP (ann) | N/A — nothing public | N/A | N/A | N/A | N/A | N/A | N/A |
| ACP (beta, 2026-04-17) | Merchant/PSP (mints token) | Merchant's PSP | Y (token bound to session/allowance) | Merchant rails only | P — PSP audit trail | Expiry only | Y |
| DigiCert AI Trust Manager (GA 2026-09-15) | Owning org (passport + visas) | Operator policy engine + kill switch | ? | Y (no shared IdP needed) | ? | Y (kill switch) | Y (receiver can deny) |
| Cubitrek Agent Passport (draft v0.1) | Issuing business | Receiving operator | Y (key/DNS-bound) | Y (DNS + Ed25519, offline) | P — signed conversation log | ? | Y |
| ERC-8004 | Registering owner | Receivers (signals only) | Y (token ownership) | Y (on-chain) | Y (reputation/validation on-chain) | Transfer/owner-update | Y |
| AIP / IBCT (paper + draft) | Token issuer / delegating principals | Resource servers | Y | Y (public-key, offline) | P — provenance completion records | ? | Y |
| obsigna Agent Receipts | Issuing agent/tooling (agent side) | N/A — evidence only | Y (model-agnostic VCs) | Y (offline) | Y — hash-chained VCs | N/A (past actions) | N/A — receiver does NOT attest |
| agent-provenance (frozen 2026-10-04) | Harness operator | N/A — audit layer | Y | Y | Y — signed export bundles | Key-revocation events | N/A — witness federation unimplemented |
| UCAN (1.0-rc) | User DID → agent DID | Receiving service | Y (DID/key-bound) | Y (self-contained, offline) | N — authority only, no decision records | N — no in-protocol revocation | Y |
| Biscuit (v3) | Token issuer (Datalog facts) | Receiver/verifier | Y (public-key) | Y | N — authority only | P — revocation IDs, receiver blocklists | Y (receiver adds policies) |
| Macaroons (2014) | Issuing service | Service | Y | Symmetric-only | Y (bearer + caveats) | Third-party caveats (indirection) | Y |

---

## Distinctiveness verdict

**No existing system surveyed combines all six properties (a)–(f).** The evidence, system by system against the six criteria:

- **Google AP2 (User Credential model)** comes closest. It satisfies (a) the user defines the mandate on a Trusted Surface; (b) each verifier (Merchant, Credential Provider, MPP) independently verifies signatures, constraints, and key bindings and MUST return a signed receipt (accept or reject) before the consequential effect settles; (c) mandates are signed SD-JWTs naming amounts, payees, constraints, and agent keys — never a model or provider — and are verifiable by any party holding the issuer's keys; (e) signed mandates plus signed Checkout/Payment Receipts form a portable, non-repudiable record usable as dispute evidence; (f) each verifier evaluates constraints and applies its own policy. **It fails (d): the v0.2 spec explicitly places revocation and mandate management outside its scope** (implementation_considerations.md: the agent SHOULD limit mandate duration; management of delegated mandates by external Trusted Surfaces "is outside the scope of this specification"). Agent-to-agent delegation is also explicitly out of scope in v0.2. Scope is payments.
  Primary: https://github.com/google-agentic-commerce/AP2 (docs/ap2/ read 2026-10-06)

- **UCAN / Biscuit** satisfy (a), (b), (c), (f): user/issuer-defined attenuated delegation, offline-verifiable by any receiver, provider-agnostic, receiver adds its own policy at authorization time. **They fail (d) and (e): no in-protocol revocation** (UCAN: short expiry + issuer-side lists with known freshness pitfalls; Biscuit: revocation IDs for receiver-maintained blocklists, no standard) **and no decision records** — they are portable *authority*, not portable *evidence of decisions*.

- **Proof x401** satisfies (b), (f), and partially (d): an independent Verifier composes requirements, cryptographically validates presentations, returns structured PROOF-RESULT feedback, and may issue scoped, expiring, revocable Verification Tokens. **It fails (a) as stated**: the authority relationship is verifier-composed (admission) — the Verifier authors the proof requirement and "remains authoritative for issuer trust enforcement" — not an owner-defined mandate the verifier merely accepts or refuses. Tokens are narrowly scoped to the verifier's own routes, so (c) holds only within that verifier's domain.

- **LFDT Proof-of-Control** satisfies (e) better than anything else surveyed (signed, hash-chained, anchored evidence verifiable "without trusting the operator") but is explicitly **not** an authority or enforcement system — (a), (b), (c), (d) are out of its scope by design.

- **obsigna / agent-provenance** satisfy (e) for *action* records but the signer is the agent side — the receiver does not attest — so they are not (b) receiver decisions, and they define no authority model at all.

- **MCP / A2A / OAuth token-exchange drafts / workload identity / OPA / Cedar / enterprise PAM and gateway products** define no owner-originated portable mandate at all: authority is the operator's policy, the IdP's issuance, or the verifier's admission criteria. They satisfy (b) and (f) — the receiver decides under its own policy — but that is precisely the *admission* side of the question, not portable owner authority.

- **Sierra/Meta Personal Agent Protocol**: announced 2026-10-06; v0.1 spec planned for later in October 2026; payments and granular permissions are explicitly future extensions. Nothing can be evaluated yet.

**Plain-English summary:** the landscape splits into two halves that never meet in one system. One half (AP2, UCAN, Biscuit, AIP) gives the *owner* a portable, signed, attenuable statement of authority — but revocation is expiry-based and nobody produces a portable record of the receiver's decision. The other half (x401, OPA/Cedar/AVP, MCP gateways, enterprise PAM, RSA Agent ID) gives the *receiver* a real decision point under its own policy — but the authority it enforces is its own policy or the verifier's admission criteria, not an independently originated owner mandate. AP2 is the single system that puts owner-signed mandates and receiver-signed decision receipts in the same protocol — and its spec explicitly defers revocation. If a future AP2 revision (or Sierra PAP, when it ships) adds a revocation mechanism with continuity across provider replacement to the existing mandate+receipt structure, the combination would be substantially replicated and this paper's distinctiveness claim would need to narrow to non-payment domains or to revocation/history semantics specifically.

---

## [VERIFY] — could not confirm from primary sources

1. Sierra/Meta PAP: everything beyond the 2026-10-06 announcement; v0.1 not shipped.
2. AP2: revocation primitive — confirmed absent from v0.2 spec text read; confirm no companion revocation doc exists; no production deployment evidence beyond samples; PyPI package unreleased.
3. Mastercard Verifiable Intent: primary spec (verifiableintent.dev) not directly read.
4. Visa TAP: live merchant-side deployment status unverified (spec published).
5. Visa Intelligent Commerce / Mastercard Agent Pay implementation status — announced 2025-04; no public spec or deployment evidence found.
6. DigiCert AI Trust Manager: wire format of AI Passport/visa tokens; whether cross-org verification demonstrated vs documented.
7. Cubitrek Agent Passport: revocation story.
8. ZeroID (Highflame): only third-party deep-dive; no primary docs found.
9. UCAN: 1.0-rc vs settled standard; any standardized revocation profile; ucan.xyz not directly read.
10. Biscuit: biscuitsec.org not directly read; receiver-side blocklist standard not confirmed.
11. Macaroons: no 2026 agent-specific deployment primary source.
12. x402 v2: primary spec not read (secondary comparisons only).
13. ACP Shared Payment Token: protocol details from secondary sources only.
14. obsigna: no gaps — spec read directly.
15. SPIFFE/SPIRE: per-credential revocation mechanism beyond expiry/rotation/entry deletion not found in spec pages read.
16. OPA v1.14.0 version: from secondary source; changelog not checked.
17. Strands harness-sdk Cedar plugin: design doc exists; shipped implementation not verified.
18. Traefik Hub MCP Gateway v3.20: GA planned late 2026-04; actual GA not verified.
19. Kagenti SPIRE integration: community ADR only; Red Hat primary status not verified.
20. MCP 2026-07-28 spec: finalization status vs stable line 2025-11-25.
21. MCP EMA: normative revocation section not read.
22. draft-araut transaction-tokens-for-agents: revocation semantics beyond short-lived tokens.
23. Okta for AI Agents GA (2026-04-30), Agent SSO GA (2026-08-24), XAA as MCP EMA extension: secondary reports; Okta primary docs not fetched.
24. Microsoft Entra Agent ID GA details: secondary only.
25. CyberArk→Idira rebrand (2026-05): single secondary source.
26. Six-bank paper: clause-level content (paper PDF not retrieved; findings rest on PYMNTS/Reuters summaries).
27. Cisco MCP proxy, Google Cloud Agent Identity GA: secondary reports only.
28. Radware Agent Trust Management: secondary coverage only.
29. Anon (agent identity vendor): no primary 2026 source found; existence unconfirmed.
30. AIP/IBCT: explicit revocation mechanism; deployment beyond reference implementations.
31. AAuth working demo (Keycloak/Agentgateway/A2A/MCP): secondary only; revocation and decision-record portability unspecified in excerpts read.
32. Skyfire KYAPay "early production deployments": draft's own claim, not independently verified.
33. draft-song-oauth-ai-agent-collaborate-authz, draft-embesozzi-oauth-agent-native-authorization: seen in secondary landscape lists only; not verified on datatracker.
34. AuthZEN standards status: not verified from primary source.
35. draft-prakash-aip (Biscuit token chain) vs NVIDIA draft-aip-agent-identity-protocol: known from secondary landscape summary only.
36. Astrix "8.5% of OSS MCP servers use OAuth" (2025-10): cited via OWASP AISVS wiki; underlying report not fetched.
37. NVIDIA Open Agent Safety Platform / OpenShell: announced ~2026-09-30; secondary only; shipped status unverified.
38. No dedicated 2026 notarization service for agent actions found — agent-specific evidence uses hash chains + optional external anchors; generic RFC 3161 TSAs exist but are not agent-targeted.
