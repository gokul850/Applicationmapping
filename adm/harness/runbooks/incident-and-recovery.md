# Harness Incident and Recovery Runbook

**Status:** Draft; route details and contacts must be filled by the organization before operational use.

## Trigger examples

Secret or prohibited data detected; source scope exceeded; unexpected write-capable tool/endpoint available; untrusted input attempts to override policy; unauthorized model destination; material output misstatement; or suspected production impact.

## Immediate response

1. Stop the affected run and prevent further model/tool calls. Do not retry automatically.
2. Disable or revoke the affected connector/session if safe and within the responder’s authority; otherwise escalate to the service owner.
3. Preserve run ID, time, source references, policy/code/model versions, tool/action metadata, and safe error details. Do not copy secret values into the incident record.
4. Notify the designated security/data owner and platform/service owner through the approved incident channel.
5. Contain and assess exposure under organizational incident procedures; follow retention and legal/privacy instructions.
6. Quarantine outputs until their accuracy and sensitivity are reviewed. Mark the run blocked and affected artefacts as untrusted.
7. Resume only after a human incident owner authorizes remediation and the control/test gap is closed.

## Recovery evidence

Record incident owner, scope, containment, affected runs/artefacts, notifications, remediation, regression tests, approval to resume, and any required data deletion or user notification. This file does not define organization-specific contact details or override its incident response plan.
