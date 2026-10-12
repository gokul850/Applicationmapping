# AI Agent Harness and AI-SDLC Implementation Plan

**Status:** Proposed build plan; not an authorization to connect to or modify operational systems
**Source of truth:** [`skill.md`](./skill.md)
**Operating model:** [`approach.md`](./approach.md)
**Audience:** Agent/platform engineers, data and CMDB owners, security, QA/evaluation, and human approvers
**Planning horizon:** Six months. Phase 1 (App-Resource) Month 1-2; Phase 2 (Business Service) Month 3; Q2 ITSM rollout. Confirm calendar dates, scope, capacity, and thresholds during mobilisation.

## 1. Purpose and scope

Implement a governed AI-agent harness that can assist with Phase 1 (Application-Resource Matrix) and Phase 2 (Business Service Mapping) discovery. The harness should generate reviewable proposals and traceable artefacts. It must not autonomously alter Azure, on-premises, Device42, or ITSM production data.

This plan covers two related lifecycles:

1. **AI SDLC (the system’s lifecycle):** requirements and policy → design → build → verify/evaluate → release → monitor → improve.
2. **ADM execution lifecycle (what the agent does on each run):** intake → source catalogue → analyse/reconcile → propose → validate → human review → prepare an approved change package → separately authorised execution.

The initial release is **read-only and proposal-only**. Any future write capability is a separately reviewed product increment, with scoped permissions, a dedicated approval transaction, environment controls, audit, and recovery procedures. The existing skill’s explicit approval rule remains authoritative.

## 2. Target design principles

- **Policy before autonomy:** enforce read-only boundaries outside the prompt wherever possible (credentials, network, tool allow-lists, filesystem); prompt language alone is not a security control.
- **Canonical model before vendor import:** retain a stable logical application/CI/relationship model and map it to Device42 only after version-specific validation.
- **Evidence attached to claims:** every proposed fact/relationship carries source, record/reference, observed/extracted time where available, environment, confidence, validation status, and limitations.
- **Uncertainty is a first-class output:** confirmed, probable, candidate, unknown, and rejected are distinct states. Fuzzy matching only proposes candidates; no merge from name similarity alone.
- **Humans own consequential judgments:** product classification, ownership, criticality, regulatory context, ambiguous merges, high-impact dependencies, and production imports require human review/decision.
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
      05-service-mapping.md                # Phase 2: business service to application
      06-dependency-discovery.md           # evidence-backed candidate graph
      07-device42-preflight.md             # target-version validation and dry run
      08-quality-report.md                 # coverage, data quality, risk and next action
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

- Product steward owns changes to `skill.md`, taxonomy, and confidence semantics.
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
| Taxonomy assistant | Suggest product group/name and controlled values with evidence/rationale | Invent an application or force unknown records into a group |
| Reconciliation assistant | Generate identity candidates using declared key priority and corroborating evidence | Merge/delete/update source records; merge on fuzzy/name match alone |
| CI classifier | Suggest logical CI class and deployment/environment links | Treat resource group/repository/VM as an application without evidence |
| Service mapper | Suggest business service links and support relationships | Invent a business service or force one-to-one mapping without evidence |
| Dependency assistant | Propose typed candidate relationships with evidence and confidence | Assert runtime dependencies from shared network/subscription or repository reference alone |
| Device42 preflight | Compare canonical fields with verified version-specific templates and validate exports | Call a file import-ready without target validation or perform a live import |
| Reporter | Summarize confirmed/probable/candidate/unknown, gaps, artefacts, decisions, next safe step | Conceal exceptions or claim completeness unsupported by evidence |

Every stage is idempotent where practical, has explicit inputs/outputs, supports restart from a run manifest, and emits structured errors rather than silently dropping records.

## 6. Canonical contracts and evidence requirements

The contracts should reflect the skill’s field model while allowing additional operational provenance. At minimum:

### Run manifest

- Run ID, requester, business purpose, approved scope, execution mode (`offline-read-only`, `connected-read-only`, later separately approved modes), source references and extraction dates, target environment, skill/policy/schema/code/model versions, output path, retention class, reviewer, start/end/status.

### Application record

- Stable canonical ID; canonical name/description; product group and product name; application type/context; consumer/product/customer/tenant/programme/jurisdiction/payment metadata when applicable; environment/lifecycle/criticality; hosting/vendor; ownership/support fields; source systems/record IDs; evidence; confidence; validation status; last observed; notes.

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
- Identify sponsor/product owner, CMDB product steward, security/privacy, data owner, platform/agent engineer, QA/evaluation lead, and human approvers.
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
- Convert the skill’s rules into testable invariants: strong-key order, no name-only merge, no unsupported dependency inference, explicit Unknown/Unassigned, secret exclusion, separate product groups, one connected graph, no live writes.
- Write workflow contracts and report format consistent with the skill’s required response.

**Deliverables:** versioned contracts/policies, workflow specifications, traceability matrix, policy unit tests.

**Exit gate:** product steward and security approve contracts/policies; tests demonstrate deny-by-default operation and reject forbidden modes.

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
- Create an evaluation set covering the Product Groups, edge cases, ambiguous identities, conflicting sources, MTi tenant/deployment, government subsidy jurisdiction/payment model, shared service false positives, environment mismatches, stale evidence, and missing owners.
- Add adversarial input fixtures containing instruction-like text, secret-like values, malformed files, prompt injection, misleading repository references, false “orphan” cases, and cross-portfolio ambiguity.
- Evaluate precision/recall for candidate matching and classification only where labels/ground truth exist; otherwise use expert-rated acceptance, severe-error counts, evidence completeness, calibration and reviewer override analysis.
- Compare deterministic rules, model-assisted suggestions, and human baseline; test model/config upgrades against the regression suite.
- Test access controls, redaction, data minimization, audit completeness, failure, timeout, restart, and recovery.

**Deliverables:** eval report, threat-model updates, regression suite, documented release thresholds and failure criteria.

**Exit gate:** zero critical policy violations in the release suite; performance thresholds and unacceptable-error classes agreed by domain/security owners; all unresolved limitations explicitly accepted or deferred.

### Phase 4 — Controlled pilot on approved data (Weeks 9–12)

**Activities**
- Run on one bounded, approved priority scope in an isolated/non-production context with read-only exports or read-only access.
- Human reviewers inspect record-level proposals, evidence quality, duplicate/exception queues, product group/name assignments, and dependency candidates.
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

**Activities**
- Review data and model quality, reviewer overrides, exceptions, freshness, and user feedback on a defined cadence.
- Add scenarios before broadening scope, adding sources, changing models, changing schemas, or enabling new tools.
- Reassess threat model, data handling, access, and retention after material change.
- Version any change to `skill.md`; migrate schema and prompt fixtures deliberately; maintain backward compatibility or documented migrations.
- Treat write-capability as a separate gated programme increment, not a configuration toggle.

## 8. ADM Workflow lifecycle and human checkpoints

| Workflow stage | Agent output | Required human checkpoint |
|---|---|---|
| Intake | Validated run manifest or rejection with missing items | Confirm scope/data-use authorization where policy requires |
| Source assessment | Catalogue, schemas, counts, freshness and data-quality notes | Source owner confirms intended source/scope and limitations |
| Taxonomy/mapping | Proposed product group/name and target-field mappings | Business/product steward confirms ambiguous classifications and controlled values |
| Reconciliation | Canonical candidates, duplicate/conflict queues, field provenance | Owners decide merge, conflict resolution, identity and ownership assertions |
| CI/deployment mapping | Phase 1: Candidate application-to-resource links | Technical/platform owner validates material mappings |
| Service mapping | Phase 2: Candidate business service definitions and links | Business steward confirms service-to-application support links |
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

- Narrow, approved product decision and business case.
- Formal threat model and penetration test for the connector and agent configuration.
- Exact change-set review and independently recorded human approval (approval transaction).
- Validation of the target production version and schema.
- Non-production rehearsal and recovery plan.
- Release gate for the write-capable product increment.
- Dedicated monitoring and incident-response capability for live system updates.

## 10. Implementation assumptions and open decisions

1. **Model choice:** select and evaluate models based on logic, structured output reliability, context window, and cost.
2. **Infrastructure:** identify an approved, isolated compute and storage environment for Phase 2 prototype.
3. **Data access:** agree the approval route and format for the Phase 0 read-only exports.
4. **Metrics:** agree how to measure utility and performance before the Phase 4 pilot.
5. **Ownership:** Who is the Phase 1 Resource owner vs the Phase 2 Service owner?
