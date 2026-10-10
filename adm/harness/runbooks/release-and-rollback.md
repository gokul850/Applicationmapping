# Harness Release and Rollback Runbook

**Status:** Draft; no deployment target is selected by this file.

## Release prerequisites

- Approved change review for the exact commit and deployment environment.
- Contract/workflow validation and all applicable regression/adversarial tests pass.
- Evaluation report records dataset scope, thresholds, failures, limitations, and reviewer sign-off.
- Policy, schema, workflow, prompt, code, and model/runtime versions are pinned in the release record.
- Access/tool allow-list remains read-only; no production write credential or import tool is present in the MVP.
- Monitoring, retention, support owner, incident contact route, and rollback/disable procedure are ready.

## Promotion

Promote immutable artifacts through the organization’s approved environments. Do not edit prompt or policy content in place after evaluation. Record release ID, commit, approvals, environment, model/runtime configuration, deployment time, and smoke-test results.

## Rollback or disable

1. Stop new runs and revoke/disable the deployed agent or affected connector using the approved platform procedure.
2. Preserve run/audit metadata and quarantine outputs generated since the last known-good release.
3. Revert to the previous reviewed release only if it remains compatible with current schemas/policies; otherwise remain disabled.
4. Re-run smoke, policy, and regression tests before resuming.
5. Record the decision, impact, recovery result, and follow-up issue.

Rollback returns the harness to an earlier reviewed version; it does not reverse any external system change. This draft has no system-write operation.
