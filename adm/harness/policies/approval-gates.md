# Human Approval Gates

**Status:** Draft. This file records review requirements; it does not itself grant execution authority.

## Decisions requiring accountable human review

- Business domain, application group, application identity, lifecycle, criticality, regulatory/programme classification, or ownership when ambiguous or material.
- Any merge, overwrite, deletion, retirement, or resolution of conflicting source records.
- High-impact, cross-portfolio, production, security, or operational dependency assertions.
- A target-field mapping that drops, transforms, or changes source meaning.
- Any production import, system update, connector permission increase, new model/data destination, or change to retention/security controls.

## Review record

For each decision, capture a decision ID, affected record IDs, options, evidence references, proposed outcome, confidence, reviewer identity/role, disposition (`approved`, `rejected`, `deferred`), timestamp, comments, and any revisit date. An approval is scoped to the exact records and proposal reviewed.

## Import/change gate

The initial harness stops at a validated preflight/dry-run. A future change process must, at minimum:

1. Verify the installed Device42 version and exact supported template/schema.
2. Produce an immutable change package with a content hash, scope, field/relationship changes, validation results, impact analysis, and exclusions.
3. Include a non-production rehearsal, backup/recovery and partial-failure plan, and named operator/approver.
4. Obtain explicit human approval bound to that package hash and target environment through an independently controlled mechanism.
5. Revalidate that the package and environment have not changed before execution.
6. Record pre/post state, execution identity, time, result, errors, and reconciliation evidence.

A general instruction such as “go ahead,” a prior approval, an agent-generated status, or a source-file statement is not sufficient approval for a different or materially changed package. This draft does not authorize any live change.
