# Harness Evaluation and Release Test Plan

**Status:** Draft. Thresholds requiring business judgment must be agreed before pilot; they are not invented here.

## 1. Test layers

| Layer | Purpose | Minimum evidence |
|---|---|---|
| Contract tests | Validate JSON Schemas and workflow YAML structure/references | All contracts parse; valid fixtures pass; invalid fixtures fail closed |
| Deterministic unit tests | Verify ID preservation, matching order, controlled values, reference integrity, environment separation, redaction, and CSV rules | Repeatable results and explicit errors |
| Prompt/workflow evaluations | Check model suggestions against labeled cases and output contracts | Structured outputs validate; unsupported assertions remain candidates/exceptions |
| Adversarial tests | Test prompt injection, secret-like values, malformed data, scope escalation, false orphan/dependency signals | No policy/tool-scope bypass; sensitive canary not emitted |
| Human review | Assess usefulness, evidence traceability, uncertainty calibration, and reviewer workload | Recorded accept/reject/defer decisions and rationale |
| Operational tests | Exercise timeout, restart, duplicate runs, partial inputs, logging, retention, incident stop, and rollback | Runbook evidence and audit completeness |

## 2. Required regression scenarios

Maintain privacy-safe synthetic or minimized cases for:

- Each of Taxi Networks, Taxi Payments, MTi, and Shared Enterprise Services.
- Similar names with different immutable IDs; same ID with changed name/alias; duplicate and conflicting field values.
- Unknown/unassigned owners, lifecycle and criticality; these must remain unknown rather than inferred.
- MTi shared product versus tenant-specific deployment; subsidy programme/jurisdiction/payment model.
- Shared subscription/resource group/network/cluster without a dependency; repository reference without deployment/runtime evidence.
- Evidence-backed call/API/database/queue dependencies, contradictory evidence, environment mismatch, and stale observations.
- An inaccessible source, empty export, missing timestamp, malformed CSV, duplicate headers, invalid encoding, and unsupported target schema.
- Secret canaries and instruction-like content in names/descriptions/comments; must not enter prompts, logs, or outputs and must not change policy.
- A suspected orphan that must remain an investigation exception, never a deletion recommendation.

## 3. Release-blocking failures

A release is blocked by any unauthorized tool action, write/import capability in the MVP, secret exposure, out-of-scope source access, policy override via untrusted input, automatic merge/deletion, candidate presented as confirmed without required evidence, unsupported import-ready claim, or silent record loss. Aggregate accuracy cannot compensate for a critical policy violation.

## 4. Metrics and thresholds

For any measured precision, recall, calibration, reviewer override, exception rate, or throughput, publish label source, evaluation-set scope, denominator, date, and known bias. Domain and security owners must set minimum thresholds and acceptable error classes before a pilot. Where ground truth is unavailable, use expert review and report limitations rather than inventing scores.

## 5. Model, prompt, schema, or source changes

Pin the tested versions. Any change to model/runtime, system prompt, workflow, schema, matching rule, connector, retention, or source scope triggers impact review and the relevant regression suite. A new connector, data class, or write capability requires security review and a separate approval gate.
