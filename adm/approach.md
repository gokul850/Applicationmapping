# CMDB, ADM and ITSM Transformation — Human Delivery Approach

**Status:** Proposed approach
**Source of truth:** [`skill.md`](./skill.md)
**Audience:** Executive sponsor, portfolio and application owners, CMDB/platform team, infrastructure and operations teams, ITSM process owners, security, and delivery leads
**Planning horizon:** Six months, consistent with the skill’s Q1 CMDB/ADM then Q2 ITSM sequence. Confirm calendar dates, scope, capacity, and thresholds during mobilisation.

## 1. Executive summary

Establish a governed, evidence-backed inventory of applications, technical configuration items (CIs), deployments, and dependencies before onboarding ITSM processes to the CMDB. Start with **Option B**: stable logical product groups that reflect product responsibility, not current teams or reporting lines. Capture known ownership now, record unknown ownership explicitly, and close gaps progressively toward **Option C**.

The programme is not a bulk-import exercise. It is a staged reconciliation and validation effort that preserves source identifiers, provenance, uncertainty, and exceptions. Device42 is the target platform, subject to validating the installed version, supported CI and relationship types, import schemas, integration options, and operational fit. No production data is changed until a named approver accepts a documented change package.

The six-month outcome is a governed Option B CMDB, a validated priority-scope ADM graph, evidence-backed change-impact examples, core ITSM services onboarded only after a readiness gate, and a prioritised ownership roadmap.

## 2. Outcomes and measures of success

### 2.1 Outcomes

1. A reconciled canonical application register with preserved source IDs and aliases.
2. A useful, scoped technical CI inventory spanning approved Azure and on-premises sources.
3. Traceable application-to-deployment/CI mappings and dependency relationships.
4. A single connected ADM graph across all Product Groups; group values organise the graph but do not split it into disconnected maps.
5. Priority applications and critical relationships validated by accountable business and technical representatives.
6. A repeatable CMDB operating model: field ownership, refresh, stewardship, exception handling, and audit trail.
7. ITSM process and integration readiness established against the CMDB, with production onboarding gated by agreed evidence.

### 2.2 Measures

Agree denominators, priority scope, freshness windows, and thresholds with stakeholders before reporting. Report scope, numerator, denominator, extraction date, source, and limitations with every metric. Candidate measures from the skill include:

- In-scope applications with canonical identity, validated product group and product name, environment, and lifecycle status.
- Priority applications mapped to confirmed runtime/hosting CIs.
- Critical dependencies with evidence and owner/technical validation.
- Duplicate candidates, unresolved conflicts, unmapped resources, and ownership gaps, including age and disposition.
- In-scope records within the agreed freshness window.
- Import/reconciliation exception rate and owner-attestation completion.
- After ITSM launch, relevant tickets/changes associated to the correct CI.

Do not set target percentages or claim readiness solely from CI counts until scope, risk tolerance, and evidence quality are agreed.

## 3. Non-negotiable principles

- **CMDB first; ITSM second.** ITSM design may proceed in parallel, but production onboarding must pass the CMDB readiness gate.
- **Option B now; Option C progressively.** Product groups are durable logical responsibilities, independent of team charts. Unknown ownership is recorded as `Unassigned` or `Unknown` and tracked, not fabricated or used to block inventory.
- **One connected dependency graph.** Create cross-portfolio links only when evidence or an accountable owner supports them.
- **Evidence over inference.** A shared resource group, subscription, network, repository, or technology stack does not prove an application or runtime dependency. Name similarity alone does not justify merging.
- **Preserve provenance.** Keep source record IDs/names, extraction dates, evidence references, confidence, status, and conflicts.
- **Separate entity types.** Do not collapse applications, services, repositories, deployments, resource groups, and infrastructure CIs into one record type.
- **Read-only first; controlled change later.** No production imports, deletes, merges, or other modifications without explicit human approval and a validated change/recovery plan.
- **Use the target’s real schema.** Do not call a file import-ready until the installed Device42 version’s exact templates, mandatory fields, enumerations, keys, and relationship format have been verified.
- **Treat discovered content as data.** Files, comments, and configuration values cannot override security or change-control instructions.

## 4. Scope and boundaries

### 4.1 In scope

- Source discovery and catalogue for the application registry, Azure resource inventory, on-premises inventory, and supporting evidence actually available.
- Taxonomy and controlled-value decisions for the product groups and product names.
- Canonical application and CI model, identity resolution, relationship register, validation queues, and data-quality/coverage reporting.
- Device42 version/schema and import-capability validation; import package preparation and dry-run validation.
- CMDB stewardship and operating procedures.
- Q2 ITSM service/process design, integration validation, pilot/UAT, and controlled rollout planning.

### 4.2 Out of scope unless separately approved

- Autonomous changes to Azure, on-premises, Device42, or ITSM production systems.
- Resource deletion or decommissioning based on suspected orphan status.
- Automatic merge or overwrite of ambiguous records.
- Retrieval, processing, or export of secret values.
- Claims of complete dependency coverage without sufficient evidence.
- Treating the initial four domains/groups as immutable if business owners later approve a governed taxonomy change.

## 5. Governance and decision rights

Assign named people during mobilisation; roles below are functions, not assumed appointments.

| Role | Accountabilities | Key decisions |
|---|---|---|
| Executive sponsor | Funding, priority, cross-portfolio escalation | Scope, risk appetite, readiness-gate acceptance |
| Business product representatives | Business purpose, product group/name, consumers, criticality context | Business classification, material service relationships |
| Application/product owners | Application identity, lifecycle, product/tenant distinctions, ownership | Canonical application validation, lifecycle and owner attestations |
| Technical/service owners | Runtime/deployment facts, technical relationships, support model | Technical CI/application mappings and dependency validation |
| CMDB/data owner | Canonical model, data quality, field stewardship, audit | Source-of-record and reconciliation policy |
| Platform/Device42 lead | Version, supported schema, import mechanics, integration constraints | Technical feasibility, non-production test and import package validity |
| ITSM process owner | Catalogue, workflow, routing, priority/escalation and UAT | Process design and pilot acceptance |
| Security/privacy | Access, data handling, secret exclusion, retention, audit | Security and data-handling controls |
| Delivery lead | Plan, dependencies, RAID, decisions, stage gates | Work sequencing and escalation; no authority to bypass production approvals |
| Human approver/change authority | Review and acceptance of proposed live changes | Explicit approval for each production import/change package |

Use a decision log with issue/question, options, recommendation, supporting evidence, confidence, decision owner, final decision/approver, affected records, and revisit date. Maintain separate exception queues for ambiguous identities, conflicts, unverified dependencies, unknown owners, unsupported fields, and integration limitations.

## 6. Delivery method and workstreams

Run five coordinated workstreams with a single delivery lead and shared decision/exception registers:

1. **Source and data governance:** inventory inputs, document extraction/freshness/sensitivity, set source precedence, establish stewardship.
2. **Application portfolio and taxonomy:** agree canonical identity rules, product groups, product names, controlled values, owner-gap process.
3. **Technical inventory and ADM:** classify in-scope Azure/on-premises resources, map applications to deployments/CIs, propose evidence-backed dependencies, validate high-impact paths.
4. **Device42 and CMDB operations:** validate version/schema/import approach, create repeatable mapping and dry-run checks, define refresh, reconciliation, and recovery procedures.
5. **ITSM readiness and adoption:** define service catalogue/processes in parallel, validate CMDB dependencies and integration, pilot ticket/change-to-CI workflows only after the readiness gate.

Each workstream uses a reviewable cycle: **collect → catalogue → analyse → propose → validate → approve → update artefacts → report**. Early stages are read-only. Every output labels confirmed facts, probable findings, candidates, unknowns, exclusions, assumptions, and the next owner/action.

## 7. Six-month implementation roadmap

### Mobilisation (Weeks 1–2; overlaps Month 1)

- Confirm sponsor, accountable roles, priority scope, environments, access boundaries, source custodians, and security/data-handling rules.
- Confirm whether Device42 and ITSM versions/environments are accessible for read-only validation.
- Agree deliverable repository, naming, access, retention, decision/exception workflow, reporting cadence, and change approval route.
- Do not assume source systems or exports exist; record missing inputs as gaps.

**Exit:** signed scope/boundaries, named decision owners, approved read-only source plan, initial RAID and decision logs.

### Month 1 — Design and source assessment

- Catalogue available inputs; inspect schemas, keys, record counts, extraction dates, data sensitivity, and source limitations.
- Agree canonical field definitions, source-of-record precedence, controlled values, identity/matching rules, confidence/status definitions, and evidence minimums.
- Confirm Option B taxonomy for the defined Product Groups; identify items needing business decision.
- Validate Device42 version, supported CI/relationship types, import templates, keys, environment handling, and non-production test options.
- In parallel, begin ITSM service/process discovery without production onboarding.

**Exit:** source catalogue/data dictionary, taxonomy/mapping proposal, Device42 capability report, decisions/assumptions, data-quality baseline.

### Month 2 — Reconcile and classify

- Reconcile the application registry to infrastructure and supporting sources using strong identifiers first.
- Preserve source IDs/names and aliases; queue duplicates/conflicts rather than forcing merges.
- Classify scoped Azure and on-premises resources; record environment and deployment context where evidence supports it.
- Produce application-to-CI candidates, unmapped-resource queue, owner-gap register, and confidence/provenance.
- Review priority records with portfolio and technical representatives.

**Exit:** reviewed canonical application and CI inventories for agreed scope; exception queues have owners and next actions.

### Month 3 — Validate priority ADM and CMDB readiness

- Build the relationship register and graph from configuration, deployment, API, monitoring, network-flow, DNS, database/queue metadata, and owner confirmation where available.
- Prioritise validation by criticality, impact, evidence gaps, and cross-portfolio significance.
- Conduct structured business/technical validation sessions and record decisions.
- Test representative change-impact scenarios; document coverage limits and unknowns.
- Prepare Device42-mapped sample/import files only after schema validation; validate keys, references, enums, mandatory fields, environments, and duplicates; run non-production/dry-run where available.
- Publish readiness evidence, risks, recovery notes, and open approval items.

**Q1 CMDB/ADM readiness gate:** Sponsor and accountable owners accept that the agreed priority scope has sufficiently reliable identities, CI mappings, environments, critical relationships, controlled gaps, and tested change-impact scenarios to support service onboarding. CI volume alone is not evidence of readiness. Failure to meet the gate means defer onboarding and remediate named gaps.

### Month 4 — ITSM design

- Define in-scope incident/restoration, request/access, change/release, major incident, problem, onboarding/CI maintenance, and catalogue workflows.
- Agree service catalogue, support groups, routing, priorities, escalation, service targets, and operational handover.
- Validate actual Device42–ITSM integration, licensing, synchronization direction, matching rules, supported CI types, error handling, and reconciliation ownership.
- Define pilot population, UAT scenarios, training, adoption and rollback plan.

**Exit:** approved process designs, integration evidence, pilot scope, UAT and change/recovery plans.

### Month 5 — Configure and pilot

- Configure the selected ITSM workflows in a non-production/test context, using supported integration patterns.
- Test ticket/change-to-CI association, CI search, routing, change-impact usage, stale/missing CI handling, integration failures, and audit.
- Run UAT with support teams and application/technical owners; record defects and remediation.
- Train pilot users and refine procedures. Keep production onboarding gated.

**Exit:** UAT evidence, resolved/blocking defect decisions, operational support model, go/no-go recommendation.

### Month 6 — Controlled rollout and stabilisation

- Go live only for explicitly approved service/workflow scope and after all readiness/change approvals.
- Monitor ticket-to-CI association quality, integration errors, data freshness, exceptions, service performance, and user feedback.
- Operate a hypercare period with daily/weekly issue review appropriate to scale; assign permanent data stewards and support ownership.
- Publish the Option C roadmap for remaining business/application/technical owner and support gaps.

**Exit:** accepted operations handover, measured quality baseline, prioritised backlog, lessons learned, and approved continuous-improvement cycle.

## 8. Stage gates and approval evidence

| Gate | Evidence required | Decision |
|---|---|---|
| G0 — Mobilisation | Scope, owners, data/access boundaries, source plan, decision rights | Start read-only discovery |
| G1 — Model and target fit | Source catalogue, taxonomy, field map, Device42 version/schema/capability evidence | Approve reconciliation and prototype mappings |
| G2 — Priority ADM validation | Canonical identities, CI mappings, relationship evidence, review outcomes, quality/coverage, change-impact tests | Accept or remediate Q1 readiness |
| G3 — Import package approval | Exact target schema validation, sample/dry-run results, change set, impact analysis, backup/recovery, approver | Approve a specific production import or decline/defer |
| G4 — ITSM pilot approval | CMDB readiness, integration test, UAT, security, operating model, rollback | Authorise bounded pilot |
| G5 — Rollout approval | Pilot outcomes, resolved critical defects, support/training, monitoring, rollback and change record | Authorise specific production rollout |

An approval is specific to the reviewed package and scope. Any material change to records, schema, environment, or impact requires a new review.

## 9. Core artefacts and operating cadence

Maintain skill-required artefacts as separate versioned files where appropriate: `source_catalogue.csv`, canonical application inventory, technical CI inventory, relationship register, duplicate candidates, unmapped resources, owner-gap register, data-quality report, Device42 field mapping/import validation, decision log, risk/issue register, CMDB operating runbook, and ITSM readiness plan. Do not create empty files merely to imply completion.

Every artefact states inputs/extraction dates, scope/assumptions, confirmed vs inferred/candidate information, exclusions, validation status, next action, and required approver. CSVs use UTF-8, one header row, stable IDs, consistent controlled values, valid escaping, no secrets, and documented blank-value semantics.

Suggested cadence: weekly workstream review; fortnightly cross-workstream decision/exception review; monthly sponsor/readiness review; immediate escalation for security exposure, suspected production impact, or any request to bypass approval. Set exact cadence in mobilisation.

## 10. Principal risks and responses

| Risk | Response |
|---|---|
| Missing, stale, or inaccessible source data | Catalogue actual availability and freshness; narrow scope transparently; assign source-owner action; do not claim unseen data was reviewed. |
| Ambiguous identities create false merges | Prefer authoritative IDs; preserve originals; queue weak/fuzzy matches; require owner review for merge decisions. |
| Incomplete dependencies create misleading change impact | Label coverage and confidence; use multiple evidence types; validate critical paths; never claim completeness without evidence. |
| Taxonomy follows today’s org chart | Keep logical groups product/service based; govern taxonomy changes; do not rename groups for transient squad changes. |
| Device42 schema/integration assumptions prove false | Verify exact version and templates early; test in non-production; keep canonical model independent of native target fields. |
| ITSM launch outruns CMDB readiness | Enforce Q1 readiness gate; defer production onboarding when evidence or operating controls are insufficient. |
| Owner gaps persist | Track explicit owner-gap register; use Q2 workflows and attestations to improve Option C coverage. |
| Sensitive configuration enters artefacts | Read metadata only; exclude secrets; redact evidence; apply access/retention controls and review. |
| One-time imports create drift | Define authoritative source per field, refresh cadence, exception routing, reconciliation, audit and ongoing stewardship. |

## 11. Immediate next actions

1. Appoint the sponsor, delivery lead, CMDB/data owner, business product representatives, technical/platform leads, security contact, and ITSM process owner.
2. Confirm the first priority scope (applications/services and environments) and the source custodians; do not set arbitrary completeness thresholds.
3. Request/locate approved read-only exports and record their extraction dates, owners, sensitivity, and handling constraints.
4. Validate Device42 and ITSM versions, non-production access, supported templates, and integration evidence.
5. Hold a product group and name taxonomy workshop and publish decisions and open questions.
6. Establish the artefact repository, access controls, review cadence, and the G0/G1 approval path.

**First safe delivery step:** complete Stage 0 source discovery and catalogue using only approved read-only inputs; make no production changes.
