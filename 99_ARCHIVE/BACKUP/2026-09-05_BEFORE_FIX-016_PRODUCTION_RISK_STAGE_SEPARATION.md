# FIX-016 Backup Manifest

- Date: 2026-09-05
- Purpose: Before FIX-016 separation of Knowledge Production Promotion permission from current Risk permission
- Scope: Object / State dictionary semantic correction only
- Base head before backup: `c376a1f13c2a4a62535746a45f9f647c5405400d`

## Files and pre-change blob SHAs

```text
01_DICTIONARY/OBJECT_DICTIONARY.md
3edc2040b2837746db7a0561111ed2b33e80c10d

01_DICTIONARY/STATE_DICTIONARY.md
8eae9fc8042e43f1aeb2edd5cf37eb73c4164af2
```

## Intended FIX-016 boundary

```text
HypothesisPoolEntry.max_production_stage
→ authorized_production_stage_ceiling

RiskState.allowed_trade_stage
→ RETIRED legacy field
```

Canonical separation:

```text
production_stage
= current Knowledge-specific Production Promotion Stage

authorized_production_stage_ceiling
= maximum Production Promotion Stage currently authorized for that Knowledge

RiskState.state
= current OS / portfolio / market-instance Risk permission

RiskState.allowed_exposure
= current exposure permission under Risk policy
```

## Invariants to preserve

- Production Promotion Stage ≠ Risk State
- Risk deterioration must not rewrite Knowledge `production_stage`
- Knowledge-specific `PAUSED` ≠ system-wide `NO_NEW_ENTRY / EMERGENCY`
- `production_stage` must not exceed `authorized_production_stage_ceiling` for ordinal promotion stages
- `PAUSED` is not an ordinal promotion level for min/max calculations
- `authorized_production_stage_ceiling` does not auto-promote `production_stage`
- `allowed_trade_stage` must not be migrated into `authorized_production_stage_ceiling`
- Historical objects remain readable under their original schema version
- No Role / Python / DB schema change in FIX-016

## Rollback

Restore the exact pre-change blobs above or reset to the pre-FIX-016 head recorded in this manifest if rollback is required.
