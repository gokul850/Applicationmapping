# AI Agent Harness and AI-SDLC Implementation Plan

**Status:** Proposed build plan; not an authorization to connect to or modify operational systems
**Source of truth:** [`skill.md`](./skill.md)
**Operating model:** [`approach.md`](./approach.md)
**Audience:** Agent/platform engineers, data and CMDB owners, security, QA/evaluation, and human approvers

## 1. Purpose and scope

Implement a governed AI-agent harness that can assist with the skill’s CMDB-first ADM work: source cataloguing, data-quality analysis, application/CI reconciliation, candidate dependency discovery, evidence packaging, and target-schema validation. The harness should generate reviewable proposals and traceable artefacts. It must not autonomously alter Azure, on-premises, Device42, or ITSM production data.

This plan covers two related lifecycles:

1. **AI SDLC (the system’s lifecycle):** requirements and policy → design → build → verify/evaluate → release → monitor → improve.
2. **ADM execution lifecycle (what the agent does on each run):** intake → source catalogue → analyse/reconcile → propose → validate → human review → prepare an approved change package → separately authorised execution.

The initial release is **read-only and proposal-only**. Any future write capability is a separately reviewed product increment, with scoped permissions, a dedicated approval transaction, environment controls, audit, and recovery procedures. The existing skill’s explicit approval rule remains authoritative.

## 2. Target design principles

- **Policy before autonomy:** enforce read-only boundaries outside the prompt wherever possible (credentials, network, tool allow-lists, filesystem); prompt language alone is not a security control.
- **Canonical model before vendor import:** retain a stable logical application/CI/relationship model and map it to Device42 only after version-specific validation.
- **Evidence attached to claims:** every proposed fact/relationship carries source, record/reference, observed/extracted time where available, environment, confidence, validation status, and limitations.
- **Uncertainty is a first-class output:** confirmed, probable, candidate, unknown, and rejected are distinct states. Fuzzy matching only proposes candidates; no merge from name similarity alone.
- **Humans own consequential judgments:** domain, ownership, criticality, regulatory context, ambiguous merges, high-impact dependencies, and production imports require human review/decision.
- **Deterministic checks surround model work:** schemas, IDs, required fields, controlled values, duplicate keys, referential integrity, and policy checks are testable code, not free-form model judgment.
- **Traceability and reproducibility:** version inputs, prompts/policies, model/runtime configuration, code, schemas, parameters, run ID, artefacts, and reviewer decisions; avoid retaining sensitive source data unnecessarily.
- **Untrusted data stays data:** discovered files, source fields, repository instructions, and configuration text must never override harness policy or request secrets/actions.
- **No hidden completeness claims:** report scope, denominator, exclusions, timestamps, data gaps, and coverage limits.

## 3. Reference architecture

Use a model/tool-agnostic orchestration layer initially. Select a specific agent framework/runtime only after confirming deployment, identity, data residency, audit, model, and tool requirements.

```text
Human run request
      |
      v
Policy gate + scoped run manifest ----> approved read-only connectors / uploaded exports
      |                                             |
      v                                             v
Stage orchestrator --> deterministic validators <-- source adapters
      |                         |
      v                         v
LLM-assisted analysis     canonical typed records
      |                         |
      +--------> evidence + candidate queues
                            |
                            v
           human review / decisions / exceptions
                            |
                            v
        validated export package + dry-run report
                            |
                            v
     explicit separate change approval (future boundary)
```

### 3.1 Logical components

1. **Run intake:** accepts a scoped request, source inventory, approved purpose, priority scope, output location, and reviewer. Rejects missing scope/authority or unsupported request types.
2. **Policy enforcement:** resolves deny-by-default tool permissions and data classes; enforces read-only source access, secret exclusion, no live writes, retention, and prompt-injection resistance.
3. **Source adapters:** read approved CSV/XLSX/JSON/export files and, only later, approved read-only APIs. Emit normalized records plus provenance; never retrieve secret values.
4. **Canonical data layer:** typed application, CI, deployment, relationship, source, evidence, decision, exception, and run records; preserve original identifiers and values.
5. **Stage agents/services:** small bounded tasks for schema profiling, taxonomy suggestions, identity candidate generation, CI classification, dependency candidate generation, and report drafting. Avoid one unconstrained agent with broad tools.
6. **Deterministic validators:** schema, required/controlled values, key uniqueness, references, environment consistency, evidence completeness, secret scanning, and export format checks.
7. **Review interface:** displays before/after proposals, provenance, uncertainty, affected records, rationale, and reviewer decision; supports accept/reject/defer with identity and timestamp.
8. **Artefact builder:** creates the skill-required outputs and a run manifest; distinguishes proposals from approved facts and packages from executed imports.
9. **Observability/evaluation:** records run/version IDs, model/tool calls, validation results, reviewer overrides, errors, latency/cost if applicable, and policy events while minimizing sensitive logging.

## 4. Proposed repository harness files

Keep the business/domain authority in `adm/skill.md`. The harness files below are proposed additions; adapt to the selected agent runtime rather than binding the domain model to one vendor’s prompt convention.

```text
AGENTS.md                                  # repository-wide contributor/agent instructions
adm/
  skill.md                                 # canonical domain skill (existing)
  approach.md                              # human delivery approach
  implementation-plan.md                  # this plan
  harness/
    README.md                              # runtime-neutral entrypoint and local workflow
    policies/
      safety-policy.md                     # invariant security/change boundaries
      data-handling.md                     # sensitivity, redaction, retention and secrets
      approval-gates.md                    # what requires human decision and evidence
    contracts/
      run-manifest.schema.json             # scope, sources, actor, mode, versions, outputs
      source-catalogue.schema.json         # source metadata and freshness
      application.schema.json              # canonical application contract
      ci.schema.json                       # technical CI/deployment contract
      relationship.schema.json             # relationship plus evidence/confidence
      decision.schema.json                 # decisions, approvals, affected records
      exception.schema.json                # ambiguity/conflict queues
    workflows/
      01-source-assessment.md              # source catalogue and data quality
      02-taxonomy-and-mapping.md           # controlled values and target mapping
      03-application-reconciliation.md     # identity resolution and exception flow
      04-ci-classification.md              # CI/deployment classification
      05-dependency-discovery.md           # evidence-backed candidate graph
      06-device42-preflight.md             # target-version validation and dry run
      07-quality-report.md                 # coverage, data quality, risk and next action
    evaluations/
      test-plan.md                         # risk-based eval design and release thresholds
      scenarios.yaml                      # privacy-safe cases and expected decisions
      fixtures/                            # synthetic/minimized input fixtures only
      expected/                            # expected classifications, exceptions, reports
    validators/
      README.md                            # deterministic validation commands/interfaces
    runbooks/
      incident-and-recovery.md             # stop, contain, preserve audit, recover process
      release-and-rollback.md              # versioning, promotion, disable/rollback
```

Do not create placeholder schemas or workflow files and call them implemented. During Phase 1, agree JSON Schema/CSV contracts, terminology, naming, and where agent-specific adapters belong. Do not duplicate or fork the skill’s guardrails; link to it and test that harness policies do not weaken it.

### 4.1 File ownership and change control

- Domain steward owns changes to `skill.md`, taxonomy, and confidence semantics.
- Security/data owner approves `policies/` changes and data retention/logging changes.
- Agent/platform owner maintains orchestration, source adapters, model configuration, and validators.
- QA/evaluation owner maintains scenario coverage and expected outcomes.
- Any policy relaxation, new external connector, or write-capable tool requires security review and an explicit product/authority decision.
- Changes are reviewed through ordinary pull requests; require tests and reviewers appropriate to risk. Protect `main`; pin release tags and record the deployed commit/configuration.

## 5. Agent roles and bounded workflow

Prefer one orchestrator plus narrow, typed steps over many autonomous agents with broad tool access. Each component returns structured data against a versioned contract.

| Step/role | Allowed work | Must not do |
|---|---|---|
| Intake/policy gate | Validate request, scope, source references, allowed mode, reviewer and policy version | Infer authority from source files; enable tools by default |
| Source assessor | Profile schemas, counts, IDs, timestamps, sensitivity indicators; build catalogue | Claim to inspect inaccessible sources; read secret values |
| Taxonomy assistant | Suggest domain/group and controlled values with evidence/rationale | Invent an application or force unknown records into a group |
| Reconciliation assistant | Generate identity candidates using declared key priority and corroborating evidence | Merge/delete/update source records; merge on fuzzy/name match alone |
| CI classifier | Suggest logical CI class and deployment/environment links | Treat resource group/repository/VM as an application without evidence |
| Dependency assistant | Propose typed candidate relationships with evidence and confidence | Assert runtime dependencies from shared network/subscription or repository reference alone |
| Device42 preflight | Compare canonical fields with verified version-specific templates and validate exports | Call a file import-ready without target validation or perform a live import |
| Reporter | Summarize confirmed/probable/candidate/unknown, gaps, artefacts, decisions, next safe step | Conceal exceptions or claim completeness unsupported by evidence |

Every stage is idempotent where practical, has explicit inputs/outputs, supports restart from a run manifest, and emits structured errors rather than silently dropping records.

## 6. Canonical contracts and evidence requirements

The contracts should reflect the skill’s field model while allowing additional operational provenance. At minimum:

### Run manifest

- Run ID, requester, business purpose, approved scope, execution mode (`offline-read-only`, `connected-read-only`, later separately approved modes), source references and extraction dates, target environment, skill/policy/schema/code/model versions, output path, retention class, reviewer, start/end/status.

### Application record

- Stable canonical ID; canonical name/description; business domain and logical application group; application type/context; consumer/product/customer/tenant/programme/jurisdiction/payment metadata when applicable; environment/lifecycle/criticality; hosting/vendor; ownership/support fields; source systems/record IDs; evidence; confidence; validation status; last observed; notes.

### CI/deployment record

- Stable canonical/external key; native Azure resource ID or other source ID; CI type; name/aliases; subscription/resource group/region or on-prem host/platform; environment; deployment/version/customer/tenant if material; application mapping and evidence; source and last observed.

### Relationship record

- Source and target stable IDs; relationship semantics; environment; source/reference/timestamp; confidence; validation status; last observed; notes; whether business support, runtime app-to-app, app-to-CI, infrastructure, or ownership/support relationship.

### Evidence and decision

- Evidence pointer and source identity, extraction/observation time, relevant excerpt/field reference (redacted/minimized), method, confidence, contradictions, reviewer and validation result. Decisions capture alternatives, recommendation, approver, date, affected IDs, and revisit date.

No schema should put secrets, credentials, tokens, private keys, or raw sensitive payloads into LLM prompts, logs, or generated artefacts. Source policies must specify whether personal, payment, customer, or regulated data is excluded, tokenized, or handled in an approved isolated environment.

## 7. AI-SDLC work plan and gates

The timeboxes below are an implementation proposal, not a committed schedule. Align resourcing and calendar dates during mobilisation. The agent build runs in parallel with the six-month human CMDB/ADM programme but does not replace it.

### Phase 0 — Product framing and constraints (Weeks 1–2)

**Activities**
- Identify sponsor/product owner, CMDB domain steward, security/privacy, data owner, platform/agent engineer, QA/evaluation lead, and human approvers.
- Select initial use case: offline analysis of approved exports for source catalogue and reconciliation proposals.
- Define users, risk classification, in-scope domains/data, supported formats, deployment boundary, model/runtime choices to evaluate, expected output and retention.
- Document explicit prohibited actions and authority boundary. Default to no production connectors and no write tools.
- Establish acceptance criteria, decision rights, issue/incident path, and repo branch/review/release controls.

**Deliverables:** product brief, system context/data-flow diagram, initial threat model, risk register, and approved read-only MVP scope.

**Exit gate:** security/data owner confirms handling boundary; sponsor approves the MVP objective; no unresolved decision about access or intended use is hidden as an implementation assumption.

### Phase 1 — Harness contracts and policy foundation (Weeks 2–4)

**Activities**
- Create `AGENTS.md` contributor instructions and `adm/harness/README.md` runtime-neutral architecture.
- Define run manifest, source catalogue, canonical application/CI/relationship, evidence, decision, and exception schemas.
- Define allowed modes/tools, source sensitivity labels, redaction, retention, logging, prompt-injection handling, refusal/stop conditions, and approval record format.
- Convert the skill’s rules into testable invariants: strong-key order, no name-only merge, no unsupported dependency inference, explicit Unknown/Unassigned, secret exclusion, separate domains, one connected graph, no live writes.
- Write workflow contracts and report format consistent with the skill’s required response.

**Deliverables:** versioned contracts/policies, workflow specifications, traceability matrix, policy unit tests.

**Exit gate:** domain steward and security approve contracts/policies; tests demonstrate deny-by-default operation and reject forbidden modes.

### Phase 2 — Read-only prototype and deterministic validators (Weeks 4–7)

**Activities**
- Build local/offline parsing for a bounded, privacy-safe set of approved input formats.
- Implement source profiling, normalization that preserves original IDs/values, canonical records, and evidence pointers.
- Implement deterministic checks for schema, key uniqueness, required/controlled values, references, environment consistency, valid CSV escaping, secret-like content checks, and audit manifest completeness.
- Add model-assisted candidate generation only behind structured output schemas; keep model output as suggestions, never direct record writes.
- Generate source catalogue, data-quality report, candidate duplicate/unmapped/owner-gap queues, and relationship candidates without operating-system/production connections.

**Deliverables:** reproducible prototype, validators, synthetic or minimized fixtures, sample reports, known-limitations list.

**Exit gate:** same inputs/version produce reproducible canonical/validator outputs within defined deterministic scope; no prohibited network/write capability; failure paths preserve run status and evidence.

### Phase 3 — Evaluation and red-team hardening (Weeks 6–9)

**Activities**
- Create an evaluation set covering the four domains, edge cases, ambiguous identities, conflicting sources, MTi tenant/deployment, government subsidy jurisdiction/payment model, shared service false positives, environment mismatches, stale evidence, and missing owners.
- Add adversarial input fixtures containing instruction-like text, secret-like values, malformed files, prompt injection, misleading repository references, false “orphan” cases, and cross-portfolio ambiguity.
- Evaluate precision/recall for candidate matching and classification only where labels/ground truth exist; otherwise use expert-rated acceptance, severe-error counts, evidence completeness, calibration and reviewer override analysis.
- Compare deterministic rules, model-assisted suggestions, and human baseline; test model/config upgrades against the regression suite.
- Test access controls, redaction, data minimization, audit completeness, failure, timeout, restart, and recovery.

**Deliverables:** eval report, threat-model updates, regression suite, documented release thresholds and failure criteria.

**Exit gate:** zero critical policy violations in the release suite; performance thresholds and unacceptable-error classes agreed by domain/security owners; all unresolved limitations explicitly accepted or deferred.

### Phase 4 — Controlled pilot on approved data (Weeks 9–12)

**Activities**
- Run on one bounded, approved priority scope in an isolated/non-production context with read-only exports or read-only access.
- Human reviewers inspect record-level proposals, evidence quality, duplicate/exception queues, domain/group assignments, and dependency candidates.
- Capture accept/reject/defer decisions and reasons; tune taxonomy/prompt/rules only through reviewable changes and regression tests.
- Validate Device42 version/templates separately; build only a dry-run/preflight package with no production write credentials.
- Rehearse incident handling, rollback of the agent deployment, and containment if sensitive data is detected.

**Deliverables:** pilot report, reviewed proposal set, revised risks, approved runbook, go/no-go recommendation for wider read-only use.

**Exit gate:** human owners accept the pilot’s utility/limitations; security approves operational controls; no unresolved severe defect; documented production-write exclusion remains enforced.

### Phase 5 — Release and operationalization (Weeks 12–14)

**Activities**
- Release pinned code/policy/schema/model configuration with software bill of materials/dependency review as appropriate, signed/versioned artefact, deployment and rollback procedure.
- Define access provisioning, operator training, monitoring, support ownership, run scheduling, change approval, data retention/deletion, incident response, and model/version change process.
- Publish a release card with version, scope, evaluated models/config, known limitations, inputs allowed, controls, data handling, and non-capabilities.

**Deliverables:** versioned release, operator runbook, release notes, monitoring dashboard specification, approved support model.

**Exit gate:** change authority approves release into the defined read-only operating boundary; rollback and incident procedures tested.

### Phase 6 — Continuous improvement (ongoing)

- Review data and model quality, reviewer overrides, exceptions, freshness, and user feedback on a defined cadence.
- Add scenarios before broadening scope, adding sources, changing models, changing schemas, or enabling new tools.
- Reassess threat model, data handling, access, and retention after material change.
- Version any change to `skill.md`; migrate schema and prompt fixtures deliberately; maintain backward compatibility or documented migrations.
- Treat write-capability as a separate gated programme increment, not a configuration toggle.

## 8. AI workflow lifecycle and human checkpoints

| Workflow stage | Agent output | Required human checkpoint |
|---|---|---|
| Intake | Validated run manifest or rejection with missing items | Confirm scope/data-use authorization where policy requires |
| Source assessment | Catalogue, schemas, counts, freshness and data-quality notes | Source owner confirms intended source/scope and limitations |
| Taxonomy/mapping | Proposed domain/group and target-field mappings | Business/domain steward confirms ambiguous classifications and controlled values |
| Reconciliation | Canonical candidates, duplicate/conflict queues, field provenance | Owners decide merge, conflict resolution, identity and ownership assertions |
| CI/deployment mapping | Candidate application-to-CI links and classifications | Technical/platform owner validates material mappings |
| Dependency discovery | Candidate graph edges with evidence/confidence | Technical and business owners validate critical/high-impact edges |
| Device42 preflight | Version-specific mapping, validation/dry-run, exceptions | Platform owner confirms schema; change authority separately approves any import |
| ITSM readiness | Integration/process evidence, UAT, gaps | ITSM owner and sponsor accept readiness; separate change approval for go-live |
| Reporting | Scope, findings, artefacts, quality, risks, decisions, next safe step | Delivery lead validates reporting; named approvers decide open items |

The agent may continue safe, reversible analysis where a low-risk question is unresolved, but records the assumption. It must stop when missing authority, access, or a material business choice changes the permitted action or outcome.

## 9. Security, safety, and release controls

### 9.1 Minimum controls for MVP

- Use approved exports first; isolate runs and restrict filesystem/source access to the declared run scope.
- No production write credentials, import endpoints, delete operations, or broad shell/browser/network tools in the MVP runtime.
- Least-privilege, time-bounded read access only if approved connectors are later added.
- Secret scanning/redaction before model submission and before artefact publication; never read secret values from vaults.
- Data minimization: send only fields required for a specific task; avoid raw full datasets in prompts when deterministic local processing suffices.
- Separate source data from instructions; flag and neutralize instruction-like source content; maintain a system policy that is not constructed from untrusted input.
- Per-run audit record: actor, scope, input fingerprints/references, extraction times, code/policy/schema/model versions, tool actions, outputs, validators, reviewer decisions, and errors.
- Human review for criticality, ownership, regulatory classification, ambiguous merges, high-impact relationships, and all production imports/changes.
- Retention/deletion and access controls defined before processing sensitive data; logs must not become a shadow copy of source data.

### 9.2 Future write-capability prerequisites (not MVP)

Do not add until separately authorized and reviewed. Required design includes distinct service identity, narrow API allow-list, environment binding, immutable proposal package, idempotency and dry-run, explicit named approval bound to an exact package hash/scope, two-person review where policy requires, pre/post state capture, rate/volume limits, transaction logging, partial-failure handling, rollback/recovery, monitoring, emergency stop, and tested non-production promotion. The agent cannot approve its own change or convert a review acknowledgment into execution authority.

### 9.3 Stop/contain conditions

Stop processing and preserve an incident/run record if secrets or prohibited sensitive values are detected, source scope is exceeded, an unexpected write-capable tool is available, policy instructions conflict, target schema cannot be verified, output references unknown IDs as confirmed, or a request asks to bypass approval. Do not copy the exposed value into reports/logs; notify the designated security/data owner through the approved channel.

## 10. Verification and acceptance criteria

### 10.1 Functional acceptance

- Source catalogue records only sources actually accessed and includes extraction date/freshness, schema, counts where computable, limitations, sensitivity, and intended use.
- Canonical outputs preserve source IDs/original names and represent unknowns explicitly.
- Reconciliation follows authoritative IDs → native IDs → CMDB IDs → stable identifiers → corroborated normalized name; fuzzy matching only creates candidates.
- No automatic merges, deletes, production updates, or live imports are possible in MVP.
- Relationship proposals include source, target, type, evidence, environment, timestamp where available, confidence, and validation state.
- Shared subscription/network/resource group/repository references alone never elevate a dependency to confirmed.
- Taxonomy distinguishes Taxi Networks, Taxi Payments, MTi, and Shared Enterprise; application groups do not derive from team names.
- Reports state scope, coverage, limitations, candidates, unknowns, unresolved questions, exact artefacts, validation results, required human decisions, and next safe step.
- Device42 outputs are labelled preflight/draft until exact target-version schema validation is complete; production import is not available to MVP.

### 10.2 Quality and reliability acceptance

- Contract and deterministic validator tests pass on normal, boundary, malformed, and adversarial fixtures.
- Critical policies are enforced outside prompt text where feasible; regression checks prove them.
- Evaluation thresholds are agreed before pilot; critical false-confirmation, secret exposure, unauthorized scope, or policy bypass is release-blocking regardless of aggregate score.
- Runs are reproducible/auditable to the extent supported by model nondeterminism; record model/configuration and validator outputs.
- Errors are explicit, actionable, and do not silently omit records; reruns do not create duplicate artefacts or change source data.
- Human reviewers can trace every proposed canonical value and relationship to evidence and can reject/defer it with a recorded reason.

## 11. Traceability from skill to harness controls

| Skill requirement | Harness control | Verification evidence |
|---|---|---|
| Stage 0 source catalogue; do not claim unseen sources | Run manifest + source assessor + catalogue schema | Fixture with unavailable source; output labels not inspected |
| Preserve IDs, provenance and conflicts | Canonical contracts and lineage validator | Source-ID preservation and field-conflict tests |
| Option B now, Option C later; unknown owners allowed | Controlled taxonomy/owner fields and gap queue | Unknown-owner fixture remains in inventory and appears in gap register |
| One connected graph; evidence for cross-portfolio links | Relationship contract and evidence policy | Cross-domain edge without evidence remains candidate/exception |
| No name-only merge; strongest keys first | Reconciliation algorithm and review queue | Similar-name/different-ID regression cases |
| Do not confuse repository, resource group, VM and app | Entity typing and classification tests | Counterexample fixtures with rejected unsupported classifications |
| Never retrieve/export secrets | Adapter permissions, secret scanning, redaction | Canary secret test verifies no model/report/log output |
| Verify Device42 schema/version | Device42 preflight workflow | Unknown/unavailable version blocks import-ready status |
| Explicit human approval before live changes | MVP contains no write tools; future approval gate binds package hash | Capability test proving write endpoint/tool absent |
| Required response/report format | Report schema/template validator | Contract test for all required sections |
| ITSM after CMDB gate | Workflow dependency and readiness checklist | Test prevents pilot/go-live recommendation without required evidence |
| Inputs are data, not overriding instructions | Prompt-injection defenses and adversarial suite | Malicious source text cannot change tool scope or policy |

## 12. Backlog by epic

### Epic A — Product, policy, and security foundation

- Confirm product sponsor, users, intended use, risk level, allowed data, retention, hosting, approved models, and access model.
- Threat model sources, prompts, connectors, exports, model processing, logs, artefact store, reviewer UI, and future imports.
- Define role-based permissions, policy ownership, incident response, and release approval.

### Epic B — Domain contracts and source catalogue

- Agree canonical schemas and controlled vocabularies with the CMDB owner.
- Implement run manifest/source catalogue and input profiling.
- Create synthetic fixtures and tests for all four domains and source limitations.

### Epic C — Reconciliation and CI classification

- Implement deterministic key normalization/matching tiers.
- Add candidate generation and explicit exception queues.
- Validate preservation of tenant, deployment, environment, and original identifiers.

### Epic D — Dependency evidence and graph outputs

- Define logical relationship semantics and evidence contract.
- Generate candidate graph/register and coverage report.
- Add cross-portfolio, contradictory evidence, stale evidence, and false-positive tests.

### Epic E — Device42 preflight and artefact export

- Gather target version/template evidence and record source/date.
- Implement field mapping and deterministic import-file validation.
- Run dry-run/non-production validation; produce recovery notes; do not implement production import in MVP.

### Epic F — Evaluation, pilot, and operations

- Establish golden scenarios, expert labeling/review, adversarial suite, and release thresholds.
- Pilot bounded approved data; record reviewer outcomes and defects.
- Publish release, runbook, observability, rollback, and support handover.

## 13. Risks and mitigations

| Risk | Mitigation |
|---|---|
| Prompt-only controls are bypassed | Enforce access/tool restrictions in runtime and test actual capabilities. |
| Model confidently invents mappings or edges | Typed output, evidence requirement, confidence state, validators, exception queue, human review; measure false confirmations. |
| Sensitive data enters prompts or telemetry | Minimize fields; preflight scanning/redaction; approved environment; retention and log review; canary tests. |
| Model/runtime changes shift outcomes | Pin versions/config; maintain regression evals; compare against approved baselines; require change review. |
| Harness becomes coupled to unverified Device42 assumptions | Keep canonical model independent; validate exact installed version and schema in preflight. |
| Agent duplicates source truth or overwrites records | Proposal-only outputs and source provenance; canonical repository is a review artefact, not authority until accepted. |
| Human reviewers face unmanageable candidate volumes | Prioritize by risk/evidence/impact; batch low-risk review; track review latency and override rates; improve deterministic filters. |
| “AI SDLC” is confused with authority to run IT production changes | Separate build lifecycle and ADM workflow; no write tools in MVP; clear approval boundary. |
| Unsupported thresholds create misleading launch decisions | Set thresholds with stakeholders and report scope/denominator/date; use qualitative gate evidence where no threshold is agreed. |

## 14. Decisions required before implementation

1. Which agent runtime/framework and deployment boundary are approved? What data may be sent to which model, under what terms and location?
2. What are the first approved source formats and priority application scope? Who authorizes each source?
3. What is the authoritative source for each canonical field, and who is the domain/data steward?
4. Which Device42 version, import templates, relationship semantics, environments, and test instance are available for validation?
5. Which ITSM platform/version is in scope, and who owns integration and process decisions?
6. What retention, access, audit, and incident-notification requirements apply to source extracts and generated artefacts?
7. What review thresholds and acceptance criteria constitute sufficient utility for a read-only pilot?
8. Where will human decisions and exceptions be recorded, and who can approve a future import/change package?

Do not select answers implicitly in code. Record decisions in the project decision log.

## 15. Recommended first increment

Build a **local/offline, read-only source-assessment prototype** against a small approved and sanitized sample. It should produce a source catalogue, data-quality report, and a run manifest; include deterministic validators and a synthetic prompt-injection/secret test; and have no production connectors or write tools. Review the prototype with CMDB, business, technical, and security owners before expanding to reconciliation or connected sources.

**Definition of done for the first increment:** the sample’s sources and limitations are accurately catalogued; outputs preserve IDs and distinguish known/unknown; validators pass; sensitive canaries are not emitted; the run is reproducible/auditable; reviewers can trace findings to evidence; and the read-only/no-write boundary is technically demonstrated.
