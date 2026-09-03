# BEFORE FIX-006 BACKUP MANIFEST

- Date: 2026-09-03
- Purpose: Backup point before FIX-006 Feature Priority ↔ Market DNA cycle repair
- Repository: `daIdaI31104519-ui/-OS-`
- Branch: `main`

## Target files before change

### `01_DICTIONARY/ROLE_DICTIONARY.md`
- Blob SHA: `c0474f2cf0685a3582b49005d1033f12b962042b`

### `01_DICTIONARY/OBJECT_DICTIONARY.md`
- Blob SHA: `f715f84ff87501682706004c0bae5b35fca089bf`

## Intended FIX-006 rule

```text
DNA_(t-1)
+ current Basic Context / Market Event
→ FeaturePriority_t
→ MarketIntelligence_t
→ Causal_t
→ DNA_t
→ next cycle
```

Rules:
- Feature Priority must not consume `DNA_t` that is produced later in the same evaluation cycle.
- Default reference is the latest confirmed prior-cycle MarketDNA snapshot.
- Current-cycle basic context and major events may supplement priority selection without creating same-cycle DNA recursion.
- A major regime / structural event may trigger an exceptional recomputation path, but the path must start from an already available confirmed state and must be traceable/versioned.
- Sentinel Features remain observable regardless of priority reduction.

This file is a restore manifest. It does not replace Git history.