# FIX-014 Backup Manifest

- Date: 2026-09-04
- Purpose: Before FIX-014 separation of per-retrieval SourceMetadata, Source Lifecycle, and Data Quality
- Scope: Role / State / Object dictionary semantic correction only

## Files and pre-change blob SHAs

```text
01_DICTIONARY/ROLE_DICTIONARY.md
4f77d7e64728d8839e5fc586e1cb904418ff22ad

01_DICTIONARY/STATE_DICTIONARY.md
df5c3438a83d2b8e287a8095dc2c811a520b9288

01_DICTIONARY/OBJECT_DICTIONARY.md
5f8e8a1462132f53fc9bfcb83c511741dbf2c373
```

## Intended FIX-014 boundary

```text
SourceMetadata
= one retrieval / receive fact and provenance

Source Lifecycle
= current operational / governance state of a Logical Source or Provider Source

Data Quality
= trustworthiness of the retrieved data content
```

Formal principle:

```text
Retrieval Result
≠ Source Lifecycle
≠ Data Quality State
```

## Planned semantic migration

```text
SourceMetadata.status
→ Legacy ambiguous field. New design uses retrieval_status for per-retrieval outcome.

Source Lifecycle state
→ remains in STATE-SRC-001 and Current State / StateTransitionEvent mechanism.

Data Quality
→ remains QualityProfile / STATE-DQ-001 responsibility.
```

## Logical / Provider Source boundary

```text
Logical Source
≠ Provider Source

Example:
OPEN_INTEREST logical source = ACTIVE
Provider A = UNAVAILABLE
Provider B = FALLBACK
```

Provider failure must not automatically retire the market-understanding concept itself.

## Intended dictionary changes

```text
OBJECT_DICTIONARY.md
- redefine SourceMetadata as immutable per-retrieval provenance
- replace ambiguous status with retrieval_status / retrieval_reason_codes
- add logical_source_ref / provider_source_ref
- allow source lifecycle snapshot-at-capture references without making them lifecycle source of truth
- clarify retrieval success != data quality healthy

STATE_DICTIONARY.md
- clarify SOURCE_LIFECYCLE is source-level current state, not request result
- one failed retrieval does not automatically imply DEGRADED / UNAVAILABLE
- one successful retrieval does not automatically restore ACTIVE
- source recovery follows FIX-013 strict recovery path

ROLE_DICTIONARY.md
- Source Adapter / Collector generate retrieval metadata and lifecycle evidence/request material
- they do not directly Apply Source Lifecycle transitions unless explicitly acting through the assigned Apply Authority
```

## Files intentionally not changed

```text
all .py files
all DB schema files
```

## Rollback

Restore the three pre-change blobs above if FIX-014 verification fails.
