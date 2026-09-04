# FIX-012 Backup Manifest

- Date: 2026-09-04
- Purpose: Before FIX-012 complete separation of Hypothesis / Edge Lifecycle, Knowledge Aging / Health, Production Promotion Stage, and Risk State
- Scope: semantic dictionary correction only

## Files and pre-change blob SHAs

```text
01_DICTIONARY/OBJECT_DICTIONARY.md
b6ecbeea9a0ac7635d3113c6b1f1e275e9958da9

01_DICTIONARY/STATE_DICTIONARY.md
fdb8d0163b8693701475e839581534b7445937f8
```

## Intended FIX-012 boundary

```text
Hypothesis / Edge Lifecycle
= research maturity

Knowledge Aging / Health
= freshness / current validity health

Production Promotion Stage
= production permission

Risk State
= current OS safety permission
```

## Planned semantic migration

```text
Hypothesis Lifecycle ACTIVE
→ do not use as production-use state; production use belongs to Production Promotion Stage

Hypothesis Lifecycle AGING
→ Knowledge Aging / Health = AGING

Hypothesis Lifecycle SUSPENDED
→ degradation/freshness belongs to Knowledge Aging / Health; production stop belongs to Production Promotion Stage = PAUSED

Edge Lifecycle ACTIVE
→ do not use as production-use state; production use belongs to Production Promotion Stage

Edge Lifecycle DEGRADED
→ Knowledge Aging / Health = DEGRADED

Edge Lifecycle SUSPENDED
→ degradation/freshness belongs to Knowledge Aging / Health; production stop belongs to Production Promotion Stage = PAUSED

Knowledge Aging SUSPENDED
→ remove ambiguous suspension semantics; use DEGRADED for knowledge health and PAUSED for production permission
```

## Object targets

```text
CausalHypothesis
Edge
KnowledgeLifecycleProfile
HypothesisPoolEntry
```

## Files intentionally not changed

```text
01_DICTIONARY/ROLE_DICTIONARY.md
all .py files
```

## Rollback

Restore the two pre-change blob versions above if FIX-012 verification fails.
