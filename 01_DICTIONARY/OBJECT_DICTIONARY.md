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
object_id:              # Objectを一意に識別
object_type:            # RawData / Evidence / TradeThesis 等
schema_version:         # Data SchemaのVersion
object_version:         # 同一論理ObjectのVersion
created_at:             # 生成日時 UTC基準
updated_at:             # 変更可能Objectのみ
valid_from:             # 有効開始時刻がある場合
valid_until:            # 有効期限がある場合
status:                 # Lifecycleを持つ場合
market_profile_id:      # BTCUSDT等の対象Profile
trace_id:               # OS内部処理系列
parent_object_ids: []   # 直接生成元Object
source_refs: []         # 外部Source / Raw参照
provenance:             # 生成経路
quality_ref:            # Data Quality参照
uncertainty:            # 不確実性
created_by_role:        # 生成Role
code_version:           # 再現性が必要な場合
config_version:         # 再現性が必要な場合
```

全Objectへ無条件ですべてを複製するのではなく、Objectの性質に応じて必須 / 任意をContractで定義する。

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

原則:

- Legacy名を理由に新しいTable / Class / Objectを増やさない
- 過去資料を読む時は上記Mappingで解釈する
- Historical / OOS / Replay / Demo / Stress等の研究経路は、原則 `ResearchResult` の `evidence_source_channel` / `experiment_mode` / `trial_id` で区別する
- Stress固有のFailure Boundary研究では、正式な `StressResult` を追加の専門Objectとして持つことができる。これは一般的なResearch結果名の乱立とは区別する

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

例:

```text
短期Cache
一時Queue Item
Transient Health Probe
```

ただしFailure分析に必要なものはAudit / Diagnosticsへ保存する。

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

`StateTransitionEvent` はCurrent Stateそのものではなく、Stateが変化したという履歴上の事実を表す。

## Generator

当該State Machineで正式にTransitionを適用したAuthorized State-owning Role / Authority。

FIX-010では最終Authorityの割当そのものは確定せず、`requested_by_role / authorized_by_role / applied_by_role`を保存可能にする。最終Authority MatrixはFIX-013 /後続Contractで固定する。

## Custodian

Logger / Audit Storage。

LoggerはState変更を決定せず、生成済みStateTransitionEventを改変せず保存する。

## Consumers

- Current State Projection / State Machine
- Governance / Audit
- Monitoring
- Research / Post-Trade
- Recovery / Migration
- Human Interface

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

## FIX-010 Canonical Boundary

```text
Trigger / Request
↓
Transition Rule Check
↓
Authority Check
↓
Current State / expected_previous_state Check
↓
Transition Apply
↓
StateTransitionEvent
↓
Current State Projection Update
↓
Logger / Audit / Research / Monitoring
```

## Current State Relationship

```text
Current State
= 現在値を高速参照するためのProjection / Cache

StateTransitionEvent
= 実際に適用された過去遷移のImmutable Historical Fact
```

Transition Historyを`transition_sequence`順にReplayすることで、対象State MachineのCurrent Stateを再構築可能な設計を目標とする。

## Invariants

- 実際に成功・適用されたTransitionだけを`StateTransitionEvent`として生成する
- Authority Check / Rule Check等で拒否されたTransition Attemptを成功Eventとして記録しない
- `from_state` は適用直前のCurrent Stateと整合しなければならない
- `expected_previous_state` を用いて古いStateを前提としたConcurrent / Stale Writeを検出可能にする
- `to_state` は当該`state_machine_type / state_machine_version`で合法なStateでなければならない
- `transition_sequence` は同一State Machine / Target内で順序を一意に追跡可能にする
- `state_machine_version`を保持し、将来State定義が変化しても過去遷移を当時の意味で再現可能にする
- `trigger_refs` と `reason_codes` を分け、何が起点だったかと、なぜ遷移したかを追跡可能にする
- Authority provenanceとして`requested_by_role / authorized_by_role / applied_by_role`を必要に応じて保持する
- Manual Overrideを通常自動遷移に偽装せず、`manual_override_ref / authorization_ref`から追跡可能にする
- 生成後に過去Eventを削除・上書きしない
- 判断訂正が必要な場合は元Eventを書き換えず、新しい合法TransitionまたはCorrection / Superseded関係を使う
- Current Stateの高速Projectionが破損しても、正当なTransition Historyから再構築可能な設計を維持する

## Failed / Rejected Transition

Transition Requestが拒否・失敗した場合、それは`StateTransitionEvent`ではない。

```text
Rejected / Failed Transition Attempt
→ AuditEvent / Diagnostics / ApprovalDecision候補
```

必要性が将来確認された場合のみ、Object追加Gateを通して`StateTransitionAttempt`等を検討する。FIX-010では新Objectを増やさない。

## StateTransitionEvent vs AuditEvent

```text
StateTransitionEvent
= Stateが実際に変わった事実

AuditEvent
= 誰が何を操作・承認・変更要求したかという監査記録
```

同じState変更に両方が存在してよいが、同一Objectへ統合しない。

## Long-Term Notes

Hypothesis / Edge / Research / Risk / Runtime等ごとに個別の`HypothesisStateHistory` / `RiskStateHistory` / `RuntimeStateHistory` Objectを乱立させず、共通`StateTransitionEvent`を`state_machine_type`で区別する。

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

## Retention

長期保存方針はStorage Lifecycleで定義する。`Rawを残す原則` と `Storageは有限` をRetention / Compression / Archiveで両立させる。

---

# OBJ-DATA-002: Observation

## Meaning

Raw DataをNormalizerが標準形式へ変換し、研究・比較可能な単一観測値として表すObject。

## Owner

Normalizer

## Example

```text
open_interest = 12.4B at T
```

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

## Invariants

- Observationは正規化済みの観測値であり、市場現象の解釈ではない
- RawData参照を必ず保持し、一次証拠まで逆引き可能にする
- Timestamp / Unit / Symbol等の変換履歴を失わない
- Normalization処理でMarket判断・Cause判断を混ぜない

---

# OBJ-DATA-003: MarketEvent

## Meaning

Observation / TimeSeriesMeasurement / Featureから検出された、「市場で何かが発生した」という事実寄りの現象を一意に識別するImmutable Event Object。市場解釈・予測・因果主張とは分離する。

## Owner / Generator

`ROLE-EVT-001: Event Detection Processor`

Market IntelligenceはMarketEventのConsumer / Interpreterであり、Canonical MarketEventの直接Generatorではない。

## Example

```text
OI_SHOCK
ETF_FLOW_SHOCK
LIQUIDITY_COLLAPSE
FUNDING_RATE_JUMP
LIQUIDATION_SHOCK
VOLATILITY_EXPANSION
SPREAD_WIDENING
```

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

## FIX-007 Generation Boundary

```text
Observation / TimeSeriesMeasurement / Feature
↓
Event Detection Processor
↓
MarketEvent
├→ Feature Priority
├→ Market Intelligence
├→ Causal Engine
└→ Research
```

Meaning boundary:

```text
Observation = 何を観測したか
Feature = どう測ったか
MarketEvent = 何が発生したか
MarketContext = 今どんな市場か
CauseCandidate = なぜ発生した可能性があるか
```

## Invariants

- MarketEventは事実寄りのEventであり、BUY / SELL・価格予測・因果確定を含めない
- `BTC_WILL_CRASH` 等の予測をEvent Typeとして使わない
- `WHALES_ARE_MANIPULATING_MARKET` 等の因果解釈をEvent Typeとして使わない
- Market Intelligence / Causal Engineが事後的にCanonical MarketEventを直接追加しない
- Trace IDとMarket Event IDを分離する
- 同じ実市場Eventを複数Source / Observation / Trialで別Eventとして水増ししない
- 複数Providerが同じ現象を観測した場合でも、必要に応じて一つのCanonical MarketEventへ統合し、Source / Observation参照は失わない
- Event Detection Rule変更時は`detection_rule_version`を残し、過去Eventの意味を新Ruleで無言に書き換えない
- `NO_EVENT_DETECTED` と `EVENT_DETECTION_UNAVAILABLE / DEGRADED` を区別する。Detector異常時の空Event一覧を「市場Eventなし」とみなさない
- Data Quality低下時は`quality_ref / confidence / detector_health_at_detection`から下流が不確実性を判断できるようにする
- Event時刻はCausal EngineのTemporal Order / Lag研究へ利用可能な形で保持する
- MarketEvent生成後に意味を上書きするのではなく、訂正が必要ならCorrection / Superseded関係を用いる

## Failure / Research Feedback

Market Intelligenceが重要変化を認識したのに対応するCanonical MarketEventが存在しない場合、MIが直接Eventを作るのではなく:

```text
UnexplainedEvent / ResearchCandidate
→ Event Detection Research
```

へ戻す。

Detection Miss / False Positive / Duplicate Split / Duplicate Merge Error / Rule StalenessはEvent Detection自体の研究対象にできる。

---

# OBJ-DATA-004: NormalizedObservation [RETIRED / MERGED]

## Status

RETIRED / MERGED INTO `OBJ-DATA-002: Observation`

## Reason

旧定義では `Observation` と `NormalizedObservation` が意味・Owner・生成位置で重複していたため、FIX-001で一本化した。

Canonical Flow:

```text
Collector
→ RawData
→ Normalizer
→ Observation
→ Data Quality
→ Time Series Processor
```

## Migration Rule

旧実装・旧資料で `NormalizedObservation` を参照している場合、新設計では `Observation` へMigrationする。

旧ID `OBJ-DATA-004` は履歴追跡のため再利用しない。

## Prohibitions

- 新規コード・新規DB Schemaで `NormalizedObservation` を新しい独立Objectとして作らない
- `Observation` と二重保存しない

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

## Examples

```text
Delta
Return
Velocity
Acceleration
Persistence
Percentile
Volatility
Lagged Value
```

## Invariants

異なる時間窓を暗黙に混ぜない。

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

## Invariants

- Formulaを上書きして過去再現性を失わない
- 数学的正しさと市場的妥当性を分ける
- Production利用FormulaはResearch履歴を参照可能にする

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

## Invariants

FormulaDefinitionと結果を分ける。

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

## Invariants

- BUY / SELL意味をFeature自体へ埋め込まない
- Raw DataまでTrace可能にする
- Formula Versionを必須参照とする

---

# OBJ-FEAT-002: FeaturePriorityProfile

## Meaning

現在Evaluation Cycleで、各Featureをどの程度注目する価値があるかを表す優先度Object。FIX-006により、同一Cycleで後段生成されるMarketDNAを参照せず、原則としてPrevious Confirmed MarketDNAを利用する。

## Owner

Feature Priority

## Main Fields

```yaml
priority_profile_id:
evaluation_cycle_id:
market_profile_id:
current_context_ref:
current_event_refs: []
previous_confirmed_dna_ref:      # 原則 DNA_(t-1)。Cold Startではnull可
previous_confirmed_dna_as_of:
dna_reference_mode:              # PREVIOUS_CONFIRMED | COLD_START_BASELINE
horizon:
selected_features: []
low_priority_features: []
sentinel_features: []
priority_reasons: []
redundancy_map:
quality_constraints:
recompute_reason:                # NORMAL_CYCLE | MAJOR_EVENT_NEW_CYCLE 等
priority_version:
created_at:
valid_until:
```

## FIX-006 Cycle Invariants

- Priority = Confidenceではない
- Low Priority = Raw取得停止ではない
- Safety Sentinel FeatureはPriority低下だけで観測停止しない
- `previous_confirmed_dna_ref` は当該`evaluation_cycle_id`より前に確定済みのMarketDNAだけを参照する
- **同一Evaluation Cycleで後段生成される`MarketDNA_t`を参照してはならない**
- FeaturePriorityProfile生成後に、同CycleのMarketDNAを使ってPriority理由・選択Featureを事後改変しない
- Previous Confirmed MarketDNAが存在しないCold Startでは、`dna_reference_mode = COLD_START_BASELINE` とし、Baseline Feature / Sentinel Feature / Current Basic Context中心で生成する
- Major Regime / Structural Eventで再計算する場合は、同一Cycleを再帰更新せず新しい`evaluation_cycle_id`を発行する
- `previous_confirmed_dna_ref` / `previous_confirmed_dna_as_of` / `priority_version` を保存し、後から当時のPriority判断を再現できるようにする

## Canonical Cycle

```text
DNA_(t-1)
+
Current Basic Context / already-available MarketEvent information
↓
FeaturePriority_t
↓
MarketIntelligence_t
↓
Causal_t
↓
MarketDNA_t
↓
次Evaluation Cycle
```

禁止:

```text
FeaturePriority_t
→ MarketDNA_t
→ FeaturePriority_t
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

## Invariants

Cause確定・BUY / SELLを含めない。

MarketContextはMarketEventを解釈・文脈化できるが、MarketEventのCanonical生成責任を持たない。

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

## Invariants

- Evidence Source Channelを保持する
- 同じ証拠を複数Hypothesisで共有する場合 `shared_evidence` として追跡する
- Evidenceの数をそのまま強さとみなさない

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

## Invariants

Contradictionを削除して都合の良いEvidenceだけ残さない。

`SOFT / HARD` のProduction Gateへの正式影響は後続Contractで確定する。

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

## Invariants

説明不能時に物語を捏造しない。

Market Intelligenceが重要変化を認識したのに対応するCanonical MarketEventが無い場合、UnexplainedEvent / ResearchCandidate経由でEvent Detection Researchへ戻し、Canonical MarketEventを直接捏造しない。

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

## Invariants

- Cause CandidateはCauseではない
- 相関だけで原因確定しない
- Effectより後の事象を無条件でCause化しない

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

## Invariants

Effect定義を後から都合よく変更して過去Trialを再解釈しない。変更時はVersionを分ける。

---

# OBJ-CAUSAL-003: CausalHypothesis

## Meaning

Cause Candidate / Effect / Mechanism / 反証条件を統合した検証可能な仮説Object。

## Owner

Causal Engine / Knowledge Domain

## Main Fields

```yaml
hypothesis_id:
hypothesis_version:
status:
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

## Candidate Lifecycle

```text
DRAFT
→ TESTING / RESEARCHING
→ SUPPORTED
→ APPROVED
→ ACTIVE
→ AGING / WEAK / SUSPENDED
→ RETIRED
↘ REOPENED
```

最終State集合は `STATE_DICTIONARY.md` で確定する。

## Invariants

- 1回当たっただけでSUPPORTEDにしない
- 1回外れただけでRetireしない
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

## Invariants

単一Composite ScoreだけでLifecycleやTrade可否を決定しない。

表示用Scoreを将来持つ場合でも、元Dimensionを保存する。

---

# OBJ-DNA-001: MarketDNA

## Meaning

現在または過去の市場状態を比較・検索・研究できる正規化された圧縮表現。FIX-006では各MarketDNAを特定Evaluation Cycleの後段生成物として扱い、同CycleのFeature Priorityへ逆流させない。

## Owner

Market DNA Role / Knowledge Domain

## Candidate Axes

```yaml
trend:
volatility:
liquidity:
leverage:
derivatives:
participant_flow:
whale:
etf_institution:
macro_linkage:
time_session:
news_event_context:
cross_market_correlation:
```

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

## Invariants

- AIの説明不能なMystery Scoreにしない
- 各軸からRaw / Feature / Formulaまで遡れる
- Market DNA = 市場状態
- Feature Priority = 何を見る価値が高いか
- Signalを直接生成しない
- `evaluation_cycle_id = t` で生成されたMarketDNAは、同じ`t`のFeaturePriorityProfile入力に使用しない
- 当該MarketDNAは確定後、次Evaluation Cycle以降で`previous_confirmed_dna_ref`として参照可能になる
- `previous_market_dna_ref`を保持する場合も、Current DNAとの世代関係を示すためであり、同Cycle再帰計算の許可を意味しない

---

# OBJ-DNA-002: RegimeProfile

## Meaning

Market DNA / 市場状態を、ProductionやResearchで扱いやすい粗い分類へ写像する派生Object候補。

## Owner

Market DNA / Research

## Example

```text
BULL
BEAR
RANGE
HIGH_VOL
LOW_LIQUIDITY
```

## Main Fields

```yaml
regime_profile_id:
market_dna_ref:
regime_labels: []
classification_rule_version:
confidence:
valid_until:
```

## Invariants

独立した第二の市場状態体系としてMarket DNAと競合させない。

RegimeとDNAの正式関係はArchitecture Canonical化時に最終固定する。

---

# F. RESEARCH OBJECTS

# OBJ-RSCH-001: ResearchCandidate

## Meaning

Researchへ送るべき未検証課題・疑問・異常・仮説候補を統一して表す入口Object。

## Owner

Research Router / Research Orchestrator

## Candidate Sources

```text
Causal Hypothesis
Alternative Hypothesis
Contradiction
Feature Candidate
Formula Candidate
Market DNA Candidate
Unexplained Event
Novel Regime
Unexpected Failure
Unexpected Success
Missed Opportunity
Defense Block
Supervisor Warning
Execution Anomaly
Entry Snapshot Integrity / Timing Gap
Live Evidence Incomplete / Divergence
Data Quality Anomaly
Event Detection Miss / False Positive
Production Thesis Applicability / Composition Gap
Demo vs Live Divergence
AI Review New Idea
```

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

## Invariants

Research CandidateをそのままProductionへ利用しない。

---

# OBJ-RSCH-002: ResearchPlan

## Meaning

何を、どのEvidence Channel・Dataset・評価基準で検証するかを事前定義するVersioned研究計画Object。FIX-011ではResearchPlanの進行状況と編集可否を一つのstatusへ混ぜず、Lifecycle StateとLock Stateの2軸で独立管理する。

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

## FIX-011 State Separation

```text
ResearchPlan Identity / Version
        ├→ ResearchPlan Lifecycle
        │   DRAFT / READY / ACTIVE / COMPLETED / SUPERSEDED / CANCELLED
        │
        └→ ResearchPlan Lock State
            EDITABLE / PRE_REGISTERED / FROZEN
```

Meaning:

```text
Lifecycle State
= このPlan Versionの研究計画が今どの進行段階にあるか

Lock State
= このPlan Versionの評価規則・Data Scope・Metric・Stop Rule等をどの程度変更してよいか

plan_version
= 研究計画内容の世代
```

## StateTransitionEvent Relationship

LifecycleとLockは別State Machineとして扱う。

```text
ResearchPlan Lifecycle transition
→ StateTransitionEvent(state_machine_type = RESEARCH_PLAN_LIFECYCLE)

ResearchPlan Lock transition
→ StateTransitionEvent(state_machine_type = RESEARCH_PLAN_LOCK)
```

同一ResearchPlanに両方の履歴が存在できる。

例:

```text
Lifecycle = ACTIVE
Lock = FROZEN
```

は正常な組み合わせである。

## Version / Freeze Rule

`frozen_at`はLock Stateの正本ではなく、FROZENへ入った時刻を補助的に記録するTimestamp。

```text
lock_state = FROZEN
= 編集可否のCurrent State

frozen_at
= Freeze時刻Metadata
```

FROZEN後に研究条件を意味変更する必要がある場合、同じPlan Versionを無言で上書きしない。

```text
Plan v3 = FROZEN
↓ meaningful change required
Plan v4 = DRAFT / EDITABLE
↓
PRE_REGISTERED / FROZEN
↓
新しいTrial / 必要なら新しいT0
```

## Invariants

- Lifecycle StateとLock Stateを一つの`status`へ統合しない
- `ACTIVE + FROZEN` は合法であり、LifecycleとLockを相互排他的に扱わない
- `ACTIVE` だから研究条件を自由編集できるとは扱わない
- `FROZEN` だからLifecycleもFROZENとは扱わない
- `frozen_at`だけからLock Stateを推定しない
- FROZEN Planの評価基準・Data Scope・Metric・Stop Rule等を結果を見てから都合よく上書きしない
- FROZEN後のMeaningful Changeは新しい`plan_version`を作る
- Lifecycle TransitionとLock Transitionは別`StateTransitionEvent`として履歴化する
- `lifecycle_state_machine_version / lock_state_machine_version`を保持し、過去Planを当時のState定義で再現可能にする
- `SUPERSEDED`になった旧Plan / Versionを削除せず、後続Trialの再現に利用できるよう保持する
- Demo Forward / protected holdout等でどのLock Stateを開始条件に要求するかはResearch / Data / Processing Contractで正式化する

---

# OBJ-RSCH-003: ResearchTrial

## Meaning

特定ResearchPlan Versionに基づいて実行される一つの試行。FIX-011ではTrialが「どのPlan Version・どのLock状態を前提に開始されたか」を固定参照し、後から新しいPlan Versionで旧Trialを再解釈しない。

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

## FIX-011 Plan Binding

Trial開始時に使用したResearchPlanを、少なくとも概念上次で固定する。

```text
ResearchTrial
→ exact ResearchPlan reference
→ exact plan_version
→ Lifecycle State at Trial start
→ Lock State at Trial start
→ corresponding StateTransitionEvent references
```

Planが後で更新・SUPERSEDEDされても、既存TrialのPlan bindingを新Versionへ差し替えない。

Protected Research Modeでは後続Contractで例として:

```text
experiment_mode = DEMO_FORWARD
→ plan_lock_state_at_start must be FROZEN
```

等のHard Gateを定義可能にする。

## Invariants

- Demo ForwardではT0以降だけを評価する
- Rule変更時は同じTrialを上書きせず、新Plan Version / 新Trial / 必要なら新T0を作る
- Trial数とUnique Market Event数を分離する
- Trialが使用した`research_plan_version`を後から書き換えない
- 旧Trialを新しいResearchPlan Versionの結果として再解釈しない
- `plan_lock_state_at_start` はTrial開始時のSnapshotであり、後からPlanのCurrent Lock Stateが変わっても書き換えない
- LifecycleとLockのStateTransitionEvent参照を混同しない

---

# OBJ-RSCH-004: ResearchResult

## Meaning

ResearchTrialで観測された結果を、結論とDiagnosticsを分けて保存する正式な共通研究結果Object。

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

## Invariants

- 失敗結果も保存する
- Historical / OOS / Regime / Replay / Demo等の違いを理由に `HistoricalValidationResult` 等の別Result Objectを乱立させない
- 研究経路は `ResearchTrial.experiment_mode`、`evidence_source_channel`、Trial参照で区別する
- Stress固有の境界情報が必要な場合は、ResearchResultとTrace可能な `StressResult` / `FailureBoundary` を併用できる

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

## Invariants

Historical 4200 + Demo 420 + Live 28を単純4648件として扱わない。

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

## Invariants

「有効 / 無効」だけでなく条件付き適用性を保持する。

---

# OBJ-RSCH-007: Edge

## Meaning

Researchで再現可能性・期待値・適用条件が確認された利用可能な優位性Object。

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
source_hypothesis_refs: []
mechanism_or_pattern:
expected_effect:
expected_horizon:
expected_value_profile:
applicability_ref:
evidence_package_ref:
constraint_refs: []
status:
last_validated_at:
```

## Invariants

因果説明が強い = EVが正とは限らない。
Empirical EdgeもOOS / Demo等の再現性を要求する。

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

## Invariants

Researchの価値と計算資源を別々に無限化しない。

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

## Invariants

Case LibraryはこのObjectへのViewであり、別コピーを作らない。

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

## Invariants

単なるError Logで終わらせない。重要FailureはBoundary / Constraint / Recovery Knowledgeへ昇格可能にする。

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

## Invariants

Boundaryが未知ならUNKNOWNを明示する。

---

# OBJ-KNW-007: Constraint

## Meaning

Researchで得たFailureBoundary等をProductionの適用・禁止・修正条件へ変換したSafety Knowledge。

## Owner

Knowledge Promotion / Risk Governance

## Candidate Types

```text
REQUIRE
EXCLUDE
MODIFY
ALLOW
```

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

## Invariants

ConstraintはBUY / SELL方向を生成しない。

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

## Invariants

同じ無駄研究を繰り返さない。ただし構造変化時のReopen条件を持てる。

---

# OBJ-KNW-009: KnowledgeRelationship

## Meaning

Knowledge Object同士の関係を一元的に表すGraph Edge Object。

## Owner

Knowledge Domain

## Example Types

```text
SUPPORTS
CONTRADICTS
DEPENDS_ON
DERIVED_FROM
APPLIES_TO
FAILS_UNDER
SUPERSEDES
DUPLICATES
RELATED_TO
```

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

## Invariants

Knowledge GraphはRelationshipを表示するViewであり、別Knowledgeを複製しない。

---

# OBJ-KNW-010: KnowledgeLifecycleProfile

## Meaning

Knowledgeの年齢・劣化・再検証必要性を管理する長期運用Object。

## Owner

Knowledge Aging Governance

## Main Fields

```yaml
knowledge_lifecycle_id:
target_object_id:
created_at:
last_validated_at:
last_demo_pass_at:
last_live_evidence_at:
revalidation_due_at:
staleness_state:
edge_health_state:
current_risk_stage:
reopen_or_suspend_reason:
```

## Invariants

一度SUPPORTED / APPROVEDになったKnowledgeを永久真理として扱わない。

---

# H. PRODUCTION / TRADING OBJECTS

# OBJ-PRD-001: HypothesisPoolEntry

## Meaning

Productionが参照するHypothesis / Edgeの登録情報。Hypothesis自身の研究成熟度と、本番で許可された利用段階を混ぜずに保持するEntry Object。

## Owner

Knowledge Promotion / Production

## Main Fields

```yaml
pool_entry_id:
hypothesis_or_edge_ref:
hypothesis_state_ref:         # Hypothesis / Edge側のLifecycle State参照
production_stage:             # RESEARCH_ONLY / SHADOW / DEMO_FORWARD / MICRO_LIVE / LIMITED_LIVE / NORMAL_LIVE / PAUSED
applicability_ref:
constraint_refs: []
assessment_profile_ref:
evidence_package_ref:
max_production_stage:         # Policy上許可される最大Production Promotion Stage
expires_or_review_due_at:
```

## State Separation

```text
Hypothesis Lifecycle
DRAFT / RESEARCHING / SUPPORTED / WEAK / APPROVED / ACTIVE / ...

≠

Production Promotion Stage
RESEARCH_ONLY / SHADOW / DEMO_FORWARD / MICRO_LIVE / LIMITED_LIVE / NORMAL_LIVE / PAUSED
```

## Invariants

- `SUPPORTED` と `DEMO_FORWARD` を同じstatus列挙へ入れない
- `APPROVED` はKnowledge / Hypothesisの承認状態であり、現在市場で即取引可能という意味ではない
- Productionは `hypothesis_state_ref`、`production_stage`、Applicability、Constraint、Risk Stateを別々に確認する
- 未承認Hypothesis / EdgeをProductionが自由に有効化しない
- `max_production_stage` をRisk Stateと混同しない

---

# OBJ-PRD-002: ApplicableHypothesisSet

## Meaning

Approved / production-eligibleなHypothesis / Edgeのうち、現在のMarketContext / MarketDNA / Applicability / Constraint / Quality等へ実際に適用可能なものを役割付きで束ねたImmutable Production Object。

## Owner / Generator

`ROLE-PRD-001: Production Thesis Builder`

Knowledge PromotionはApproved Poolを供給するが、現在市場でのApplicable Setを直接生成しない。
Signal EngineはApplicableHypothesisSetを利用するが、Canonical Set生成責任を持たない。

## Roles

```text
PRIMARY
SUPPORTING
CONDITIONAL
CONTRADICTING
```

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

## FIX-008 Generation Boundary

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
Signal Engine
```

## Invariants

- `Approved != Applicable`。承認済みでも現在条件に適用できなければSetへ入れない
- DRAFT / RESEARCHING / 未承認HypothesisをLive用ApplicableHypothesisSetへ入れない
- Production Promotion Stage / `max_production_stage` を超えた利用を許可しない
- 仮説数の多数決でDirectionを決めない
- PRIMARY / SUPPORTING / CONDITIONAL / CONTRADICTINGの意味を保持する
- Shared Evidenceを複数Hypothesisの独立Evidenceとして二重評価しない
- Dependency / Redundancy / Common Causeを隠さない
- Constraint違反を単なる弱い減点として隠さない
- Knowledge Aging / Staleness / Revalidation requirementを無視しない
- Set生成時点のBuilder Versionと参照したMarketContext / MarketDNA / Applicability / Lifecycle / Production Stageを追跡可能にする
- 生成後に現在Knowledgeが変化しても過去Setを無言で書き換えない

---

# OBJ-PRD-003: TradeThesis

## Meaning

Production Thesis BuilderがApplicableHypothesisSetを現在市場で検討可能な一つの取引論拠へ構成したImmutable Object。TradeThesisは「何を期待しているか」を表すが、最終的な`BUY / SELL / NO_TRADE` Decisionそのものではない。

## Owner / Generator

`ROLE-PRD-001: Production Thesis Builder`

Signal EngineはTradeThesisを評価して`SignalDecision`を生成するConsumerであり、TradeThesisのCanonical Generatorではない。

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

## FIX-008 Meaning Boundary

```text
ApplicableHypothesisSet
= 今の市場で利用可能なApproved Hypothesis / Edgeを役割付きで束ねたもの

TradeThesis
= そのSetから、何を・なぜ・どのHorizonで期待するかを構成した論拠

SignalDecision
= そのTradeThesisに対して実際にBUY / SELL / NO_TRADEを判断したDecision
```

## Invariants

- `Approved != Applicable != Trade-worthy` を崩さない
- TradeThesisは複数仮説の平均・多数決ではない
- TradeThesisは必ずCanonical `ApplicableHypothesisSet` を参照する
- 新しいAI案・未検証Hypothesisをその場で追加しない
- Shared Evidenceを重複加点しない
- Dependency / Redundancy / Common Causeを消さない
- Contradicting Hypothesis / Contradictionを都合よく削除しない
- Constraint / Quality / Uncertaintyを保持する
- `expected_direction` は論拠上の期待方向であり、`SignalDecision.decision` と同一ではない
- Production Thesis BuilderはTradeThesisから直接注文を生成しない
- SignalとTrade Thesisを同一Objectにしない
- TradeThesisを構成できない場合、情報を捏造して空・弱いThesisを生成しない。`THESIS_NOT_BUILDABLE` はTradeThesis ObjectそのものではなくDiagnostics / Processing Contract上の非成立結果として扱う
- Builder Versionと入力参照を保持し、過去Tradeで当時のThesis構成を再現可能にする

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

## Invariants

AI ReviewはAdvisory。AI停止時も本番経路を成立可能にする。

AIが新しいHypothesisを提案しても、現在TradeThesisへ直接追加せずResearchCandidateへ送る。

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

## Invariants

- 万能総合点だけで決めない
- NO_TRADEも必要に応じて研究可能なDecisionとして保存する
- Signal EngineはApplicableHypothesisSet / TradeThesisのCanonical Generatorではない
- `THESIS_NOT_BUILDABLE` と `NO_TRADE` を混同しない。前者は有効Thesisが成立していない状態、後者は有効Thesisを評価したうえでRiskを取らないDecision

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

## Invariants

Signal = 取りたいか。
Defense = 今取ってよいか。
両者を統合しない。

---

# OBJ-PRD-007: RiskState

## Meaning

現在OSがどの程度Riskを許可するかを表す横断状態Object。

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
edge_health_state:
execution_health_state:
data_health_state:
market_novelty_state:
allowed_exposure:
allowed_trade_stage:
effective_from:
review_condition:
```

## Invariants

損失発生後だけでなく、Edge / Data / Execution劣化もRisk縮小理由になり得る。

数値閾値はResearch / Risk Designで別途定義する。

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

## Invariants

- Exchange固有payloadをCoreへ漏らさない
- `entry_thesis_ref` を持たないProduction OrderIntentを生成しない
- EntryThesis生成失敗時にOrderIntentを作らない
- OrderIntent生成後にEntryThesisを事後改変しない

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

## Invariants

- Strategy OutcomeとExecution Outcomeを分離可能にする
- ExecutionRecordは注文がどう送信・約定されたかの一次実行証拠であり、ProductionEvidenceそのものではない

---

# OBJ-PRD-010: EntryThesis

## Meaning

実際にOrderIntentを生成してRiskを取りに行く直前に、TradeThesis・Hypothesis Set・市場状態・Signal・Defense・Risk・Constraint・Versionを固定するImmutable Entry Snapshot。

## Owner / Generator

`ROLE-EXEC-001: Execution Logic / Entry Snapshot Builder`

## Custodian

`ROLE-LOG-001: Logger`

Loggerは生成済みEntryThesisを改変せず保存する。Canonical生成責任を持たない。

## Consumers

- Order Planning / Execution Logic
- Position Supervisor
- Logger
- Post-Trade Analysis
- Research

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

## FIX-009 Generation Boundary

```text
TradeThesis
↓
SignalDecision
↓
DefenseDecision
↓
RiskState / Current Market references確認
↓
Entry Snapshot Builder
↓
EntryThesis
↓
OrderIntent
```

## Invariants

- EntryThesisはTradeThesisそのものではなく、実際にRiskを取る直前の判断根拠Snapshot
- EntryThesis生成前にOrderIntentを作らない
- `SignalDecision` / `DefenseDecision` / `RiskState` / `TradeThesis` / `ApplicableHypothesisSet`へのTraceを失わない
- `entry_snapshot_at` と `snapshot_builder_version` を保持する
- Entry後にMarketDNA / Hypothesis State / Evidence / TradeThesisが更新されてもEntryThesisを書き換えない
- Snapshot生成時に不足した必須情報を推測して埋めない
- `ENTRY_SNAPSHOT_NOT_BUILDABLE` の場合はOrderIntent生成を禁止する
- Correctionが必要な場合も元Snapshotを無言で上書きせず、Correction / Superseded関係を使う

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

EMERGENCYはIn-Trade Defense側のSafety状態として分離する。

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

## Invariants

短期ノイズで状態が頻繁反転しないよう、Hysteresis / Persistence規則を後続設計で持つ。

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

## Invariants

Position Supervisorの助言と最終Exit判断を分離する。

---

# OBJ-PRD-013: ProductionEvidence

## Meaning

ExecutionRecordとLive市場文脈から、実市場でしか得られない約定品質・Slippage・Fee・Funding・Partial Fill・Liquidity Impact・Latency等を構造化したImmutable LIVE Evidence Object。

## Owner / Generator

`ROLE-EXEC-001: Execution Domain / Live Evidence Collector`

Exchange AdapterはExecutionRecordを生成するが、ProductionEvidenceのSemantic Generatorではない。

## Custodian

`ROLE-LOG-001: Logger`

## Analyzer / Consumer

`ROLE-ANL-001: Post-Trade Analysis` / Research / Knowledge Domain

Post-TradeはProductionEvidenceを分析するが、元Evidenceを生成・改変しない。

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

## FIX-009 Generation Boundary

```text
ExecutionRecord
+ EntryThesis / TradeThesis
+ Live Market / Fee / Funding / Liquidity context
↓
Live Evidence Collector
↓
ProductionEvidence
↓
Logger / Immutable Storage
↓
Post-Trade Analysis
```

## Invariants

- `ExecutionRecord != ProductionEvidence`
- `evidence_source_channel = LIVE` を保持し、Historical / Demo Evidenceへ無言で混ぜない
- LoggerはProductionEvidenceを保存するだけでSemantic生成しない
- Post-TradeはProductionEvidenceを分析するだけで元Evidenceを再生成・改変しない
- Live Evidenceの一部が取得不能でも取引事実を削除しない
- 不完全時は`evidence_completeness / quality_ref / diagnostics_ref`でDEGRADED / INCOMPLETE相当を表現し、推測値で補完しない
- Actual Slippage / Fee / Funding等とDemo / Historical推定値を別Channelとして比較可能にする
- `evidence_builder_version`を保存し、後から抽出Logicを再現可能にする

---

# OBJ-PRD-014: TradeResult

## Meaning

一つの実取引またはSimulation取引の最終Outcome Object。

## Owner

Logger / Post-Trade

## Main Fields

```yaml
trade_id:
trial_id:                  # Demo / Researchなら使用
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

## Invariants

Trade Result = Hypothesis Resultではない。

---

# I. POST-TRADE / FEEDBACK OBJECTS

# OBJ-POST-001: OutcomeAnalysisResult

## Meaning

Trade ResultをWIN / LOSSだけでなく、期待値・Horizon・MAE/MFE・Opportunity Cost等で分析したObject。

## Owner

Post-Trade Analysis

## Candidate Classification

```text
EXPECTED_SUCCESS
UNEXPECTED_SUCCESS
EXPECTED_FAILURE
UNEXPECTED_FAILURE
MISSED_OPPORTUNITY
SYSTEM_FAILURE
```

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

## Invariants

TradeがLossでもHypothesisを自動Retireしない。

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

## Invariants

Live Failureだけで即HypothesisをRetireせず、Execution / Market / Demo Model差を分解する。

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

## Invariants

反実仮想を実測Evidenceと同等に扱わない。

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

## Invariants

直接Trainerへ流さず、原因に応じたResearchへ戻す。

---

# J. PLATFORM / OPERATIONS OBJECTS

# OBJ-OPS-001: SystemStatus

## Meaning

Runtime / Market Instance /主要Subsystemの現在状態を表すOperational Object。

## Owner

Runtime / Monitoring

## Candidate Runtime States

```text
BOOTING
RUNNING
PAUSED
DEGRADED
ERROR
RECOVERY
STOPPING
STOPPED
```

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

## Invariants

同じErrorを無限Retryしない。Retry PolicyはFailure Governanceで定義する。

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

## Invariants

市場Data障害がどのActive Positionへ影響するかForward Impact Trace可能にする。

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

## Invariants

監視だけで終わらせず、Resource Budget / Retention / DEGRADED条件へ接続できる。

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

## Invariants

Restart無限Loopを許可しない。

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

## Invariants

古いKnowledgeを新Schemaへ移行した際、出所・元Versionを失わない。

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

## Invariants

Backupが存在するだけで安全とみなさず、Restore Testを追跡する。

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

## Invariants

重要変更を「誰が・なぜ」行ったか追跡可能にする。

`AuditEvent` はStateが実際に変更された事実そのものではない。成功State遷移の履歴は`StateTransitionEvent`を正本とする。

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

原則:

```text
Historical Evidence
≠ Demo Forward Evidence
≠ Live Production Evidence
```

EvidencePackageで束ねることはできるが、元Channelを消して統合しない。

ProductionEvidenceは必ず`LIVE` Channelの出所を保持する。

---

# 8. SnapshotとCurrent Stateの区別

同じ概念でも、現在値と過去Snapshot /履歴を区別する。

例:

```text
Current MarketDNA
= 現在参照している市場状態

MarketDNA Snapshot
= 過去Case / Trade時点で固定されたDNA
```

```text
Current Hypothesis State
= 現時点の高速参照用Projection

StateTransitionEvent
= Stateが実際にどう変わったかを保存するImmutable履歴

EntryThesis
= 実際にOrderIntentを作る直前に固定したTradeThesis / Hypothesis Set / Market / Risk / VersionのSnapshot
```

ApplicableHypothesisSet / TradeThesis / EntryThesisはそれぞれ生成時点の情報を固定し、後から現在Knowledgeが変わっても過去Production判断を無言で書き換えない。

StateについてもCurrent Stateだけを上書きしてTransition Historyを失わない。

---

# 9. Object参照の原則

同じ情報を複数Objectへ全文コピーするより、ID参照を基本とする。

例:

```text
OrderIntent
→ EntryThesis
→ TradeThesis
→ ApplicableHypothesisSet
→ hypothesis_id / edge_id
→ evidence_id
→ feature_id / market_event_id
→ raw_data_id
```

State MachineではCurrent Projectionから`latest_transition_event_ref`等を参照可能にし、StateTransitionEvent側は`target_object_ref / previous_transition_event_ref / trigger_refs`から履歴を追跡できるようにする。

ただし、EntryThesis等の監査・再現に重要なSnapshotでは、必要最小限のValueを同時固定してよい。

この境界はData Contract / DB Schemaで確定する。

---

# 10. Provenance Chain

主要判断Objectは最低限次の経路を逆引きできることを目標とする。

```text
TradeResult
↓
ProductionEvidence / ExecutionRecord
↓
OrderIntent
↓
EntryThesis
↓
DefenseDecision
↓
SignalDecision
↓
TradeThesis
↓
ApplicableHypothesisSet
↓
HypothesisPoolEntry / CausalHypothesis / Edge
↓
ApplicabilityProfile / Evidence / Constraint
↓
MarketContext / MarketDNA
↓
MarketEvent / Feature
↓
Event Detection Rule / FormulaDefinition / MeasurementResult
↓
Observation / TimeSeriesMeasurement
↓
RawData
↓
SourceMetadata
```

ProductionEvidence自体も`ExecutionRecord + EntryThesis + Live Context`まで逆引き可能にする。

Stateについては:

```text
Current State
↓
latest StateTransitionEvent
↓
previous StateTransitionEvent
↓
Trigger / Evidence / Decision / Error / Authority provenance
```

を追跡可能にする。

---

# 11. Forward Impact Chain

長期運用では逆引きだけでなく、上流障害が下流へ何を壊すか追跡する。

```text
Source Failure
↓
Affected Observation
↓
Affected Feature / Event Detection
↓
Affected MarketEvent / Evidence
↓
Affected Hypothesis / Applicability
↓
Affected ApplicableHypothesisSet
↓
Affected TradeThesis
↓
Affected EntryThesis candidate
↓
Affected Order / Active Position
```

Entry後はEntryThesisを書き換えず、影響はCurrent State / Supervisor / Defense / Incidentで扱う。

重要なCurrent State変更はStateTransitionEventに残し、後からForward Impactの因果順序を追跡可能にする。

---

# 12. Objectを勝手に統合しない組み合わせ

以下は意味が異なるため、安易に一Objectへ統合しない。

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
PositionThesisState ≠ ExitDecision
TradeResult ≠ TradeThesisEvaluation
TradeResult ≠ HypothesisAttribution
ErrorEvent ≠ Failure Knowledge
```

特にFIX-008では次を固定する。

```text
Approved Knowledge
≠ ApplicableHypothesisSet
≠ TradeThesis
≠ SignalDecision
```

FIX-009では次を固定する。

```text
Generator
≠ Custodian
≠ Analyzer
```

FIX-010では次を固定する。

```text
Current State
≠ StateTransitionEvent
≠ AuditEvent
```

FIX-011では次を固定する。

```text
ResearchPlan Version
≠ ResearchPlan Lifecycle State
≠ ResearchPlan Lock State
```

---

# 13. ObjectをTop-Level Layerへ昇格させない原則

以下は重要Objectだが、それだけを理由に独立巨大Layerへしない。

```text
MarketDNA
ApplicableHypothesisSet
TradeThesis
EntryThesis
ProductionEvidence
StateTransitionEvent
EvidencePackage
FailureBoundary
Constraint
ResearchCandidate
RiskState
```

Role Dictionaryに独立責任主体が定義されている場合のみRole / Module化を検討する。

ApplicableHypothesisSetとTradeThesisはFIX-008で`Production Thesis Builder`が生成するが、両Objectを別々のTop-Level Layerへ昇格させない。

EntryThesis / ProductionEvidenceの生成処理はFIX-009でExecution Domain内部Submoduleへ割り当てるが、Object自体を新Top-Level Layerへ昇格させない。

StateTransitionEventはFIX-010でOS横断Objectとして正式化するが、`State Transition Layer`のような新Top-Level Layerは作らない。

ResearchPlanのLifecycle / LockはFIX-011で二つのState Machineとして扱うが、各Stateを別Object / 別Top-Level Layerへ昇格させない。

---

# 14. Object追加Gate

新Objectを追加する前に次を確認する。

```text
1. 既存ObjectのField追加で表現できないか
2. 単なるViewではないか
3. 単なるStateではないか
4. 単なるContractではないか
5. 別Objectとして独立ID / Lifecycle / Versionが必要か
6. 監査・再現・研究上、独立保存する価値があるか
7. 既存Objectとの重複率は高くないか
8. 何十年保存する意味があるか
```

独立ID / Lifecycle / Version / Ownerが不要なら、新Objectを増やさない。

FIX-010のStateTransitionEventは、異なるState Machineを横断して独立ID・時系列・Version・Authority provenance・Replay価値を持つため、独立Persistent Objectとして正式化する。

FIX-011ではResearchPlan Lifecycle / Lockのために新しいState Objectを増やさず、既存ResearchPlan + StateTransitionEventで表現する。

---

# 15. Object変更ルール

Objectの意味・必須Field・Lifecycle・Ownerを変更する場合:

1. `DESIGN_CHANGE_RULES.md` に従う
2. 既存Objectとの互換性を確認する
3. Schema Migration要否を確認する
4. 過去Objectを読み込めるか確認する
5. Trace / Provenanceが切れないか確認する
6. Production Snapshot再現性を確認する
7. Migration / Rollback Planを作る

単なるPython class変更だけでSemantic Object定義を勝手に変えない。

---

# 16. Object LifecycleとSTATE_DICTIONARYの関係

本書はObjectの意味を定義する。

Stateの正式語彙・遷移は後続の:

```text
01_DICTIONARY/STATE_DICTIONARY.md
```

をSingle Source of Truth候補とする。

FIX-010以降、成功したState変更履歴の共通Semantic Objectは`OBJ-STATE-001: StateTransitionEvent`を使用する。

各State Machine専用の重複History Objectを無秩序に増やさない。

FIX-011ではResearchPlanが同時に二つのState Machineを持つことをObject側へ正式接続する。

```text
STATE-RSCH-002   = ResearchPlan Lifecycle
STATE-RSCH-002-L = ResearchPlan Lock State
```

両State MachineのCurrent State / Version / latest transition referenceを混同しない。

---

# 17. ObjectとDATA_CONTRACTの関係

本書:

```text
Object = 何の情報か
```

後続 `DATA_CONTRACT.md`:

```text
誰が
どのObjectを
どの条件で
どのVersionで
誰へ渡すか
```

を定義する。

Object Dictionary内へRole間通信規則を過剰に埋め込まない。

FIX-008で決めた`THESIS_NOT_BUILDABLE`の具体的なResult表現、Required / Nullable、Build失敗時の受け渡し、Builder Version型、Set / ThesisのCardinalityはData / Processing Contractで固定する。

FIX-009で決めた次もData / Processing Contractで固定する。

```text
EntryThesis required references / nullability
ENTRY_SNAPSHOT_NOT_BUILDABLEのResult表現
EntryThesis → OrderIntent hard gate
entry_snapshot_atとOrderIntent.created_at / ExecutionRecord.submitted_atの時系列制約
ProductionEvidence completeness / quality表現
ExecutionRecord → ProductionEvidence cardinality
LIVE Channel固定
Logger Custody Contract
Post-Trade Read-only Consumer Contract
```

FIX-010で決めた次もData / Processing Contractで固定する。

```text
StateTransitionEvent required / nullable fields
state_machine_type namespace / enum
transition_sequence uniqueness scope
expected_previous_state compare-and-set rule
successful applyとEvent persistのatomicity
Current Projection update ordering
latest_transition_event_ref
Replay / rebuild rule
StateTransitionEvent retention
AuditEventとの関連Cardinality
Manual Override authorization reference
```

FIX-011で決めた次もResearch / Data / Processing Contractで固定する。

```text
ResearchPlan lifecycle_state / lock_state required / nullable rule
Lifecycle / Lock state_machine_type namespace
Lifecycle / Lock transition atomicity and latest reference consistency
ResearchPlan Version identity / supersedes cardinality
FROZEN後のMeaningful Change判定
ResearchTrial → exact ResearchPlan Version binding
plan_lock_state_at_start / plan_lifecycle_state_at_start snapshot rule
DEMO_FORWARD / protected holdout開始時のFROZEN hard gate
Plan Version変更時のnew T0 requirement
旧Trialを新Plan Versionへ再帰属させない制約
```

---

# 18. ObjectとDATABASE_SCHEMAの関係

本書のObjectとDB Tableを1:1固定しない。

例:

```text
Logical Object
CausalHypothesis

Physical Storage
hypotheses table
hypothesis_versions table
evidence_links table
```

のように分割される可能性がある。

StateTransitionEventもSemantic Objectとしては1つだが、DBではCurrent Projection TableとTransition History Tableへ分かれる可能性がある。

ResearchPlanもLogical Objectとしては1つだが、Plan Version / Current Lifecycle / Current Lock / Transition Historyを物理的にどう保存するかはDB Schemaで決める。

DB Schemaは後からStorage効率・Query・Migrationを考えて決める。

重要:

> Semantic ObjectがDB都合で壊れないこと。

---

# 19. ObjectとPython Modelの関係

本書のObjectとPython classを1:1永久固定しない。

実装時には例として:

```text
Pydantic Model
Dataclass
ORM Model
TypedDict
Event Schema
```

等へ落とせる。

ただしコード側の命名と意味は本辞書へ従う。

---

# 20. Storage Lifecycle区分

Objectごとに将来Retention Classを付与する。

候補:

```text
HOT
WARM
COLD
ARCHIVE
EPHEMERAL
IMMUTABLE_LONG_TERM
```

例:

```text
Recent Observation
→ HOT / WARM

Old compressed Raw
→ COLD / ARCHIVE

Cache
→ EPHEMERAL

Important Hypothesis / Failure / Trade / Knowledge
→ IMMUTABLE_LONG_TERM候補
```

EntryThesis / ProductionEvidence / ExecutionRecord / TradeResult / StateTransitionEventは監査・研究・再構築価値が高いため長期Immutable保存候補。

FROZEN / SUPERSEDED ResearchPlan Versionと、それに紐づくResearchTrialも研究再現性のため長期保存候補。

具体的保存期間は `STORAGE.md` / Long-Term Governanceで決める。

---

# 21. Knowledge Aging共通原則

長期Knowledge Objectは最低限、可能なものについて:

```text
created_at
last_validated_at
revalidation_due_at
status
staleness
```

を追跡できるようにする。

古い = 無効ではない。

ただし:

> 長期間再検証されていない = 不確実性が増えている可能性

として扱う。

Production Thesis BuilderはKnowledge Agingを無視してApplicableHypothesisSetへ採用しない。

---

# 22. Object Definition of Done

一つの重要Objectを完全設計済みとみなす最低条件:

```text
□ Object ID / Name
□ Meaning
□ Category
□ Owner Role
□ Generator
□ Primary Inputs / Parent Objects
□ Main Fields
□ ID rule
□ Version rule
□ Lifecycle / State relation
□ Trace / Provenance
□ Quality / Uncertainty relation
□ Mutability
□ Retention class候補
□ Security / Privacy consideration
□ Invariants
□ Prohibitions
□ Downstream users
□ Migration consideration
□ Failure / invalid object handling
```

本辞書はSemantic Levelの完全設計候補であり、各Objectの型・nullability・enum・index等はData Contract / DB Schemaでさらに固定する。

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

`NormalizedObservation` はFIX-001で `Observation` へ統合済み。旧 `OBJ-DATA-004` は履歴用Tombstoneとしてのみ保持する。

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

以下は現段階では独立Persistent Objectとして確定しない。

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
```

理由:

- Case Library / Market Memory / Failure Museum / Knowledge Graph = Knowledge DomainのView
- `THESIS_NOT_BUILDABLE` = FIX-008時点ではTradeThesisとは別Objectを新設せず、BuilderのDiagnostics / Processing Contract上の非成立結果として扱う
- `ENTRY_SNAPSHOT_NOT_BUILDABLE` = FIX-009時点ではEntryThesisとは別Objectを新設せず、Entry Snapshot BuilderのDiagnostics / Processing Contract上のHard Gate非成立結果として扱う
- `StateTransitionAttempt` = FIX-010では拒否・失敗Attempt専用Objectを増やさずAuditEvent / Diagnostics等で扱う
- 個別State History名 = 共通`StateTransitionEvent`を`state_machine_type`で利用し、履歴Objectを乱立させない
- ResearchPlan Lifecycle / Lock = FIX-011では新Objectを作らず、ResearchPlanの独立State Machineとして表現する

また:

```text
RANDOM_BASELINE
HISTORICAL
REPLAY
PAPER
DEMO_FORWARD
SHADOW
COUNTERFACTUAL
```

はExperiment Modeであり、Objectではない。

`DemoForwardTrial` を別Objectとして乱立させず、基本は `ResearchTrial + experiment_mode = DEMO_FORWARD` として表現する。

---

# 25. 未確定・次文書へ送る論点

以下は本辞書で名前と意味を整理するが、最終仕様は後続文書で確定する。

1. 正式State一覧 → `STATE_DICTIONARY.md`
2. Market DNAとRegimeの正式依存関係 → Architecture / State設計
3. **Feature Priorityが参照するDNAの時系列原則はFIX-006で確定済み。Field型・Required/Nullable・Cycle境界Validationは `DATA_CONTRACT.md` / Data Flowで固定する**
4. **MarketEventのCanonical生成責任はFIX-007でEvent Detection Processorへ確定済み。Event Field型・Detection Rule参照型・Dedup/Cardinality・Detector HealthのContractは `DATA_CONTRACT.md` / Data Flowで固定する**
5. **ApplicableHypothesisSet / TradeThesisのCanonical生成責任はFIX-008でProduction Thesis Builderへ確定済み。Builder Input Contract、Selection Cardinality、Builder Version型、`THESIS_NOT_BUILDABLE`表現、Set→Thesis→Signal受け渡しは `DATA_CONTRACT.md` / Processing / Trade Thesis Contractで固定する**
6. **EntryThesis / ProductionEvidence生成責任はFIX-009で確定済み。Entry Snapshot hard gate、OrderIntent必須参照、時系列制約、Live Evidence completeness、ExecutionRecordとのCardinality、Logger Custody / Post-Trade read-only境界は `DATA_CONTRACT.md` / Processing Contractで固定する**
7. **成功State変更履歴はFIX-010で`StateTransitionEvent`へ正式化済み。Authority最終割当、Atomic Write、Sequence uniqueness、CAS、Replay、Retention、AuditEvent関連は `AUTHORITY MATRIX` / `DATA_CONTRACT.md` / Processing / DB Schemaで固定する**
8. **ResearchPlanはFIX-011でLifecycle State / Lock Stateの2軸へObject側も正式追従済み。Stateの正式語彙は`STATE_DICTIONARY.md`、Trial開始時FROZEN Gate・Plan Version binding・new T0・supersedes関係の型/CardinalityはResearch / Data / Processing Contractで固定する**
9. Soft / Hard ContradictionのProduction Gate → Production Contract
10. Risk Stateの閾値 → Risk Design / Research
11. Object Fieldの型 / Required / Nullable → `DATA_CONTRACT.md`
12. Object間Cardinality → `DATA_CONTRACT.md` / `DATABASE_SCHEMA.md`
13. Retention期間 → Storage Governance
14. Encryption / Secret分類 → Security Design
15. Schema Migration実装 → Version / Migration Design

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
>
> **Codeはこれらを実装する手段であり、意味の正本ではない。**

FIX-009ではさらに:

> **GeneratorはObjectを作る。Custodianは改変せず保存する。Analyzerは後から意味を分析する。**

FIX-010ではさらに:

> **Current Stateは現在値のProjectionであり、StateTransitionEventは実際に適用された遷移のImmutable Historical Factである。**

FIX-011ではさらに:

> **ResearchPlan Version・Lifecycle State・Lock Stateは別軸であり、ResearchTrialは実行時に使用したPlan VersionとLock条件を固定参照する。**

何十年以上運用するため、Python・DB・Exchange・AI Providerが変わっても、Objectの意味・Version・Provenance・Research履歴・Trade判断履歴・State Transition履歴を失わない設計を優先する。