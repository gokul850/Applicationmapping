# ADM AI Agent Harness — Draft Configuration

**Status:** Draft for review; not executable authorization.
**Domain authority:** [`../skill.md`](../skill.md)
**Implementation plan:** [`../implementation-plan.md`](../implementation-plan.md)

This folder contains a vendor-neutral starter configuration for bounded AI assistance with CMDB/ADM work. It specifies policies, typed contracts, workflows, and prompt templates. It does **not** implement connectors, orchestration, validators, or a Device42 importer. No file in this folder grants access to any system.

## Operating modes

- `offline-read-only`: analyze explicitly supplied files; no network access.
- `connected-read-only`: future mode requiring separately approved, allow-listed read-only adapters and credentials.
- `dry-run`: validate a proposed export against a verified target schema without submitting it.

The initial safe default is `offline-read-only`. Write, import, delete, merge, and production-change modes are intentionally absent.

## Configuration map

- `policies/`: non-negotiable safety, data-handling, and approval rules.
- `contracts/`: JSON Schemas for run manifests, source records, canonical records, evidence, relationships, decisions, exceptions, and workflow definitions.
- `workflows/`: ordered, bounded tasks with declared inputs/outputs, tools, and human checkpoints.
- `prompts/`: task prompts with explicit untrusted-input boundaries and structured-output requirements.
- `evaluations/`: risk-based acceptance and regression scenarios.

## Run sequence

1. Validate the run manifest and approved scope before reading inputs.
2. Inventory only declared sources; record inaccessible or excluded sources as not inspected.
3. Profile and normalize data without discarding original values or provenance.
4. Generate suggestions as `candidate` or `probable`; use `confirmed` only when its evidence and required human validation satisfy the skill.
5. Run deterministic validators. Invalid or unsupported items go to an exception queue; do not silently repair identity or relationships.
6. Present findings, exact artefacts, limitations, human decisions, and the next safe action.
7. For Device42, stop at preflight/dry-run until the target version and schema are verified. Production import requires a separately authorized change process outside this draft.

## Versioning and implementation notes

Pin the harness commit, policy, schemas, prompts, workflow, and model/runtime configuration in every run record. Runtime adapters must fail closed if a mode, tool, source, schema, or approval state is missing. Keep canonical schemas independent of Device42-specific import templates. Map the contract fields to the installed version only after target validation.

Before operational use, the implementation team must add executable schema/workflow validators, authentication and authorization controls, secure storage, retention, observability, incident handling, and the tests in `evaluations/test-plan.md`. Static validity of these files is not proof that runtime controls work.

## Draft workflow and prompt inventory

Four workflows are defined: `workflows/source-assessment.yaml`, `workflows/application-reconciliation.yaml`, `workflows/dependency-discovery.yaml`, and `workflows/device42-preflight.yaml`. Each declares its inputs, contract references, ordered steps, allowed tools, failure behavior, outputs, and any human gate. The workflow status remains `draft` and the YAML files are not executable until an orchestrator and runtime policy enforcement are implemented.

Prompt templates are in `prompts/`: `source-assessor.md`, `reconciliation-analyst.md`, `dependency-analyst.md`, and `report-writer.md`. Each prompt is bounded to one task, marks source content as untrusted, and names a typed JSON output contract. Prompts do not grant tools or authority.

`evaluations/scenarios.yaml` references synthetic fixtures in `evaluations/synthetic/`. They demonstrate policy expectations only; they are not representative production data and do not by themselves constitute a complete model evaluation.
