# OBJECT_DICTIONARY.md

# 市場理解OS 情報Object辞書・完全設計

## 0. 文書情報

- 文書種別: DICTIONARY / OBJECT SOURCE OF TRUTH
- 状態: CANONICAL DICTIONARY CANDIDATE
- 上位ルール:
  - `00_GOVERNANCE/GIT_RULES.md`
  - `00_GOVERNANCE/DESIGN_CHANGE_RULES.md`
- 関連辞書:
  - `01_DICTIONARY/ROLE_DICTIONARY.md`
- 対象: 市場理解OS内で生成・保存・受け渡し・研究・本番利用される情報Object
- 目的: Objectの意味・Owner・生成元・必須情報・Version・Trace・不変条件・保存方針を統一し、同じ意味のデータを別名・別形式で無秩序に増殖させない
- 最上位原則: **Objectは「情報の意味」を表し、Role / Contract / View / Experiment Modeと混同しない**

---

# 1. Object Dictionaryの目的

市場理解OSを何十年以上運用する場合、コードより長く残るのは研究結果・市場Case・Hypothesis・Failure・Trade履歴等のKnowledgeである。

そのためObjectは、その時点のPython class名だけではなく、将来別言語・別DB・別Providerへ移行しても意味を失わない論理定義として管理する。

本書で解決すること:

1. Objectが何を意味するか
2. どのRoleが生成・所有するか
3. どのObjectを入力として作られたか
4. どのVersion / Formula / Schemaで生成されたか
5. Raw Evidenceまで遡れるか
6. Objectが変更可能か、Snapshotとして固定されるか
7. Research / Demo / Liveの証拠を混ぜない
8. Knowledgeを重複コピーせず再利用する
9. 数年後・数十年後もMigration可能にする
10. 後続のData Contract / DB Schema / Python Modelの基準を作る

---

# 2. Role / Object / Contract / View / Modeの境界

## Role

処理・判断・管理を行う責任主体。

例:

```text
Collector
Market Intelligence
Causal Engine
Research Orchestrator
Production Thesis Builder
Signal Engine
Runtime
```

## Object

Roleが生成・保存・受け渡す情報構造。

例:

```text
RawData
Feature
Evidence
CausalHypothesis
MarketDNA
ResearchResult
ApplicableHypothesisSet
TradeThesis
EntryThesis
ProductionEvidence
StateTransitionEvent
OrderIntent
TradeResult
```

## Contract

ObjectをRole間でどう受け渡すかを定義する規則。

例:

```text
Data Contract
Processing Contract
Error Contract
Research Contract
Trade Thesis Contract
```

## View

同じKnowledge Object群を別の目的から参照する方法。

例:

```text
Case Library
Market Memory
Failure Museum
Knowledge Graph
```

## Experiment Mode

Research Trialの実験方式。

例:

```text
RANDOM_BASELINE
HISTORICAL
REPLAY
PAPER
DEMO_FORWARD
SHADOW
COUNTERFACTUAL
```

重要:

> ViewやModeのために同じObjectをコピーして別DBへ重複保存しない。

---

# 3. Object分類

市場理解OSのObjectを大きく次へ分類する。

```text
A. COMMON / CONTROL OBJECTS
A-3. STATE / TRANSITION OBJECTS
B. OBSERVATION / DATA OBJECTS
C. MEASUREMENT / FEATURE OBJECTS
D. MARKET UNDERSTANDING OBJECTS
E. CAUSAL / DNA OBJECTS
F. RESEARCH OBJECTS
G. KNOWLEDGE OBJECTS
H. PRODUCTION / TRADING OBJECTS
I. POST-TRADE / FEEDBACK OBJECTS
J. PLATFORM / OPERATIONS OBJECTS
```

---

# 4. 全Persistent Object共通Metadata

DB等へ永続保存する主要Objectは、原則として次を持つ。

```yaml
object_id:
object_type:
schema_version:
object_version:
created_at:
updated_at:
valid_from:
valid_until:
status:
market_profile_id:
trace_id:
parent_object_ids: []
source_refs: []
provenance:
quality_ref:
uncertainty:
created_by_role:
code_version:
config_version:
```

全Objectへ無条件ですべてを複製するのではなく、Objectの性質に応じて必須 / 任意をContractで定義する。

FIX-012以降、`status`というGeneric FieldをHypothesis / Edgeの研究成熟度・Knowledge Aging / Health・Production Promotion・Risk Stateをまとめる万能Fieldとして使用しない。

---

# 5. IDとTraceの共通原則

最低限、次を混同しない。

```text
Object ID
= 一つの情報Objectを識別

Trace ID
= OS内部の処理系列を追跡

Market Event ID
= 実市場で起きた現象を識別

Trial ID
= 研究試行を識別

Trade ID
= 実取引または取引結果を識別

Trade Thesis ID
= 取引論拠を識別

Hypothesis ID
= 研究仮説を識別

State Transition Event ID
= 実際に適用された一つのState遷移事実を識別
```

同じMarket Eventを複数Hypothesisで研究しても、Unique Market Event数を水増ししない。

---

# 6. Object Version原則

## Immutable Snapshot型

生成後に原則変更しない。

例:

```text
RawData
MarketEvent
Feature
ResearchResult
ApplicableHypothesisSet
TradeThesis
EntryThesis
OrderIntent
ExecutionRecord
TradeResult
ProductionEvidence
StateTransitionEvent
```

修正が必要な場合は元Objectを上書きせず、新Version / Correction Object / Superseded関係を使う。

## FIX-005 正式Object名 / Legacy Alias

過去の草案・会話・Role定義で使われた次の表現は、新規設計上の独立Object名として使用しない。

```text
FeatureResult
→ Feature

ResearchTrialResult
Trial Result
Historical Validation Result
OOS Validation Result
Regime Validation Result
Demo Validation Result
Validation Result
→ ResearchResult

Research Test Request
→ ResearchCandidate

CausalEdge / EmpiricalEdge を別Objectとして使用
→ Edge + edge_type = CAUSAL_EDGE / EMPIRICAL_EDGE
```

## Versioned Mutable Knowledge型

研究進行により状態が変わる。

例:

```text
CausalHypothesis
MarketDNA definition
FeatureKnowledge
FormulaKnowledge
Constraint
```

変更履歴を失わない。

## Ephemeral型

Runtime内部の一時状態として扱え、永続保存を必須としない。

---

# A. COMMON / CONTROL OBJECTS

# OBJ-COM-001: MarketProfile

## Meaning

一つの市場・Instrumentを、Coreを書き換えず切り替えるための論理設定Object。

## Owner

Outer Control / Configuration

## Main Fields

```yaml
market_profile_id:
asset_class:
base_asset:
quote_asset:
symbol:
venue_type:
exchange_or_provider:
timezone:
price_precision:
quantity_precision:
minimum_order_size:
market_specific_rules:
data_sources: []
execution_enabled:
profile_version:
```

## Invariants

- 市場理解ロジックをProfileへ埋め込まない
- API Key等Secretを保存しない
- BTC / ETH / SOL等の差は可能な限りProfile / Adapterで吸収する

---

# OBJ-COM-002: SourceMetadata

## Meaning

外部データが「どこから、いつ、どの状態で取得されたか」を示すProvenance Object / Value Object。

## Owner

Collector / Source Adapter

## Main Fields

```yaml
source_id:
provider:
endpoint_or_stream:
requested_at:
received_at:
source_timestamp:
latency_ms:
status:
schema_version:
raw_hash:
request_id:
```

## Invariants

Source情報を下流で失わない。

---

# OBJ-COM-003: QualityProfile

## Meaning

入力情報をどこまで信用できるかを示す品質Object。

## Owner

Data Quality

## Main Fields

```yaml
quality_status:
quality_profile_version:
issues: []
missing_ratio:
duplicate_state:
latency_state:
freshness:
outlier_state:
timestamp_drift:
cross_source_consistency:
source_health:
confidence_limit:
```

## Invariants

- `PASS / FAIL`だけに圧縮しない
- Quality低下を下流でHigh Confidenceへ勝手に戻さない
- Quality Scoreを万能な謎スコアとして扱わない

---

# OBJ-COM-004: Diagnostics

## Meaning

正常結果とは別に、変換・計算・実行中に起きた警告・補正・異常を保持する補助Object。

## Owner

各Role

## Main Fields

```yaml
diagnostic_id:
severity:
category:
message_code:
details:
related_object_ids: []
created_at:
recoverable:
```

## Invariants

Primary ResultとDiagnosticsを混同しない。

---

# A-2. CONTROL / POLICY OBJECTS

# OBJ-CTL-001: RuntimeCommand

## Meaning

Outer / Human InterfaceからRuntimeへ渡すDesired State変更要求。

## Owner

Outer Control

## Main Fields

```yaml
command_id:
command_type: START | PAUSE | RESUME | STOP | STATUS | SAFE_SHUTDOWN
market_profile_id:
requested_by:
requested_at:
reason:
confirmation_level:
```

## Invariants

Command自体がProcessを直接操作しない。Runtimeが検証・実行する。

---

# OBJ-CTL-002: GlobalRiskLimit

## Meaning

市場・口座・Portfolio全体で許可するRisk上限のPolicy Object。

## Owner

Risk Governance / Outer Control

## Main Fields

```yaml
risk_policy_id:
max_exposure:
max_position_size:
max_drawdown_policy:
max_consecutive_loss_policy:
max_open_positions:
kill_switch_condition:
effective_from:
policy_version:
```

## Invariants

Signalが上限を書き換えない。

---

# A-3. STATE / TRANSITION OBJECTS

# OBJ-STATE-001: StateTransitionEvent

## Meaning

State Machine上で実際に成功・適用された一回のState遷移を、誰が・なぜ・何を根拠に・どのVersionのState Machineで変更したかまで含めて保存するImmutable Historical Event Object。

## Generator

当該State Machineで正式にTransitionを適用したAuthorized State-owning Role / Authority。

## Custodian

Logger / Audit Storage。

## Main Fields

```yaml
state_transition_event_id:
target_object_ref:
state_machine_id:
state_machine_type:
state_machine_version:
from_state:
to_state:
expected_previous_state:
transition_sequence:
trigger_refs: []
reason_codes: []
requested_by_role:
authorized_by_role:
applied_by_role:
requested_by_actor:
transitioned_at:
effective_at:
created_at:
manual_override_ref:
authorization_ref:
previous_transition_event_ref:
trace_id:
```

## Invariants

- 成功・適用されたTransitionだけを記録する
- `state_machine_type`ごとの意味境界を失わない
- FIX-012では同一Knowledgeに対するLifecycle / Knowledge Aging / Production Promotionを別々のStateTransitionEvent系列として保持する

---

# B. OBSERVATION / DATA OBJECTS

# OBJ-DATA-001: RawData

## Meaning

外部Sourceから取得した加工前の一次証拠。

## Owner

Collector

## Main Fields

```yaml
raw_data_id:
source_metadata:
market_profile_id:
payload:
source_timestamp:
received_at:
raw_hash:
schema_version:
```

## Invariants

- 原本を上書きしない
- Featureで置換しない
- Source / Timestampを失わない
- 後から再計算できる形を優先する

---

# OBJ-DATA-002: Observation

## Meaning

Raw DataをNormalizerが標準形式へ変換し、研究・比較可能な単一観測値として表すObject。

## Owner

Normalizer

## Main Fields

```yaml
observation_id:
metric_or_field:
value:
unit:
observed_at:
standard_symbol:
market_profile_id:
raw_data_refs: []
normalization_version:
conversion_history: []
diagnostics_ref:
quality_ref:
trace_id:
```

---

# OBJ-DATA-003: MarketEvent

## Meaning

Observation / TimeSeriesMeasurement / Featureから検出された、「市場で何かが発生した」という事実寄りの現象を一意に識別するImmutable Event Object。

## Owner / Generator

`ROLE-EVT-001: Event Detection Processor`

## Main Fields

```yaml
market_event_id:
event_type:
market_profile_id:
detected_at:
start_at:
peak_at:
end_at:
detection_rule_ref:
detection_rule_version:
detector_version:
observation_refs: []
time_series_measurement_refs: []
feature_refs: []
source_refs: []
event_magnitude:
quality_ref:
confidence:
detector_health_at_detection:
deduplication_key:
canonicalization_refs: []
trace_refs: []
```

---

# OBJ-DATA-004: NormalizedObservation [RETIRED / MERGED]

## Status

RETIRED / MERGED INTO `OBJ-DATA-002: Observation`

---

# OBJ-DATA-005: TimeSeriesMeasurement

## Meaning

観測値を時間変化として扱うための系列測定Object。

## Owner

Time Series Processor

## Main Fields

```yaml
measurement_id:
metric_id:
value:
sampling_interval:
measurement_window:
lookback:
valid_until:
input_refs: []
quality_ref:
formula_version:
```

---

# C. MEASUREMENT / FEATURE OBJECTS

# OBJ-CALC-001: FormulaDefinition

## Meaning

正式な計算方法をVersion付きで表すDefinition Object。

## Owner

Calculation / Measurement Service

## Main Fields

```yaml
formula_id:
formula_version:
name:
expression_or_algorithm_ref:
input_definition: []
output_definition:
unit_rule:
time_window_rule:
assumptions: []
status:
validated_evidence_refs: []
created_at:
deprecated_at:
```

---

# OBJ-CALC-002: MeasurementResult

## Meaning

FormulaDefinitionを特定入力へ適用した計算結果。

## Owner

Calculation / Measurement Service

## Main Fields

```yaml
measurement_result_id:
formula_id:
formula_version:
value:
unit:
input_refs: []
time_window:
quality_ref:
created_at:
trace_id:
```

---

# OBJ-FEAT-001: Feature

## Meaning

市場理解・研究に利用できるVersioned Feature値。

## Owner

Feature Generator

## Main Fields

```yaml
feature_id:
feature_definition_id:
feature_version:
value:
unit:
formula_id:
formula_version:
input_refs: []
time_window:
quality_ref:
confidence:
created_at:
valid_until:
trace_id:
```

---

# OBJ-FEAT-002: FeaturePriorityProfile

## Meaning

現在Evaluation Cycleで、各Featureをどの程度注目する価値があるかを表す優先度Object。

## Owner

Feature Priority

## Main Fields

```yaml
priority_profile_id:
evaluation_cycle_id:
market_profile_id:
current_context_ref:
current_event_refs: []
previous_confirmed_dna_ref:
previous_confirmed_dna_as_of:
dna_reference_mode:
horizon:
selected_features: []
low_priority_features: []
sentinel_features: []
priority_reasons: []
redundancy_map:
quality_constraints:
recompute_reason:
priority_version:
created_at:
valid_until:
```

---

# D. MARKET UNDERSTANDING OBJECTS

# OBJ-MI-001: MarketContext

## Meaning

Market Intelligenceが「今、市場で何が起きているか」を構造化したContext Object。

## Owner

Market Intelligence

## Main Fields

```yaml
market_context_id:
market_profile_id:
as_of:
horizon:
price_structure:
volatility_context:
liquidity_context:
derivatives_context:
participant_flow_context:
institutional_context:
macro_context:
news_event_context:
cross_market_context:
market_event_refs: []
evidence_refs: []
quality_ref:
uncertainty:
context_version:
```

---

# OBJ-MI-002: Evidence

## Meaning

仮説・市場解釈・研究判断を支持または反証する追跡可能な証拠Object。

## Owner

Market Intelligence / Research

## Main Fields

```yaml
evidence_id:
evidence_type:
direction: SUPPORT | CONTRADICT | NEUTRAL
claim_scope:
source_object_refs: []
market_event_refs: []
time_scope:
quality_ref:
strength_profile:
uncertainty:
evidence_source_channel:
created_at:
```

---

# OBJ-MI-003: Contradiction

## Meaning

現在の説明・Hypothesis・Trade Thesisを弱める反証材料を明示するObject。

## Owner

Market Intelligence / Causal Engine / Research

## Main Fields

```yaml
contradiction_id:
target_object_id:
evidence_refs: []
severity: SOFT | HARD | UNCLASSIFIED
mechanism:
observed_at:
quality_ref:
uncertainty:
```

---

# OBJ-MI-004: UnexplainedEvent

## Meaning

重要な市場現象だが、現在のKnowledgeでは十分説明できないことを明示する研究資産。

## Owner

Market Intelligence

## Main Fields

```yaml
unexplained_event_id:
market_event_refs: []
observed_context_ref:
known_explanations_checked: []
why_unexplained:
priority_candidate:
research_candidate_ref:
```

---

# E. CAUSAL / DNA OBJECTS

# OBJ-CAUSAL-001: CauseCandidate

## Meaning

あるEffectを生じさせた可能性がある、未検証の原因候補。

## Owner

Causal Engine

## Main Fields

```yaml
cause_candidate_id:
target_effect_id:
candidate_factor:
supporting_evidence_refs: []
contradicting_evidence_refs: []
temporal_order_state:
lag_candidate:
confounder_candidates: []
alternative_hypothesis_refs: []
quality_ref:
uncertainty:
status:
```

---

# OBJ-CAUSAL-002: EffectDefinition

## Meaning

Causal Researchで説明・検証対象とする結果側の現象定義。

## Owner

Causal Engine

## Main Fields

```yaml
effect_id:
effect_type:
target_metric_or_event:
direction:
magnitude_rule:
horizon:
measurement_formula_ref:
market_event_refs: []
```

---

# OBJ-CAUSAL-003: CausalHypothesis

## Meaning

Cause Candidate / Effect / Mechanism / 反証条件を統合した検証可能な仮説Object。FIX-012以降、Hypothesis本体のStateは研究成熟度だけを表し、Knowledge Aging / Health・Production Promotion・Risk Stateを混ぜない。

## Owner

Causal Engine / Knowledge Domain

## Main Fields

```yaml
hypothesis_id:
hypothesis_version:

hypothesis_lifecycle_state:
hypothesis_lifecycle_state_machine_version:
hypothesis_latest_transition_event_ref:
knowledge_lifecycle_ref:

cause_candidate_refs: []
effect_ref:
mechanism:
expected_direction:
expected_horizon:
lag_definition:
confounder_refs: []
alternative_hypothesis_refs: []
supporting_evidence_refs: []
contradicting_evidence_refs: []
assessment_profile_ref:
applicability_ref:
market_dna_scope:
research_result_refs: []
created_at:
last_validated_at:
revalidation_due_at:
```

## FIX-012 State Separation

```text
CausalHypothesis.hypothesis_lifecycle_state
= Research maturity only

KnowledgeLifecycleProfile
= freshness / health

HypothesisPoolEntry.production_stage
= production permission

RiskState
= current OS safety permission
```

Hypothesis Lifecycle正式候補:

```text
DRAFT
RESEARCHING
SUPPORTED
WEAK
CONFLICTED
APPROVED
RETIRED
REOPENED
```

## Invariants

- `status = ACTIVE`のような万能StateでProduction可否を表さない
- `AGING / STALE / DEGRADED`をHypothesis Lifecycleへ入れない
- `PAUSED / NORMAL_LIVE`をHypothesis本体へ入れない
- Knowledge Aging / Healthは`knowledge_lifecycle_ref`から別管理する
- Production利用段階はHypothesisPoolEntry側で管理する
- 一回のLoss / Live failureだけで研究上のAPPROVED履歴を自動消去しない
- Historical / OOS / Demo / Liveを混ぜない
- Alternative / Contradictionを保持する

---

# OBJ-CAUSAL-004: HypothesisAssessmentProfile

## Meaning

Hypothesisの健全性を単一Mystery Scoreへ潰さず、多次元で評価するObject。

## Owner

Causal Engine / Research / Post-Trade

## Candidate Dimensions

```yaml
temporal_support:
contradiction_strength:
confounder_risk:
historical_support:
oos_support:
demo_forward_support:
regime_stability:
live_evidence:
staleness:
uncertainty:
unique_event_count:
```

---

# OBJ-DNA-001: MarketDNA

## Meaning

現在または過去の市場状態を比較・検索・研究できる正規化された圧縮表現。

## Owner

Market DNA Role / Knowledge Domain

## Main Fields

```yaml
market_dna_id:
evaluation_cycle_id:
dna_definition_version:
market_profile_id:
as_of:
horizon:
previous_market_dna_ref:
axis_values:
axis_quality:
input_feature_refs: []
formula_refs: []
nearest_case_refs: []
nearest_similarity:
novelty_profile:
quality_ref:
uncertainty:
```

---

# OBJ-DNA-002: RegimeProfile

## Meaning

Market DNA / 市場状態を、ProductionやResearchで扱いやすい粗い分類へ写像する派生Object候補。

## Owner

Market DNA / Research

## Main Fields

```yaml
regime_profile_id:
market_dna_ref:
regime_labels: []
classification_rule_version:
confidence:
valid_until:
```

---

# F. RESEARCH OBJECTS

# OBJ-RSCH-001: ResearchCandidate

## Meaning

Researchへ送るべき未検証課題・疑問・異常・仮説候補を統一して表す入口Object。

## Owner

Research Router / Research Orchestrator

## Main Fields

```yaml
research_candidate_id:
candidate_type:
origin_role:
origin_object_refs: []
question:
expected_value_of_research:
urgency:
risk_relevance:
data_availability:
duplicate_candidate_refs: []
status:
created_at:
expiry_or_revisit_at:
```

---

# OBJ-RSCH-002: ResearchPlan

## Meaning

何を、どのEvidence Channel・Dataset・評価基準で検証するかを事前定義するVersioned研究計画Object。

## Owner

Research Orchestrator

## Main Fields

```yaml
research_plan_id:
plan_version:
research_candidate_ref:
target_object_refs: []
research_question:
hypothesis_or_claim:
experiment_modes: []
pre_registered_metrics: []
entry_criteria:
continue_criteria:
early_stop_criteria:
completion_criteria:
data_scope:
holdout_policy:
resource_budget_ref:
lifecycle_state:
lifecycle_state_machine_version:
lifecycle_latest_transition_event_ref:
lock_state:
lock_state_machine_version:
lock_latest_transition_event_ref:
pre_registered_at:
frozen_at:
supersedes_plan_ref:
superseded_by_plan_ref:
created_at:
updated_at:
```

---

# OBJ-RSCH-003: ResearchTrial

## Meaning

特定ResearchPlan Versionに基づいて実行される一つの試行。

## Owner

Experimental Framework

## Main Fields

```yaml
trial_id:
research_plan_ref:
research_plan_version:
plan_lifecycle_state_at_start:
plan_lifecycle_transition_ref:
plan_lock_state_at_start:
plan_lock_transition_ref:
experiment_mode:
evidence_source_channel:
t0:
dataset_or_event_scope:
unique_market_event_refs: []
model_formula_feature_versions:
execution_model_version:
started_at:
finished_at:
status:
```

---

# OBJ-RSCH-004: ResearchResult

## Meaning

ResearchTrialで観測された結果を保存する正式な共通研究結果Object。

## Owner

Research / Validation Framework

## Main Fields

```yaml
research_result_id:
trial_id:
primary_result:
metrics:
diagnostics:
quality_ref:
uncertainty:
evidence_source_channel:
unique_event_count:
limitations: []
contradictions: []
produced_candidate_refs: []
created_at:
```

---

# OBJ-RSCH-005: EvidencePackage

## Meaning

Hypothesis / Edge / Formula等をApproval判断へ送る際、複数Research EvidenceをSource別に束ねるPackage。

## Owner

Validation / Knowledge Promotion

## Main Fields

```yaml
evidence_package_id:
target_object_id:
historical_result_refs: []
oos_result_refs: []
regime_result_refs: []
demo_forward_result_refs: []
stress_result_refs: []
live_evidence_refs: []
contradiction_refs: []
limitations: []
assessment_profile:
created_at:
```

---

# OBJ-RSCH-006: ApplicabilityProfile

## Meaning

あるHypothesis / Edge / Formulaが「どの条件で利用可能か」を表すObject。

## Owner

Research / Knowledge Domain

## Main Fields

```yaml
applicability_id:
target_object_id:
market_profiles: []
market_dna_conditions:
regime_conditions:
time_conditions:
horizon_conditions:
quality_requirements:
constraint_refs: []
known_exclusions: []
last_validated_at:
```

---

# OBJ-RSCH-007: Edge

## Meaning

Researchで再現可能性・期待値・適用条件が確認された利用可能な優位性Object。FIX-012以降、Edge LifecycleはEdgeとしての研究成熟度だけを表し、Knowledge Aging / Health・Production Promotion・Risk Stateを混ぜない。

## Owner

Research / Knowledge Promotion

## Types

```text
CAUSAL_EDGE
EMPIRICAL_EDGE
```

## Main Fields

```yaml
edge_id:
edge_type:

edge_lifecycle_state:
edge_lifecycle_state_machine_version:
edge_latest_transition_event_ref:
knowledge_lifecycle_ref:

source_hypothesis_refs: []
mechanism_or_pattern:
expected_effect:
expected_horizon:
expected_value_profile:
applicability_ref:
evidence_package_ref:
constraint_refs: []
last_validated_at:
```

## FIX-012 State Separation

Edge Lifecycle正式候補:

```text
CANDIDATE
VALIDATING
SUPPORTED
APPROVED
RETIRED
REOPENED
```

## Invariants

- `ACTIVE`をEdge Lifecycleへ入れない
- `DEGRADED / STALE / AGING`をEdge Lifecycleへ入れない
- `PAUSED / NORMAL_LIVE`をEdge本体へ入れない
- Edge health低下はKnowledgeLifecycleProfileへ送る
- Production利用段階はHypothesisPoolEntryへ送る
- 因果説明が強い = EVが正とは限らない
- Empirical EdgeもOOS / Demo等の再現性を要求する

---

# OBJ-RSCH-008: ResearchBudget

## Meaning

ResearchがCPU / RAM / Storage / Trial / 時間を無限消費しないためのResource Policy Object。

## Owner

Research Governance

## Main Fields

```yaml
research_budget_id:
max_concurrent_trials:
max_trial_count:
max_cpu_budget:
max_memory_budget:
max_storage_budget:
max_runtime:
max_hypothesis_set_size:
max_combination_count:
priority_policy_version:
```

---

# G. KNOWLEDGE OBJECTS

# OBJ-KNW-001: MarketCase

## Meaning

再利用可能な一つの過去市場Case。

## Owner

Knowledge Domain

## Main Fields

```yaml
market_case_id:
market_profile_id:
start_at:
end_at:
market_dna_ref:
market_context_refs: []
market_event_refs: []
key_evidence_refs: []
research_result_refs: []
trade_result_refs: []
outcome_summary:
case_tags: []
```

---

# OBJ-KNW-002: FeatureKnowledge

## Meaning

Featureがどの条件・Horizon・DNAで有効 / 無効 / 不安定だったかという再利用可能Knowledge。

## Owner

Knowledge Domain

## Main Fields

```yaml
feature_knowledge_id:
feature_definition_ref:
applicability_ref:
research_result_refs: []
known_strengths: []
known_failures: []
redundancy_relations: []
last_validated_at:
status:
```

---

# OBJ-KNW-003: FormulaKnowledge

## Meaning

Formula Versionごとの市場的妥当性・安定性・Failureを保存するKnowledge。

## Owner

Knowledge Domain

## Main Fields

```yaml
formula_knowledge_id:
formula_ref:
research_result_refs: []
sensitivity_profile:
regime_profile:
stress_profile:
demo_forward_profile:
known_failures: []
status:
last_validated_at:
```

---

# OBJ-KNW-004: Failure

## Meaning

市場理解・研究・Production・Execution・Platformで発生した失敗を再利用可能なKnowledgeとして保存するObject。

## Owner

Knowledge Domain / Post-Trade / Operations

## Main Fields

```yaml
failure_id:
failure_type:
origin_domain:
related_object_refs: []
conditions:
observed_effect:
root_cause_state:
known_root_causes: []
unknowns: []
severity:
reproduction_refs: []
created_at:
```

---

# OBJ-KNW-005: StressResult

## Meaning

Stress / Validation実験で実際に何が起きたかを保存する実験結果Object。

## Owner

Validation Framework

## Main Fields

```yaml
stress_result_id:
target_object_id:
stress_scenario:
input_conditions:
observed_result:
failure_detected:
metrics:
trial_ref:
```

---

# OBJ-KNW-006: FailureBoundary

## Meaning

どの条件を境にHypothesis / Edge / Systemが成立しなくなるかを表す境界Object。

## Owner

Research / Knowledge Domain

## Main Fields

```yaml
failure_boundary_id:
target_object_id:
boundary_variables:
safe_region:
weak_region:
unsafe_region:
unknown_region:
supporting_stress_results: []
uncertainty:
```

---

# OBJ-KNW-007: Constraint

## Meaning

Researchで得たFailureBoundary等をProductionの適用・禁止・修正条件へ変換したSafety Knowledge。

## Owner

Knowledge Promotion / Risk Governance

## Main Fields

```yaml
constraint_id:
target_scope:
constraint_type:
condition:
action_or_effect:
source_failure_boundary_refs: []
evidence_package_ref:
status:
effective_from:
review_due_at:
```

---

# OBJ-KNW-008: NegativeKnowledge

## Meaning

効果がない・再現しない・採用しないこと自体を保存するKnowledge。

## Owner

Knowledge Domain

## Main Fields

```yaml
negative_knowledge_id:
target_object_refs: []
claim:
tested_conditions:
research_result_refs: []
reason:
reopen_condition:
created_at:
last_reviewed_at:
```

---

# OBJ-KNW-009: KnowledgeRelationship

## Meaning

Knowledge Object同士の関係を一元的に表すGraph Edge Object。

## Owner

Knowledge Domain

## Main Fields

```yaml
relationship_id:
source_object_id:
relationship_type:
target_object_id:
evidence_refs: []
confidence:
valid_from:
valid_until:
```

---

# OBJ-KNW-010: KnowledgeLifecycleProfile

## Meaning

Knowledgeの現在の鮮度・再検証必要性・最近のEvidenceに基づく健全性を管理する長期運用Object。FIX-012以降、Edge maturity・Production permission・Risk Stateを持たない。

## Owner

Knowledge Aging Governance

## Main Fields

```yaml
knowledge_lifecycle_id:
target_object_ref:

knowledge_aging_state:
state_machine_version:
latest_transition_event_ref:

created_at:
last_validated_at:
last_demo_pass_at:
last_live_evidence_at:
revalidation_due_at:

degradation_reason_codes: []
review_reason_codes: []
```

正式候補State:

```text
FRESH
CURRENT
AGING
STALE
DEGRADED
ARCHIVED
```

## FIX-012 Removed Fields / Semantics

```text
staleness_state
→ knowledge_aging_stateへ一本化

edge_health_state
→ 削除。Edge maturityはEdge Lifecycle、現在のKnowledge healthはknowledge_aging_stateで表す

current_risk_stage
→ 削除。Risk Stateの責任

reopen_or_suspend_reason
→ 分解し、degradation_reason_codes / review_reason_codes等で意味を限定
```

## Invariants

- 一度SUPPORTED / APPROVEDになったKnowledgeを永久真理として扱わない
- Knowledge `DEGRADED`だけで研究上のAPPROVED履歴を無言で削除しない
- Knowledge Aging / HealthをProduction `PAUSED`と混同しない
- Risk StateをこのObjectへコピーしない

---

# H. PRODUCTION / TRADING OBJECTS

# OBJ-PRD-001: HypothesisPoolEntry

## Meaning

Productionが参照するHypothesis / Edgeの登録情報。FIX-012では研究成熟度・Knowledge Aging / Health・Production Promotion Stageを別参照として保持し、Risk StateはPool Entryへ埋め込まない。

## Owner

Knowledge Promotion / Production

## Main Fields

```yaml
pool_entry_id:

target_knowledge_ref:
target_lifecycle_state_ref:
knowledge_lifecycle_ref:

production_stage:
max_production_stage:

applicability_ref:
constraint_refs: []
assessment_profile_ref:
evidence_package_ref:
expires_or_review_due_at:
```

## FIX-012 Four-Axis Boundary

```text
Target Knowledge Lifecycle
= research maturity

knowledge_lifecycle_ref
= freshness / health

production_stage / max_production_stage
= production permission

RiskState
= current OS safety permission, referenced separately downstream
```

## Migration

旧:

```text
hypothesis_or_edge_ref
hypothesis_state_ref
```

新:

```text
target_knowledge_ref
target_lifecycle_state_ref
```

Edgeも同じPool Entryを利用するため、Hypothesis専用に見えるField名を廃止する。

## Invariants

- `SUPPORTED / APPROVED` と `DEMO_FORWARD / NORMAL_LIVE` を同じstatus列挙へ入れない
- ProductionはLifecycle、Knowledge Aging / Health、Production Stage、Applicability、Constraint、Risk Stateを別々に確認する
- `APPROVED`だけでProduction利用可能と判断しない
- `CURRENT`だけでProduction利用可能と判断しない
- `NORMAL_LIVE`でもRisk StateがNO_NEW_ENTRY / EMERGENCYなら新規Tradeを許可しない
- `max_production_stage` をRisk Stateと混同しない
- 未承認Hypothesis / EdgeをProductionが自由に有効化しない

---

# OBJ-PRD-002: ApplicableHypothesisSet

## Meaning

Approved / production-eligibleなHypothesis / Edgeのうち、現在のMarketContext / MarketDNA / Applicability / Constraint / Quality等へ実際に適用可能なものを役割付きで束ねたImmutable Production Object。

## Owner / Generator

`ROLE-PRD-001: Production Thesis Builder`

## Main Fields

```yaml
hypothesis_set_id:
hypothesis_set_version:
builder_version:
market_profile_id:
market_context_ref:
market_dna_ref:
horizon:
selected_pool_entry_refs: []
applicability_profile_refs: []
assessment_profile_refs: []
knowledge_lifecycle_refs: []
production_stage_snapshot:
primary_hypothesis_refs: []
supporting_hypothesis_refs: []
conditional_hypothesis_refs: []
contradicting_hypothesis_refs: []
shared_evidence_map:
dependency_map:
redundancy_map:
common_cause_map:
constraint_refs: []
selection_reason_codes: []
quality_ref:
uncertainty:
created_at:
valid_until:
```

## Invariants

- `Approved != Applicable != Trade-worthy`
- Lifecycle / Knowledge Aging / Production Stageを別々に確認する
- Risk Stateは最終Safety判断まで独立して保持する
- Shared Evidenceを二重評価しない

---

# OBJ-PRD-003: TradeThesis

## Meaning

Production Thesis BuilderがApplicableHypothesisSetを現在市場で検討可能な一つの取引論拠へ構成したImmutable Object。

## Owner / Generator

`ROLE-PRD-001: Production Thesis Builder`

## Main Fields

```yaml
trade_thesis_id:
trade_thesis_version:
builder_version:
market_profile_id:
market_context_ref:
market_dna_ref:
hypothesis_set_ref:
expected_direction:
expected_effect:
expected_horizon:
expected_value_profile:
primary_hypothesis_refs: []
supporting_hypothesis_refs: []
conditional_hypothesis_refs: []
contradicting_hypothesis_refs: []
supporting_evidence_refs: []
contradiction_refs: []
shared_evidence_map:
dependency_map:
redundancy_map:
common_cause_map:
main_risks: []
invalidation_conditions: []
constraint_refs: []
quality_ref:
uncertainty:
created_at:
valid_until:
```

---

# OBJ-PRD-004: AIReviewResult

## Meaning

外部AIがTrade Thesis全体を独立査読した補助結果Object。

## Owner

External AI Review

## Main Fields

```yaml
ai_review_id:
trade_thesis_ref:
reviewer_id:
reviewer_model_version:
reviewed_at:
recommendation: TRADE | WAIT | REJECT | UNKNOWN
identified_redundancy: []
shared_evidence_concerns: []
contradictions: []
missing_alternatives: []
uncertainty:
new_research_candidate_refs: []
```

---

# OBJ-PRD-005: SignalDecision

## Meaning

Canonical TradeThesisにRiskを取るだけの期待値・適用可能性があるかをSignal Engineが判断したDecision Object。

## Owner

Signal Engine

## Main Fields

```yaml
signal_decision_id:
trade_thesis_ref:
decision: BUY | SELL | NO_TRADE
expected_value_profile:
applicability_state:
constraint_state:
data_quality_state:
uncertainty:
ai_review_refs: []
reason_codes: []
created_at:
valid_until:
```

---

# OBJ-PRD-006: DefenseDecision

## Meaning

Signalが取りたいRiskを、現在安全に取ってよいか判定したSafety Gate Object。

## Owner

Pre-Trade Defense

## Main Fields

```yaml
defense_decision_id:
signal_decision_ref:
decision: ALLOW | REDUCE | BLOCK
risk_state:
constraint_checks: []
data_quality_state:
liquidity_state:
spread_state:
slippage_risk:
drawdown_state:
consecutive_loss_state:
exposure_state:
exchange_health_state:
abnormal_event_state:
reason_codes: []
created_at:
```

---

# OBJ-PRD-007: RiskState

## Meaning

現在OSがどの程度Riskを許可するかを表す横断状態Object。FIX-012では個別Hypothesis / EdgeのProduction Promotion Stageとは完全に分離する。

## Owner

Risk Governance / Defense

## Candidate States

```text
NORMAL
CAUTION
RISK_REDUCED
MICRO_ONLY
NO_NEW_ENTRY
EMERGENCY
```

## Main Fields

```yaml
risk_state_id:
state:
trigger_refs: []
money_dd_state:
execution_health_state:
data_health_state:
market_novelty_state:
allowed_exposure:
allowed_trade_stage:
effective_from:
review_condition:
```

## Invariants

- `RiskState` はKnowledge maturityを表さない
- `RiskState` は個別KnowledgeのProduction Promotion Stageを表さない
- KnowledgeLifecycleProfileへRisk Stateを複製しない
- `NORMAL_LIVE`のKnowledgeが存在してもRiskState = NO_NEW_ENTRYなら新規Entryは禁止

---

# OBJ-PRD-008: OrderIntent

## Meaning

EntryThesisで固定されたProduction判断を、取引所非依存の注文意図として表すImmutable Object。

## Owner / Generator

Execution Logic

## Main Fields

```yaml
order_intent_id:
entry_thesis_ref:
trade_thesis_ref:
signal_decision_ref:
defense_decision_ref:
market_profile_id:
side:
order_type:
requested_size:
price_condition:
time_in_force:
risk_limits:
execution_constraints: []
created_at:
expires_at:
```

---

# OBJ-PRD-009: ExecutionRecord

## Meaning

OrderIntentがExchange Adapterでどう送信・約定されたかを記録する実行証拠Object。

## Owner

Exchange Adapter / Execution

## Main Fields

```yaml
execution_record_id:
order_intent_ref:
exchange_order_id:
submitted_at:
acknowledged_at:
fills: []
requested_price:
average_fill_price:
requested_size:
filled_size:
partial_fill_state:
fees:
slippage:
latency:
exchange_status:
diagnostics_ref:
```

---

# OBJ-PRD-010: EntryThesis

## Meaning

実際にOrderIntentを生成してRiskを取りに行く直前に、TradeThesis・Hypothesis Set・市場状態・Signal・Defense・Risk・Constraint・Versionを固定するImmutable Entry Snapshot。

## Owner / Generator

`ROLE-EXEC-001: Execution Logic / Entry Snapshot Builder`

## Custodian

`ROLE-LOG-001: Logger`

## Main Fields

```yaml
entry_thesis_id:
trade_thesis_ref:
trade_thesis_version:
hypothesis_set_ref:
market_context_ref:
market_dna_ref:
feature_priority_ref:
formula_feature_versions:
evidence_refs: []
shared_evidence_map:
dependency_map:
constraint_refs: []
signal_decision_ref:
defense_decision_ref:
risk_state_ref:
entry_snapshot_at:
snapshot_builder_version:
quality_ref:
uncertainty:
trace_id:
```

---

# OBJ-PRD-011: PositionThesisState

## Meaning

保有中のTrade Thesis健全性を時間経過で追跡する状態Object。

## Owner

Position Supervisor

## Candidate States

```text
HOLD
WATCH
CAUTION
THESIS_WEAKENING
THESIS_INVALIDATED
```

## Main Fields

```yaml
position_thesis_state_id:
trade_id:
entry_thesis_ref:
as_of:
state:
primary_hypothesis_state:
supporting_states:
contradicting_states:
expected_effect_progress:
thesis_decay_state:
reason_codes: []
persistence_duration:
change_magnitude:
```

---

# OBJ-PRD-012: ExitDecision

## Meaning

通常ExitまたはSafety Exitの判断結果Object。

## Owner

Exit Engine / In-Trade Defense

## Main Fields

```yaml
exit_decision_id:
trade_id:
position_thesis_state_ref:
decision_type:
reason:
normal_exit_logic_ref:
in_trade_defense_ref:
created_at:
```

---

# OBJ-PRD-013: ProductionEvidence

## Meaning

ExecutionRecordとLive市場文脈から、実市場でしか得られない約定品質・Slippage・Fee・Funding・Partial Fill・Liquidity Impact・Latency等を構造化したImmutable LIVE Evidence Object。

## Owner / Generator

`ROLE-EXEC-001: Execution Domain / Live Evidence Collector`

## Custodian

`ROLE-LOG-001: Logger`

## Main Fields

```yaml
production_evidence_id:
trade_id:
entry_thesis_ref:
trade_thesis_ref:
execution_record_refs: []
market_context_ref:
market_event_refs: []
fill_quality:
slippage:
fees:
funding:
partial_fill_state:
liquidity_impact:
exchange_latency:
evidence_source_channel: LIVE
evidence_builder_version:
evidence_completeness:
quality_ref:
uncertainty:
diagnostics_ref:
created_at:
trace_id:
```

---

# OBJ-PRD-014: TradeResult

## Meaning

一つの実取引またはSimulation取引の最終Outcome Object。

## Owner

Logger / Post-Trade

## Main Fields

```yaml
trade_id:
trial_id:
experiment_mode:
evidence_source_channel:
entry_thesis_ref:
execution_record_refs: []
entry_time:
entry_price:
exit_time:
exit_price:
position_size:
fees:
slippage:
pnl:
mae:
mfe:
exit_reason:
market_event_refs: []
quality_ref:
```

---

# I. POST-TRADE / FEEDBACK OBJECTS

# OBJ-POST-001: OutcomeAnalysisResult

## Meaning

Trade ResultをWIN / LOSSだけでなく、期待値・Horizon・MAE/MFE・Opportunity Cost等で分析したObject。

## Owner

Post-Trade Analysis

## Main Fields

```yaml
outcome_analysis_id:
trade_result_ref:
classification:
expected_vs_actual:
horizon_evaluation:
mae_mfe_evaluation:
exit_timing_evaluation:
slippage_evaluation:
risk_adjusted_outcome:
similar_case_refs: []
```

---

# OBJ-POST-002: TradeThesisEvaluation

## Meaning

Trade Outcomeとは独立して、Trade Thesis自体が市場説明・期待Effectとして妥当だったかを評価するObject。

## Owner

Post-Trade Analysis

## Main Fields

```yaml
trade_thesis_evaluation_id:
trade_thesis_ref:
trade_result_ref:
thesis_outcome:
expected_effect_occurred:
expected_horizon_met:
invalidation_correct:
contradiction_handling:
applicability_correct:
uncertainty_evaluation:
```

---

# OBJ-POST-003: HypothesisAttribution

## Meaning

Trade Thesisを構成した各Hypothesisが結果へどう寄与したかを個別評価するObject。

## Owner

Post-Trade Analysis

## Main Fields

```yaml
hypothesis_attribution_id:
trade_id:
hypothesis_ref:
role_in_thesis:
expected_effect_occurred:
expected_horizon_met:
evidence_remained_valid:
contradiction_strength:
applicability_correct:
shared_evidence_issue:
dependency_issue:
contribution_assessment:
research_candidate_refs: []
```

---

# OBJ-POST-004: DefenseEvaluation

## Meaning

ALLOW / REDUCE / BLOCKが本当に安全性を高めたかを後から評価するObject。

## Owner

Post-Trade Analysis

## Main Fields

```yaml
defense_evaluation_id:
defense_decision_ref:
shadow_trade_ref:
actual_or_simulated_outcome:
avoided_loss:
missed_profit:
constraint_effectiveness:
false_block_candidate:
research_candidate_refs: []
```

---

# OBJ-POST-005: SupervisorEvaluation

## Meaning

Position Supervisorの警告・Thesis State変化が有用だったかを評価するObject。

## Owner

Post-Trade Analysis

## Main Fields

```yaml
supervisor_evaluation_id:
trade_id:
state_history_refs: []
action_followed:
outcome_after_warning:
false_alarm_candidate:
late_warning_candidate:
research_candidate_refs: []
```

---

# OBJ-POST-006: DemoLiveDivergence

## Meaning

Historical / Demo Forwardでは成立したがLiveで崩れた等、Evidence Channel間の差を研究対象として保存するObject。

## Owner

Post-Trade / Research Router

## Main Fields

```yaml
divergence_id:
target_hypothesis_or_set_ref:
historical_profile:
demo_profile:
live_profile:
execution_difference:
slippage_difference:
liquidity_difference:
regime_difference:
data_quality_difference:
model_error_candidates: []
research_candidate_ref:
```

---

# OBJ-POST-007: CounterfactualResult

## Meaning

実際とは別判断をした場合の結果を比較する研究補助Object。

## Owner

Post-Trade Analysis / Experimental Framework

## Main Fields

```yaml
counterfactual_result_id:
trade_or_trial_ref:
actual_action:
alternative_action:
model_assumptions:
simulated_outcome:
uncertainty:
limitations: []
```

---

# OBJ-POST-008: ResearchRoute

## Meaning

Post-Trade / Failure / AnomalyをどのResearch領域へ戻すかを記録するRouting Object。

## Owner

Research Router

## Main Fields

```yaml
research_route_id:
origin_object_refs: []
route_type:
target_research_domain:
priority:
reason:
created_research_candidate_ref:
created_at:
```

---

# J. PLATFORM / OPERATIONS OBJECTS

# OBJ-OPS-001: SystemStatus

## Meaning

Runtime / Market Instance /主要Subsystemの現在状態を表すOperational Object。

## Owner

Runtime / Monitoring

## Main Fields

```yaml
system_status_id:
instance_id:
as_of:
runtime_state:
market_profile_id:
subsystem_states:
active_incident_refs: []
resource_snapshot_ref:
health_summary:
```

---

# OBJ-OPS-002: ErrorEvent

## Meaning

Runtime / Data / Research / Production / Adapter等で発生した処理エラーを標準化するOperational Object。

## Owner

各Role / Failure Governance

## Main Fields

```yaml
error_event_id:
error_code:
category:
severity:
origin_role:
related_object_refs: []
occurred_at:
retryable:
retry_count:
impact_scope:
containment_state:
recovery_state:
stack_or_diagnostic_ref:
```

---

# OBJ-OPS-003: Incident

## Meaning

単発Errorではなく、複数Errorや異常をまとめて扱う障害単位Object。

## Owner

Failure Governance / Monitoring

## Main Fields

```yaml
incident_id:
severity:
started_at:
resolved_at:
error_event_refs: []
affected_roles: []
affected_object_refs: []
affected_active_position_refs: []
containment_action_refs: []
root_cause_state:
recovery_action_refs: []
postmortem_ref:
```

---

# OBJ-OPS-004: ResourceSnapshot

## Meaning

RAM / CPU / Disk / Queue / Cache / DB等のResource利用状況を記録するObject。

## Owner

Monitoring / Resource Governance

## Main Fields

```yaml
resource_snapshot_id:
as_of:
cpu_usage:
memory_usage:
disk_usage:
db_size:
queue_sizes:
cache_sizes:
process_count:
research_resource_usage:
threshold_state:
```

---

# OBJ-OPS-005: RecoveryAction

## Meaning

Error / Incidentから安全に復旧するために実施した操作Object。

## Owner

Recovery / Runtime

## Main Fields

```yaml
recovery_action_id:
incident_ref:
action_type:
target:
started_at:
finished_at:
result:
retry_policy_ref:
precondition:
postcondition:
operator_or_automation:
```

---

# OBJ-OPS-006: MigrationRecord

## Meaning

Schema / Object / Storage / Code移行の履歴を残す長期互換性Object。

## Owner

Version / Migration Governance

## Main Fields

```yaml
migration_record_id:
migration_type:
from_version:
to_version:
affected_object_types: []
started_at:
finished_at:
status:
backup_ref:
validation_result:
rollback_plan_ref:
```

---

# OBJ-OPS-007: BackupRecord

## Meaning

重要Knowledge / DB / Config等のBackup取得と復元可能性を記録するObject。

## Owner

Backup / Disaster Recovery Governance

## Main Fields

```yaml
backup_record_id:
backup_scope:
created_at:
storage_location_class:
content_hash:
encryption_state:
immutable_state:
restore_tested_at:
restore_test_result:
retention_class:
```

---

# OBJ-OPS-008: AuditEvent

## Meaning

重要な設定変更・Risk変更・Production操作・Governance判断を追跡する監査Object。

## Owner

Governance / Runtime / Security

## Main Fields

```yaml
audit_event_id:
action:
actor:
target_object_refs: []
before_ref:
after_ref:
reason:
created_at:
authorization_ref:
```

---

# 7. Evidence Source Channel共通定義

Research / Post-Trade / Knowledgeでは最低限次を区別する。

```text
RANDOM
HISTORICAL
REPLAY
OOS
REGIME
PAPER
DEMO_FORWARD
SHADOW
COUNTERFACTUAL
STRESS
LIVE
```

---

# 8. SnapshotとCurrent Stateの区別

同じ概念でも、現在値と過去Snapshot / 履歴を区別する。

FIX-012ではさらに、同一Knowledgeの現在状態を一つの`status`へ圧縮しない。

例:

```text
CausalHypothesis Lifecycle = APPROVED
Knowledge Aging / Health = AGING
Production Promotion = LIMITED_LIVE
Risk State = CAUTION
```

各軸は別参照・別StateTransitionEventを持つ。

---

# 9. Object参照の原則

同じ情報を複数Objectへ全文コピーするより、ID参照を基本とする。

State軸もLifecycle / Knowledge Aging / Production Promotion / Riskをコピー統合せず、正式Owner Object / Projectionへの参照を利用する。

---

# 10. Provenance Chain

主要判断ObjectはRaw / Evidenceまで逆引き可能にする。

FIX-012以降、Production判断では少なくとも次も追跡可能にする。

```text
HypothesisPoolEntry
├→ target lifecycle state
├→ KnowledgeLifecycleProfile
├→ Production Promotion Stage
└→ Applicability / Constraint

Defense / EntryThesis
└→ RiskState
```

---

# 11. Forward Impact Chain

Knowledge Aging / Health低下が発生した場合、研究成熟度を無言で書き換えるのではなく:

```text
Knowledge Aging CURRENT → DEGRADED
↓
Production Promotion再評価
↓
必要なら LIMITED_LIVE → PAUSED
↓
ResearchCandidate / Revalidation
```

のように別Transitionとして追跡する。

---

# 12. Objectを勝手に統合しない組み合わせ

```text
Observation ≠ MarketEvent
Feature ≠ MarketEvent
MarketEvent ≠ MarketContext
MarketEvent ≠ CauseCandidate
Feature ≠ Evidence
MarketContext ≠ MarketDNA
MarketDNA ≠ FeaturePriorityProfile
CauseCandidate ≠ CausalHypothesis
CausalHypothesis ≠ Edge
ResearchPlan Lifecycle State ≠ ResearchPlan Lock State
ResearchPlan Version ≠ ResearchPlan Lifecycle State ≠ ResearchPlan Lock State
ResearchResult ≠ EvidencePackage
HypothesisPoolEntry ≠ ApplicableHypothesisSet
ApplicableHypothesisSet ≠ TradeThesis
TradeThesis ≠ SignalDecision
SignalDecision ≠ DefenseDecision
TradeThesis ≠ EntryThesis
EntryThesis ≠ OrderIntent
OrderIntent ≠ ExecutionRecord
ExecutionRecord ≠ ProductionEvidence
ProductionEvidence ≠ Post-Trade Analysis Result
Current State ≠ StateTransitionEvent
StateTransitionEvent ≠ AuditEvent
Hypothesis / Edge Lifecycle ≠ Knowledge Aging / Health
Knowledge Aging / Health ≠ Production Promotion Stage
Production Promotion Stage ≠ Risk State
PositionThesisState ≠ ExitDecision
TradeResult ≠ TradeThesisEvaluation
ErrorEvent ≠ Failure Knowledge
```

FIX-012最重要原則:

```text
Research maturity
≠ Knowledge freshness / health
≠ Production permission
≠ Current OS risk permission
```

---

# 13. ObjectをTop-Level Layerへ昇格させない原則

State軸の分離を理由に新しいTop-Level Layerを増やさない。

`KnowledgeLifecycleProfile`、`HypothesisPoolEntry`、`RiskState`は独立Semantic Object / Projectionだが、State軸そのものを巨大Layerへしない。

---

# 14. Object追加Gate

FIX-012では新Objectを増やさず、既存Objectの責任分離で解決する。

---

# 15. Object変更ルール

Objectの意味・必須Field・Lifecycle・Ownerを変更する場合、`DESIGN_CHANGE_RULES.md` に従う。

FIX-012のField rename / State migrationは旧履歴を破壊せず、Migration Mappingを保持する。

---

# 16. Object LifecycleとSTATE_DICTIONARYの関係

Stateの正式語彙・遷移は:

```text
01_DICTIONARY/STATE_DICTIONARY.md
```

をSingle Source of Truth候補とする。

FIX-012では:

```text
STATE-HYP-001 = Hypothesis research maturity
STATE-HYP-002 = Edge research maturity
STATE-KNW-001 = Knowledge Aging / Health
STATE-PRD-001 = Production Promotion Stage
STATE-RISK-001 = Current OS Risk State
```

を別軸として扱う。

---

# 17. ObjectとDATA_CONTRACTの関係

後続 `DATA_CONTRACT.md` ではFIX-012について次を固定する。

```text
CausalHypothesis lifecycle state required / nullable
Edge lifecycle state required / nullable
KnowledgeLifecycleProfile cardinality
HypothesisPoolEntry target_knowledge_ref type
Lifecycle / Knowledge Aging / Production Promotion参照整合性
Production Thesis Builder Gate順序
AGING / STALE / DEGRADED時のProduction制限Policy
Production PAUSEDの復帰条件
RiskStateとの最終Gate arbitration
StateTransitionEvent namespace / atomicity
旧Field migration
```

---

# 18. ObjectとDATABASE_SCHEMAの関係

FIX-012では次を一つの`status` Columnへ圧縮しない。

```text
Hypothesis / Edge Lifecycle
Knowledge Aging / Health
Production Promotion Stage
Risk State
```

物理Table構造は `DATABASE_SCHEMA.md` で確定する。

---

# 19. ObjectとPython Modelの関係

新規Python実装で次のような万能条件を使わない。

```python
if hypothesis.status == "ACTIVE":
    trade()
```

概念的には:

```text
Lifecycle
AND Knowledge Aging / Health
AND Production Promotion Stage
AND Applicability
AND Constraint
AND Risk State
```

を別々に評価する。

---

# 20. Storage Lifecycle区分

重要Hypothesis / Edge / KnowledgeLifecycleProfile / HypothesisPoolEntry / StateTransitionEventは過去判断再現のため長期保存候補。

旧FIX-012以前StateもMigration前の意味を保持して保存する。

---

# 21. Knowledge Aging共通原則

FIX-012以降、Knowledge Aging / Healthの正本候補は`KnowledgeLifecycleProfile.knowledge_aging_state`。

```text
FRESH
CURRENT
AGING
STALE
DEGRADED
ARCHIVED
```

古い = 無効ではない。

劣化 = 研究Approvalを自動取消、でもない。

Production制限が必要なら別のProduction Promotion Transitionとして実施する。

---

# 22. Object Definition of Done

重要Objectでは、Lifecycle / Aging / Production / Riskのどの軸を持つかを明示し、他軸の責任をコピーしないことを確認する。

---

# 23. 現時点のObject一覧

## Common / Control

```text
MarketProfile
SourceMetadata
QualityProfile
Diagnostics
RuntimeCommand
GlobalRiskLimit
```

## State / Transition

```text
StateTransitionEvent
```

## Observation / Data

```text
RawData
Observation
MarketEvent
TimeSeriesMeasurement
```

## Measurement / Feature

```text
FormulaDefinition
MeasurementResult
Feature
FeaturePriorityProfile
```

## Market Understanding

```text
MarketContext
Evidence
Contradiction
UnexplainedEvent
```

## Causal / DNA

```text
CauseCandidate
EffectDefinition
CausalHypothesis
HypothesisAssessmentProfile
MarketDNA
RegimeProfile
```

## Research

```text
ResearchCandidate
ResearchPlan
ResearchTrial
ResearchResult
EvidencePackage
ApplicabilityProfile
Edge
ResearchBudget
```

## Knowledge

```text
MarketCase
FeatureKnowledge
FormulaKnowledge
Failure
StressResult
FailureBoundary
Constraint
NegativeKnowledge
KnowledgeRelationship
KnowledgeLifecycleProfile
```

## Production

```text
HypothesisPoolEntry
ApplicableHypothesisSet
TradeThesis
AIReviewResult
SignalDecision
DefenseDecision
RiskState
OrderIntent
ExecutionRecord
EntryThesis
PositionThesisState
ExitDecision
ProductionEvidence
TradeResult
```

## Post-Trade

```text
OutcomeAnalysisResult
TradeThesisEvaluation
HypothesisAttribution
DefenseEvaluation
SupervisorEvaluation
DemoLiveDivergence
CounterfactualResult
ResearchRoute
```

## Platform / Operations

```text
SystemStatus
ErrorEvent
Incident
ResourceSnapshot
RecoveryAction
MigrationRecord
BackupRecord
AuditEvent
```

---

# 24. 現時点で意図的にObject化しないもの

```text
Case Library
Market Memory
Failure Museum
Knowledge Graph
THESIS_NOT_BUILDABLE
ENTRY_SNAPSHOT_NOT_BUILDABLE
StateTransitionAttempt
HypothesisStateHistory
RiskStateHistory
RuntimeStateHistory
ResearchPlanLifecycleObject
ResearchPlanLockObject
EdgeHealthObject
```

FIX-012では`EdgeHealthObject`のような新Objectを作らず、Edge maturityはEdge Lifecycle、現在のKnowledge healthはKnowledgeLifecycleProfileで表す。

---

# 25. 未確定・次文書へ送る論点

1. 正式State一覧 → `STATE_DICTIONARY.md`
2. Market DNAとRegimeの正式依存関係
3. Feature Priority Cycle Contract
4. MarketEvent Detection Contract
5. ApplicableHypothesisSet / TradeThesis Contract
6. EntryThesis / ProductionEvidence Contract
7. StateTransitionEvent Atomicity / Authority
8. ResearchPlan Lifecycle / Lock Contract
9. **FIX-012のLifecycle / Knowledge Aging / Production Promotion / Risk State Gate順序・Cardinality・Migration・Policyは `DATA_CONTRACT.md` / Production Contract / Authority Matrixで固定する**
10. Soft / Hard ContradictionのProduction Gate
11. Risk State閾値
12. Retention / Encryption / Schema Migration

---

# 26. 最終原則

市場理解OSでは:

> **Roleは仕事をする。**
>
> **Objectは意味を運ぶ。**
>
> **Contractは受け渡しを守る。**
>
> **StateはLifecycleを表す。**
>
> **Knowledgeは長期に残る。**

FIX-012ではさらに:

> **研究成熟度・Knowledgeの鮮度/健全性・Production利用許可・現在OSのRisk許可を一つのstatusへ統合しない。**

```text
Hypothesis / Edge Lifecycle
≠ Knowledge Aging / Health
≠ Production Promotion Stage
≠ Risk State
```

Python・DB・Exchange・AI Providerが変わっても、この意味境界を失わない設計を優先する。
