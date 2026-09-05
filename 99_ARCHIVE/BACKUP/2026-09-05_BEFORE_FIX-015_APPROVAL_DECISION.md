# FIX-015 Backup Manifest

- Date: 2026-09-05
- Purpose: Before FIX-015 formalization of ApprovalDecision and approval provenance
- Scope: Role / State / Object dictionary semantic correction only

## Files and pre-change blob SHAs

```text
01_DICTIONARY/OBJECT_DICTIONARY.md
27823e4436fbaf2f6d14f88e7790185463a380cc

01_DICTIONARY/STATE_DICTIONARY.md
765a2450976b42a0054f09c7144895b5e9b4a30c

01_DICTIONARY/ROLE_DICTIONARY.md
4f77d7e64728d8839e5fc586e1cb904418ff22ad
```

## Intended FIX-015 boundary

```text
Recommendation
≠ ApprovalDecision
≠ StateTransitionEvent
≠ AuditEvent

APPROVE
≠ APPLIED
```

ApprovalDecision is an immutable governance decision bound to an exact target / version / state machine / transition. It records APPROVE / REJECT / HOLD and does not itself change Current State.

## Planned semantic rules

```text
1 Authority = 1 ApprovalDecision
1 ApprovalDecision = normally one transition authorization
Multi-Approval = multiple ApprovalDecision objects
APPROVE decisions are scope / version / expiry / supersession checked before Apply
ApprovalDecision is normally single-use
Safety restrictive Fast Path still records approval provenance
Recovery / permission expansion uses stricter approval
```

## Planned Object changes

```text
OBJECT_DICTIONARY.md
- add OBJ-GOV-001 ApprovalDecision
- add ApprovalDecision to immutable object family / object list
- connect StateTransitionEvent to approval_decision_refs: []
- treat authorization_ref / authorized_by_role as legacy single-approval provenance
- keep ApprovalDecision separate from StateTransitionEvent and AuditEvent
- remove ApprovalDecision from intentionally-not-objectized list
```

## Planned State changes

```text
STATE_DICTIONARY.md
- APPROVE responsibility produces ApprovalDecision
- Apply requires valid required ApprovalDecision set
- multi-approval, scope intersection, expiry, supersession and single-use semantics
- emergency restrictive approval can be automated-policy based
- recovery / permission expansion requires stricter approval
```

## Planned Role changes

```text
ROLE_DICTIONARY.md
- existing Approve Authorities generate ApprovalDecision
- existing Apply Authorities validate / consume ApprovalDecision references
- no new universal Approval Manager / top-level approval layer
- AI / Logger / Telegram remain non-approvers unless separately authorized by explicit policy
```

## Files intentionally not changed

```text
all .py files
all DB schema files
DATA_CONTRACT.md
```

## Rollback

Restore the three pre-change blobs above if FIX-015 verification fails.
