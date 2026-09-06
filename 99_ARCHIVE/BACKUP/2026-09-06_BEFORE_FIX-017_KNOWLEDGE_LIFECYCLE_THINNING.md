# BEFORE FIX-017 — KnowledgeLifecycleProfile Thinning

Date: 2026-09-06
Purpose: Backup manifest before FIX-017.

## Scope

Only these Canonical dictionaries are intended to change:

```text
01_DICTIONARY/OBJECT_DICTIONARY.md
01_DICTIONARY/STATE_DICTIONARY.md
```

No Role Dictionary, Python, DB schema, DATA_CONTRACT, or other implementation files are part of FIX-017.

## Pre-change blobs

```text
01_DICTIONARY/OBJECT_DICTIONARY.md
a0da2aac3e343f69a80d69f71424d3bd3954476d

01_DICTIONARY/STATE_DICTIONARY.md
8eae9fc8042e43f1aeb2edd5cf37eb73c4164af2
```

## Pre-change KnowledgeLifecycleProfile fields

```yaml
knowledge_lifecycle_id:
target_object_ref:
created_at:
last_validated_at:
last_demo_pass_at:
last_live_evidence_at:
revalidation_due_at:
knowledge_aging_state:
state_machine_version:
latest_transition_event_ref:
degradation_reason_codes: []
review_reason_codes: []
```

## Pre-change Knowledge Aging / Health states

```text
FRESH
CURRENT
AGING
STALE
DEGRADED
ARCHIVED
```

## FIX-017 intended semantic change

```text
KnowledgeLifecycleProfile
= Current freshness / health / revalidation projection only

Research maturity
= Hypothesis / Edge Lifecycle

Evidence history
= ResearchResult / EvidencePackage / ProductionEvidence etc.

Production permission
= Production Promotion

Current runtime risk permission
= RiskState

Storage / archive placement
= Storage Lifecycle / Retention Governance
```

Planned new Canonical state candidates:

```text
FRESH
CURRENT
AGING
STALE
DEGRADED
UNKNOWN
```

Planned field change:

```text
last_demo_pass_at
→ Legacy / evidence-source history; remove from new Canonical profile

last_live_evidence_at
→ Legacy / evidence-source history; remove from new Canonical profile

freshness_basis_refs: []
→ references to evidence/results that justify the current freshness/health projection

ARCHIVED
→ remove from Knowledge Aging / Health state; archive/storage responsibility belongs to Storage Lifecycle
```

## Critical invariants to preserve

```text
STALE ≠ RETIRED
DEGRADED ≠ RETIRED
DEGRADED ≠ PAUSED
Knowledge Aging / Health ≠ Production Promotion
Knowledge Aging / Health ≠ Risk State
Knowledge Aging / Health ≠ Storage Lifecycle
```

Historical records using old fields/states must remain readable under their old schema/version. Do not infer or rewrite historical evidence timestamps or ARCHIVED state into a new meaning without an explicit migration contract.