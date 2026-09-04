# BEFORE FIX-007 BACKUP MANIFEST

- Date: 2026-09-04
- Purpose: Backup point before FIX-007 MarketEvent generation responsibility repair
- Repository: `daIdaI31104519-ui/-OS-`
- Branch: `main`

## Target files before change

### `01_DICTIONARY/ROLE_DICTIONARY.md`
- Blob SHA: `ba1e3da773c53caf64f55a480978cae6ac6b871f`

### `01_DICTIONARY/OBJECT_DICTIONARY.md`
- Blob SHA: `45352459fa321debd98a09d68d7a3406c707a5d2`

## Intended FIX-007 rule

```text
Observation / TimeSeriesMeasurement / Feature
→ Event Detection Processor
→ MarketEvent
→ Feature Priority / Market Intelligence
→ Causal Engine
```

Rules:
- `MarketEvent` has one semantic generator: `Event Detection Processor`.
- Market Intelligence consumes and interprets MarketEvent but does not create canonical MarketEvent directly.
- Event Detection is a lightweight Role / Component inside the Market Observation Plane, not a new top-level Plane.
- MarketEvent remains fact-oriented; prediction, market interpretation, and causal claims are prohibited in event generation.
- Event Detection must distinguish `no event detected` from `event detection unavailable/degraded`.
- Duplicate observations of the same real-world event must not inflate unique MarketEvent counts.
- Detection rule provenance/version and event timing must be traceable.
- Missed events discovered by Market Intelligence are returned as `UnexplainedEvent` / `ResearchCandidate` for Event Detection research rather than silently creating a canonical event.

This file is a restore manifest. It does not replace Git history.