# Data Handling Policy

**Status:** Draft; the organization’s approved classification and retention rules take precedence where stricter.

## Before processing

1. Record source owner, purpose, extraction time, scope, sensitivity, and permitted use in the source catalogue.
2. Use approved exports and the minimum fields needed for the task. Do not fetch Key Vault or equivalent secret values.
3. Exclude or redact credentials, tokens, private keys, connection strings, payment-card data, and unrelated personal/customer data before model submission.
4. If sensitivity or permitted use is unknown, do not submit the data to an external model or connector. Stop and request classification from the data owner.

## During processing

- Keep source data and derived artefacts within the approved storage boundary and access group.
- Prefer deterministic local profiling and transformation; send only task-relevant, minimized fields to a model approved for that data class.
- Treat model prompts, traces, and telemetry as potential data stores. Do not log full rows, secrets, or unrestricted prompt payloads by default.
- Use stable opaque references to source records; preserve a resolvable evidence pointer in the controlled repository rather than duplicating sensitive contents.
- Keep raw source values where explicitly required for reconciliation only in the approved data store; generated reports should contain redacted excerpts or field references.

## After processing

- Apply the approved retention and deletion schedule to inputs, temporary files, prompts, traces, outputs, and backups.
- Restrict generated inventories and dependency maps according to their highest sensitivity; these may expose business and security topology.
- Record output location, access class, retention class, and responsible owner in the run manifest.
- Do not publish artefacts broadly or move them to a less-protected location without approval.

## Suspected exposure

Stop further processing, prevent additional model/tool calls, preserve run IDs and safe metadata, avoid copying the exposed value, and notify the designated security/data owner through the approved incident path. Follow the organization’s incident and retention instructions.
