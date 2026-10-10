# Validator Interface — Draft

This repository contains contracts, not an executable validator implementation. Before runtime use, implement deterministic validation outside the language model and make each result attributable to a run ID and validator version.

## Required checks

1. Parse every JSON Schema as Draft 2020-12; resolve local `$ref` targets; fail closed on unresolved references.
2. Parse workflow YAML safely (no custom tags or object constructors), validate it against `contracts/workflow.schema.json`, and confirm every prompt, policy, schema, and workflow reference exists.
3. Validate run manifests and source/canonical/output records against their contracts; check required identifiers, controlled values, unique keys, cross-record references, and environment consistency.
4. Preserve source IDs and originals; detect duplicate/conflicting IDs without automatically merging or overwriting.
5. Validate evidence links for every proposed relationship and require candidate/exception status when evidence or endpoints are incomplete.
6. Scan inputs and outputs for secret-like data before model submission and artefact publication; use redaction/stop logic without printing the detected value.
7. Validate CSV encoding, header uniqueness, escaping, blank semantics, and stable keys before dry-run export.
8. Verify scope boundaries and declared mode before all file/tool operations; reject write/import/delete tools in the MVP capability set.

## Result format

For every check record: `run_id`, validator name/version, input/artifact reference and fingerprint, check ID, status (`pass`, `fail`, `blocked`, `not_run`), count/summary, redacted error details, and timestamp. Do not store raw sensitive payloads in validator logs.

## Testing notes

Use the privacy-safe scenarios in `evaluations/scenarios.yaml`. The listed fixture references resolve to synthetic test files under `evaluations/synthetic/`; they are examples, not production data. Static contract validation is not proof of access control, prompt-injection resistance, model quality, or production safety; those require runtime tests and a security-reviewed deployment.
