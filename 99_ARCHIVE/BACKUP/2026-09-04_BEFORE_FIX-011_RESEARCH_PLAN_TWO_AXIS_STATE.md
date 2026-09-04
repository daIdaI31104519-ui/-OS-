# BEFORE FIX-011 BACKUP MANIFEST

- Date: 2026-09-04
- Purpose: Backup point before FIX-011 ResearchPlan two-axis state alignment
- Repository: `daIdaI31104519-ui/-OS-`
- Branch: `main`

## Target file before change

### `01_DICTIONARY/OBJECT_DICTIONARY.md`
- Blob SHA: `806c4dcf8e98a17dbc4bb27fd5b570cfb86cd81b`

## Reference file checked but not planned to change

### `01_DICTIONARY/STATE_DICTIONARY.md`
- Blob SHA: `fdb8d0163b8693701475e839581534b7445937f8`
- Reason: FIX-3 and FIX-010 already define ResearchPlan Lifecycle and ResearchPlan Lock as separate State Machines and connect successful transitions to StateTransitionEvent.

## Intended FIX-011 rule

```text
ResearchPlan Identity
+ Plan Version
+ Lifecycle State
+ Lock State
```

The two State Machines remain independent:

```text
ResearchPlan Lifecycle
DRAFT → READY → ACTIVE → COMPLETED / SUPERSEDED / CANCELLED

ResearchPlan Lock
EDITABLE → PRE_REGISTERED → FROZEN
```

Rules:
- `lifecycle_state != lock_state`.
- `ACTIVE + FROZEN` is valid.
- `frozen_at` is timestamp metadata, not the source of truth for lock state.
- Lifecycle and Lock transitions use separate `StateTransitionEvent` histories.
- A FROZEN plan is not semantically overwritten. Meaningful changes require a new Plan Version.
- ResearchTrial must bind to the exact ResearchPlan reference and Plan Version used at trial start.
- For protected modes such as DEMO_FORWARD / protected holdout, contracts may require the plan to be FROZEN before T0.
- Old trials must never be reinterpreted as if they used a newer ResearchPlan Version.

This file is a restore manifest. It does not replace Git history.