# FIX-013 Backup Manifest

- Date: 2026-09-04
- Purpose: Before FIX-013 formal State Authority separation
- Scope: Role / State / Object dictionary semantic correction only

## Files and pre-change blob SHAs

```text
01_DICTIONARY/ROLE_DICTIONARY.md
efd14b7065d212738c00b6a64dd54886498ec000

01_DICTIONARY/STATE_DICTIONARY.md
3884d5da8527c550f8be59f78fba5a2138ad5044

01_DICTIONARY/OBJECT_DICTIONARY.md
1c0e3c4f493e04b7a1980b453df4eeb0d52ad0a4
```

## Intended FIX-013 boundary

```text
REQUEST
↓
RECOMMEND
↓
APPROVE
↓
APPLY
↓
StateTransitionEvent
```

Formal principles:

```text
Request authority
≠ Recommend authority
≠ Approve authority
≠ Apply authority

1 State Machine
= 原則1 Apply Authority / single-writer responsibility
```

## Safety asymmetry

```text
Safety-restrictive transition
= Emergency / Fast Path may be allowed by explicit authority policy

Recovery / permission expansion
= Strict approval path; must not be auto-restored by a restrictive authority
```

## Intended dictionary changes

```text
ROLE_DICTIONARY.md
- formalize State Authority responsibility model
- controllers are domain-internal Apply responsibilities, not new top-level layers
- AI / Logger / Telegram / Post-Trade do not directly write Current State
- shared State Transition Engine is mechanism, not authority

STATE_DICTIONARY.md
- replace broad/ambiguous State Authority section with Request / Recommend / Approve / Apply matrix
- define one-writer principle
- define restrictive Fast Path vs recovery/expansion Strict Path
- keep final ApprovalDecision Object deferred to FIX-015

OBJECT_DICTIONARY.md
- extend StateTransitionEvent authority provenance with recommended_by_role / recommendation_ref
- clarify authorized_by_role = approval/authorization responsibility and applied_by_role = actual Apply authority
- do not create ApprovalDecision Object in FIX-013
```

## Files intentionally not changed

```text
all .py files
all DB schema files
```

## Rollback

Restore the three pre-change blobs above if FIX-013 verification fails.
