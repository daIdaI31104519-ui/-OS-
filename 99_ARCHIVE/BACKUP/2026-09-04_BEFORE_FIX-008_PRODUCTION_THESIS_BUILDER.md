# BEFORE FIX-008 BACKUP MANIFEST

- Date: 2026-09-04
- Purpose: Backup point before FIX-008 ApplicableHypothesisSet / TradeThesis generation responsibility repair
- Repository: `daIdaI31104519-ui/-OS-`
- Branch: `main`

## Target files before change

### `01_DICTIONARY/ROLE_DICTIONARY.md`
- Blob SHA: `99f8775530b84402a405a4c3fa149c379a495060`

### `01_DICTIONARY/OBJECT_DICTIONARY.md`
- Blob SHA: `906f190239138164e2d5665f84150e47a7aac26e`

## Intended FIX-008 rule

```text
Approved Hypothesis / Edge Pool
+ Current MarketContext
+ Current Confirmed MarketDNA
+ ApplicabilityProfile
+ Constraint
+ HypothesisAssessmentProfile
+ KnowledgeLifecycleProfile
+ Production Promotion Stage
+ Quality / Uncertainty
↓
Production Thesis Builder
↓
ApplicableHypothesisSet
↓
TradeThesis
↓
External AI Review (optional)
↓
Signal Engine
```

Rules:
- `ApplicableHypothesisSet` and `TradeThesis` have one semantic generator: `Production Thesis Builder`.
- Builder uses only Approved / production-eligible Knowledge; Draft / Researching / unapproved hypotheses cannot enter a live thesis.
- Builder may express `expected_direction / expected_effect / expected_horizon`, but does not create final `BUY / SELL / NO_TRADE` decisions; that remains Signal Engine responsibility.
- `Approved != Applicable != Trade-worthy`.
- Hypothesis roles remain `PRIMARY / SUPPORTING / CONDITIONAL / CONTRADICTING`; majority vote is prohibited.
- Shared evidence, dependency, redundancy, and common-cause relationships must be preserved rather than double counted.
- Constraint, Production Promotion Stage, Knowledge Aging, Quality, and Uncertainty must be checked during applicability selection.
- If a valid thesis cannot be built, the Builder must not fabricate one; non-buildability is distinct from `SignalDecision.NO_TRADE`.
- New AI ideas or unapproved hypotheses return to Research as `ResearchCandidate` instead of entering the current Production thesis.
- The Builder is one Production Role with internal selection/composition submodules, not two new top-level layers.

This file is a restore manifest. It does not replace Git history.