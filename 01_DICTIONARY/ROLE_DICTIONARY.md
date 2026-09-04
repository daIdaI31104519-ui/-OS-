# ROLE_DICTIONARY.md

# 市場理解OS 役割辞書・責任境界完全設計

## 0. 文書情報

- 文書種別: DICTIONARY / ROLE SOURCE OF TRUTH
- 状態: CANONICAL DICTIONARY CANDIDATE
- 上位ルール:
  - `00_GOVERNANCE/GIT_RULES.md`
  - `00_GOVERNANCE/DESIGN_CHANGE_RULES.md`
- 対象: 市場理解OSのArchitecture / Research / Production / Platform / Governanceで使用するRole
- 目的: 各Roleの目的・責任・入力・出力・禁止事項・Failure時挙動・Researchへの戻し方を一元管理し、責任重複とLayer乱立を防ぐ
- 最上位原則: **Roleは「何を担当するか」で定義し、Object / Contract / View / Experiment Modeと混同しない**

---

# 1. Role Dictionaryの目的

市場理解OSを何十年以上運用する前提では、同じ責任を複数Moduleへ重複実装したり、新しい案のたびに新Layerを増やすと設計が崩れる。

そのため、本書をRole定義のSingle Source of Truth候補とする。

本書で解決すること:

1. 各Roleが何を担当するか
2. 各Roleが何を担当してはいけないか
3. 上流・下流の責任境界
4. Objectの所有・生成責任
5. Failure時にどこまで処理してよいか
6. Research Candidateをどこへ返すか
7. 長期運用時にRoleが肥大化しないための境界
8. 新Role追加が本当に必要か判断する基準

---

# 2. Role / Object / Contract / View / Modeの区別

## Role

処理・判断・管理の責任を持つ。

例:

```text
Collector
Market Intelligence
Causal Engine
Production Thesis Builder
Signal Engine
Position Supervisor
Runtime
```

## Object

Role間で生成・保存・受け渡しされる情報構造。

例:

```text
RawData
MarketEvent
Evidence
CausalHypothesis
MarketDNA
ApplicableHypothesisSet
TradeThesis
EntryThesis
ProductionEvidence
OrderIntent
TradeResult
```

## Contract

Role間で何をどの形式で渡すかを定義する規則。

例:

```text
Data Contract
Processing Contract
Error Contract
Research Contract
Trade Thesis Contract
```

## View

同じKnowledge Objectを異なる目的で参照する表示・検索方法。

例:

```text
Case Library
Market Memory
Failure Museum
Knowledge Graph
```

## Experiment Mode

Research Domain内で実験方式を切り替えるためのMode。

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

> Object / Contract / View / Modeを、理由なく独立Top-Level Roleへ昇格させない。

---

# 3. Role定義の標準フォーマット

全Roleは原則として次を定義する。

```text
Role ID
Role Name
Category
Status
Purpose
Owns
Inputs
Outputs
Upstream
Downstream
Responsibilities
Prohibitions
Failure Behavior
Research Feedback
Long-Term Notes
```

Role詳細設計では本辞書の責任境界を変更せず、内部処理だけを深掘りする。

責任境界を変更する場合は `DESIGN_CHANGE_RULES.md` に従う。

---

# 4. Role分類

市場理解OSのRoleを大きく次へ分ける。

```text
A. GOVERNANCE / CONTROL
B. CONNECTION / ADAPTER
C. MARKET OBSERVATION / UNDERSTANDING
D. RESEARCH / KNOWLEDGE
E. PRODUCTION / TRADING
F. POST-TRADE / FEEDBACK
G. PLATFORM / OPERATIONS
```

---

# A. GOVERNANCE / CONTROL

# ROLE-GOV-001: Governance / Constitution Authority

## Category
GOVERNANCE

## Status
ACTIVE DESIGN CANDIDATE

## Purpose

市場理解OS全体が長期原則・安全原則・設計変更原則へ違反しないよう統治する。

## Owns

- Project Constitution
- Git Governance
- Design Change Governance
- Long-Term Governance Rules
- Canonical / Proposal境界

## Inputs

- Design Change Proposal
- Architecture Review
- Risk / Failure / Research Governance情報
- Migration Requirement

## Outputs

- Governance Decision
- Change Constraint
- Approval Requirement
- HOLD / REJECT / MERGE / ADOPT判断材料

## Responsibilities

- 上位設計原則を維持する
- CanonicalとProposalを混同させない
- 長期互換性を確認する
- 安易な新Layer追加を防ぐ

## Prohibitions

- Market方向判断
- BUY / SELL生成
- Research Resultの改ざん
- Production設定の直接自動変更

## Long-Term Notes

何十年運用ではコードより寿命が長い可能性があるため、実装から独立して保持する。

---

# ROLE-CTL-001: Outer Control

## Category
GOVERNANCE / CONTROL

## Purpose

市場理解OSを「何を対象に、どの設定・資金上限・接続条件で動かすか」という外側のDesired Stateを管理する。

## Owns

- Market Profile選択
- Runtime Command要求
- Global Risk Limit入力
- 接続先選択方針

## Inputs

- User / Operator Command
- Market Profile
- Global Risk Policy
- System Status

## Outputs

- RuntimeCommand
- MarketProfile / Configuration reference
- GlobalRiskLimit
- Adapter Configuration

## Upstream

- Human / Telegram Interface
- Governance

## Downstream

- Runtime
- Source Adapter
- Exchange Adapter
- Core

## Responsibilities

- どのMarket Instanceを動かすか決める
- START / PAUSE / RESUME / STOP要求を出す
- 資金・Exposure上限を渡す

## Prohibitions

- 市場解釈
- Causal判断
- Feature Priority決定
- BUY / SELL
- Processを自力で直接管理すること

## Boundary

```text
Outer = Desired State
Runtime = Actual Execution
```

---

# B. CONNECTION / ADAPTER

# ROLE-ADP-001: Source Adapter

## Category
CONNECTION / ADAPTER

## Purpose

外部Data Provider固有形式を市場理解OS標準形式へ変換する。

## Owns

- Provider固有Schema変換
- Symbol / Field mapping
- Source provenance付与

## Inputs

- Exchange / Provider API Response
- WebSocket Message
- Vendor-specific Data

## Outputs

- RawData-compatible payload
- SourceMetadata
- Diagnostics

## Responsibilities

- Provider差をCoreから隠す
- Source情報をProvenanceとして残す
- Schema mismatchをDiagnosticsへ出す

## Prohibitions

- Feature計算
- Market Interpretation
- Signal生成
- Cause判定

## Failure Behavior

変換不能・Schema mismatchを明示し、推測値で埋めない。

---

# ROLE-ADP-002: Exchange Adapter

## Category
CONNECTION / ADAPTER

## Purpose

標準OrderIntentを取引所固有注文へ変換し、注文・約定情報を標準形式へ戻す。

## Owns

- Symbol mapping
- Tick size / Minimum order対応
- API authentication interface
- Order request / response conversion
- ExecutionRecordの生成責任

## Inputs

- OrderIntent
- Exchange Configuration

## Outputs

- ExecutionRecord
- Diagnostics

## Responsibilities

- Vendor固有API処理
- 約定情報取得
- Exchange response標準化

## Prohibitions

- BUY / SELL判断
- Position Sizeの戦略判断
- Trade Thesis生成
- EntryThesis生成
- ProductionEvidenceのSemantic生成責任
- Market理解

---

# C. MARKET OBSERVATION / UNDERSTANDING

# ROLE-DATA-001: Collector

## Category
MARKET OBSERVATION

## Purpose

外部世界からRaw Dataを取得する。

## Inputs

- Exchange
- Macro / ETF / News / SNS / On-chain / Other Sources

## Outputs

- RawData
- SourceMetadata
- Diagnostics

## Responsibilities

- 取得
- Timestamp / Source metadata保持
- Request / Receive latency記録

## Prohibitions

- Feature計算
- 「強気 / 弱気」最終解釈
- Cause判断
- BUY / SELL

## Failure Behavior

取得失敗を隠さずData Quality / Monitoringへ伝播する。

## Research Feedback

繰り返すSource latency / missing patternはResearchCandidate化可能。

---

# ROLE-DATA-002: Normalizer

## Category
MARKET OBSERVATION

## Purpose

Sourceごとの表現差を統一し、RawDataを比較・研究可能なObservationへ変換する。

## Inputs

- RawData

## Outputs

- Observation
- Diagnostics

## Responsibilities

- Timestamp / Timezone
- Symbol
- Unit
- Decimal
- Interval
- Naming
- Currency
- Missing representation

## Prohibitions

- 品質を最終確定する
- Feature生成
- Market判断

## Failure Behavior

変換失敗・補正量・Schema mismatchをDiagnosticsへ残す。

---

# ROLE-DATA-003: Data Quality

## Category
MARKET OBSERVATION / SAFETY

## Purpose

下流が入力をどこまで信用できるか判断できる品質情報を生成する。

## Inputs

- RawData / Observation
- SourceMetadata

## Outputs

- QualityProfile
- Diagnostics

## Responsibilities

- Missing
- Duplicate
- Latency
- Staleness
- Outlier
- Timestamp drift
- Cross-source inconsistency
- API failure
- Impossible value

## Prohibitions

- Market方向判断
- Cause判定
- Low Qualityを隠す

## Failure Behavior

自Role自体が不調な場合は下流へHigh Confidenceを許可しない。

---

# ROLE-DATA-004: Time Series Processor

## Category
MARKET OBSERVATION / MEASUREMENT

## Purpose

単一値を時間変化として扱える系列へ整形する。

## Inputs

- Observation
- QualityProfile

## Outputs

- TimeSeriesMeasurement

## Responsibilities

- Delta
- Return
- Rate of Change
- Velocity
- Acceleration
- Persistence
- Rolling values
- Lagged values

## Prohibitions

- 任意の時間窓を暗黙利用する
- Signal生成

## Long-Term Notes

sampling / measurement / lookback windowを明示し、Version追跡可能にする。

---

# ROLE-CALC-001: Calculation / Measurement Service

## Category
CROSS-CUTTING MEASUREMENT

## Purpose

OS全体で利用する計算式・測定定義を共通化する。

## Owns

- Formula Registry
- Feature Registryの計算定義部分
- Metric Registry
- Unit Rules
- Time Window Rules

## Inputs

- RawData / Observation / TimeSeriesMeasurement / Other approved inputs

## Outputs

- MeasurementResult
- FormulaDefinition reference

## Responsibilities

- 同じ概念を複数Roleで勝手に別Formula実装しないようにする
- Formula Versionを残す

## Prohibitions

- 「数学的に計算できる」だけでMarket意味を確定する
- Research未承認FormulaをProduction正式Formulaへ昇格

## Research Feedback

Formula比較・Sensitivity・OOS・StressはFormula Researchへ戻す。

---

# ROLE-FEAT-001: Feature Generator

## Category
MARKET UNDERSTANDING

## Purpose

RawData / Observation / TimeSeriesMeasurementから市場理解に利用できるVersioned Featureを生成する。

## Inputs

- Approved FormulaDefinition
- Observation / TimeSeriesMeasurement
- QualityProfile

## Outputs

- Feature
- FormulaDefinition reference
- QualityProfile reference
- Trace reference

## Responsibilities

- Feature値生成
- 入力参照保持
- Formula Version保持

## Prohibitions

- BUY / SELL
- Cause確定
- 自分のFeatureを自分でHigh Priorityにする

---

# ROLE-EVT-001: Event Detection Processor

## Category
MARKET OBSERVATION / EVENT DETECTION

## Purpose

Observation / TimeSeriesMeasurement / Featureから、研究・比較可能な「市場で発生した現象」を検出し、Canonicalな`MarketEvent`として固定する。

## Owns

- MarketEvent生成責任
- Event Detection Ruleの適用
- Event timing / magnitude / provenance記録
- 同一実市場Eventの重複統合・Canonical Event ID付与

## Inputs

- Observation
- TimeSeriesMeasurement
- Feature
- QualityProfile
- SourceMetadata reference
- Versioned Event Detection Rule / Configuration reference

## Outputs

- MarketEvent
- Diagnostics
- ResearchCandidate（Detection miss / false positive / rule degradation等を検出した場合）

## Upstream

- Normalizer
- Time Series Processor
- Feature Generator
- Data Quality

## Downstream

- Feature Priority
- Market Intelligence
- Causal Engine
- Research

## Responsibilities

- OI Shock / Liquidation Shock / Funding急変 / Spread急拡大 / Liquidity Collapse / ETF Flow Shock / Volume Spike / Volatility Expansion等の「発生事実」を検出する
- `detected_at / start_at / peak_at / end_at` 等の時間情報を可能な範囲で保持する
- Detection Rule / Rule Versionを残し、後から当時のEvent判定を再現可能にする
- 複数Source / Observationが同一実市場Eventを示す場合、Unique Market Event数を水増ししないようCanonical Eventへ統合する
- `NO_EVENT_DETECTED` と `EVENT_DETECTION_UNAVAILABLE / DEGRADED` を区別する
- Data Quality低下時はEventのConfidence / Quality制約を明示する

## Prohibitions

- MarketEventへBUY / SELL意味を埋め込む
- `BTC_WILL_CRASH` 等の予測をEvent Typeとして生成する
- `WHALES_ARE_MANIPULATING_MARKET` 等の因果解釈をEvent Typeとして生成する
- Causeを確定する
- MarketContextを生成する
- Trade Signalを生成する
- Event Detection失敗を「Eventなし」と偽装する

## Failure Behavior

Event Detection自体がDEGRADED / ERRORの場合、空のMarketEvent一覧だけで正常扱いしない。DiagnosticsへDetector Health / Coverage Impactを残し、下流が「Eventなし」と「Event判定不能」を区別できるようにする。

## Research Feedback

- Market Intelligenceが重要変化を認識したのに対応するMarketEventが存在しない場合、MIはCanonical Eventを直接作らず`UnexplainedEvent` / `ResearchCandidate`としてEvent Detection Researchへ戻す
- False Positive / Detection Miss / Duplicate Split / Duplicate Merge Error / Rule Stalenessを研究対象化できる

## Boundary

```text
Event Detection = 何が発生したかを検出・識別する
Market Intelligence = そのEventを市場全体の文脈で解釈する
Causal Engine = なぜ発生した可能性があるかを検証可能な因果候補へ変換する
```

## Long-Term Notes

独立Top-Level PlaneではなくMarket Observation Plane内の軽量Role / Componentとして維持する。Event Detection Rule自体を将来Research / Version管理できるようにするが、現段階ではRuleを新しい巨大Layerへ昇格させない。

---

# ROLE-FEAT-002: Feature Priority

## Category
MARKET UNDERSTANDING

## Purpose

「どのFeatureが常に最強か」ではなく、「現在条件では何を見る価値が高いか」を決める。

## Inputs

- Historical / OOS usefulness
- Current Basic Context / already-available MarketEvent information
- Time / Horizon
- QualityProfile
- Feature Stability / Redundancy
- **Previous Confirmed MarketDNA reference (`DNA_t-1`)**

## Outputs

- FeaturePriorityProfile

## Responsibilities

- 現在Cycleで注目するFeatureの優先順位を生成する
- Low Priority判断も保存する
- FeaturePriorityProfileへ参照したPrevious Confirmed MarketDNAと、その`as_of` / Versionを残す
- Cold StartでPrevious Confirmed MarketDNAが存在しない場合は、Baseline / Sentinel中心でPriorityを生成し、DNA未参照であることを明示する
- Major Regime / Structural Event等で再計算が必要な場合は、同一Cycleを自己参照で再計算せず、新しいEvaluation Cycleとして再評価する

## Prohibitions

- Low Priority FeatureのRaw取得を無条件停止する
- Signal化
- Priority = Confidenceと扱う
- **同一Evaluation Cycleの後段で生成される`DNA_t`を、同じCycleのFeature Priority入力へ戻すこと**
- FeaturePriorityProfile生成後に、そのCycleのMarketDNAを使ってPriority理由を事後改変すること
- Sentinel FeatureをPriority低下だけで観測停止すること

## FIX-006 Cycle Boundary

通常経路を次へ固定する。

```text
Previous Confirmed MarketDNA = DNA_(t-1)
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
次Evaluation Cycleで参照可能
```

禁止循環:

```text
FeaturePriority_t
→ MarketDNA_t
→ FeaturePriority_t
```

重大Event時も`DNA_t`を同じCycleへ逆流させない。必要なら新しい`evaluation_cycle_id`を発行し、直前に確定済みのMarketDNAと最新Context / MarketEventを使って新Cycleとして再評価する。

## Research Feedback

Priority判断自体の失敗をFeature Researchへ戻す。

---

# ROLE-MI-001: Market Intelligence

## Category
MARKET UNDERSTANDING

## Purpose

「今、市場で何が起きているか」を、Feature / MarketEvent / Quality等を組み合わせて市場全体の文脈として構造化・解釈する。

## Inputs

- Feature
- FeaturePriorityProfile
- MarketEvent
- QualityProfile
- Context

## Outputs

- MarketContext
- Evidence
- Contradiction
- UnexplainedEvent
- ResearchCandidate（必要時）

## Responsibilities

- Price / Structure
- Volatility
- Liquidity
- Derivatives
- Participant Flow
- ETF / Institutional
- Macro
- News / Sentiment
- Abnormal Event

を市場状態として整理する。

## Prohibitions

- Canonical MarketEventを直接生成する
- Event Detection Ruleの代わりに事後的な物語でEventを捏造する
- 原因を確定する
- Causal Hypothesisを最終確定する
- BUY / SELL

## Failure Behavior

説明不能なら物語を作らず `UnexplainedEvent` とする。

Event DetectionがDEGRADED / UNAVAILABLEの場合、「Eventなし」と解釈せずQuality / Uncertaintyへ影響を伝播する。

## Research Feedback

UnexplainedEvent / interpretation anomalyをResearchCandidateへ送る。

重要変化を認識したのに対応するCanonical MarketEventが存在しない場合は、MarketEventを直接追加せずEvent Detection Miss候補としてResearchCandidateへ送る。

---

# ROLE-CAUSAL-001: Causal Engine

## Category
MARKET UNDERSTANDING / CAUSAL

## Purpose

観測されたEffectに対して、検証可能なCause Candidate・Alternative・Contradiction・検証要求を構造化する。

## Inputs

- MarketContext
- Evidence
- MarketEvent
- Feature / FeaturePriorityProfile
- MarketCase
- QualityProfile / Uncertainty

## Outputs

- CauseCandidate
- CausalHypothesis
- Contradiction
- ResearchCandidate
- HypothesisAssessmentProfile / State Transition recommendation

## Responsibilities

- Cause Candidate
- Effect
- Temporal Order
- Lag
- Confounder
- Alternative HypothesisをCausalHypothesis関係として保持
- Contradiction
- Evidence Strength profile
- Historical / OOS / Regime / Stress検証要求をResearchCandidateへ変換
- Hypothesis Lifecycle管理材料

## Prohibitions

- MarketEventのCanonical生成責任を持つ
- 相関だけでCause確定
- Historical / OOS / Stress大規模実験を自Roleで全部実行
- BUY / SELL

## Boundary

```text
Event Detection = 何が発生したか
Market Intelligence = 発生したEventを市場文脈でどう解釈するか
Causal Engine = なぜ発生した可能性があるか・何を検証すべきか
Research = 実際に検証する
```

---

# ROLE-DNA-001: Market DNA Builder / Manager

## Category
MARKET UNDERSTANDING / KNOWLEDGE INTERFACE

## Purpose

現在市場を過去市場と比較・検索・研究可能な圧縮状態表現へ変換する。

## Inputs

- MarketContext
- Feature
- Causal / Evidence information
- Approved DNA Formula / Axis definitions

## Outputs

- MarketDNA
- MarketCase references

## Responsibilities

- 現Evaluation CycleのMarketDNA (`DNA_t`) を生成する
- DNA axis計算
- Similarity / NoveltyをMarketDNA内の追跡可能なProfileとして生成
- Raw / Feature / Formula Versionへ追跡可能にする
- `evaluation_cycle_id` と必要に応じてPrevious Confirmed MarketDNA参照を保持し、Cycle間の因果順序を追跡可能にする

## Prohibitions

- Market DNAをSignalそのものにする
- 「似ている = 同じ結果」と断定する
- Feature Priorityと同一概念化する
- **生成した`DNA_t`を同じEvaluation CycleのFeature Priorityへ逆流させること**

## Research Feedback

低Similarity / 高NoveltyはNovel Regime ResearchCandidate候補。

---

# D. RESEARCH / KNOWLEDGE

# ROLE-RSCH-001: Research Orchestrator

## Category
RESEARCH

## Purpose

ResearchCandidateを適切な研究方式へ振り分け、実験・Validation・EvidencePackage生成を管理する。

## Inputs

- ResearchCandidate
- CausalHypothesis / Edge / Feature / Formula / MarketDNA等の研究対象
- Failure / Unexpected Success
- DemoLiveDivergence

## Outputs

- ResearchPlan
- ResearchResult
- EvidencePackage
- Edge / Candidate Knowledge
- NegativeKnowledge candidate

## Responsibilities

- Research Queue
- Research Priority
- Experiment selection
- Research lifecycle
- Budget / Stop Ruleとの連携

## Prohibitions

- 未承認結果を直接Productionへ反映
- Candidateを自動でCanonical Knowledgeにする
- 無制限研究

## Long-Term Notes

Research Governanceに従い、開始・継続・停止・再開を管理する。

---

# ROLE-RSCH-002: Experimental Framework

## Category
RESEARCH SUBSYSTEM

## Purpose

複数の実験方式を共通基盤で実行する。

## Modes

- RANDOM_BASELINE
- HISTORICAL
- REPLAY
- PAPER
- DEMO_FORWARD
- SHADOW
- COUNTERFACTUAL

## Inputs

- ResearchPlan
- CausalHypothesis / ApplicableHypothesisSet candidate / other research target
- Market Data / Replay Data
- Execution Model

## Outputs

- ResearchResult
- Evidence Source Channel / Metadata reference
- Diagnostics

## Responsibilities

- Modeごとの時間・情報可用性ルールを守る
- Demo ForwardでT0以降だけを使う
- DemoとLiveで判断ロジックを可能な限り共通化する

## Prohibitions

- RandomをReal Money目的で使う
- Historical / Demo / Live結果を単純合算する
- 未来情報をReplayへ混ぜる

---

# ROLE-RSCH-003: Validation Framework

## Category
RESEARCH / VALIDATION

## Purpose

ResearchCandidate / Edge / Hypothesis SetがProduction候補になれるか検証する。

## Inputs

- ResearchResult
- CausalHypothesis / Edge candidate

## Outputs

- ResearchResult
- FailureBoundary
- Constraint candidate / reference

## Responsibilities

- OOS
- Regime Stability
- Stress Lab
- FailureBoundary抽出
- Historical / OOS / Regime / Demo / Stressの違いは別Object名ではなくResearchResultのEvidence Source Channel / Experiment Modeで保持する

## Prohibitions

- 検証成功だけで自動Production有効化
- Stress LabをResearch全体そのものとして扱う

---

# ROLE-KNOW-001: Knowledge Domain Manager

## Category
KNOWLEDGE

## Purpose

再利用可能な市場理解知識を一貫したObject群として保存・参照・Version管理する。

## Owns / Manages

- MarketCase
- MarketDNA references
- CausalHypothesis records
- FeatureKnowledge
- FormulaKnowledge
- ResearchResult
- Failure
- FailureBoundary
- Constraint
- NegativeKnowledge
- KnowledgeRelationship

## Inputs

- Approved / Candidate Knowledge
- ResearchResult
- ProductionEvidence
- Post-Trade Attribution

## Outputs

- Knowledge Object references / Query result
- MarketCase references
- Approved Knowledge references
- ResearchCandidate（Aging / Revalidation必要時）

## Responsibilities

- 同じ知識を複数DB/Viewへ無駄にコピーしない
- Version / provenance保持
- Knowledge Aging対応

## Prohibitions

- Viewごとに別の正本を作る
- 古い知識を永久真理扱いする

---

# ROLE-KNOW-002: Knowledge Promotion / Approval Gate

## Category
GOVERNANCE FUNCTION / KNOWLEDGE

## Purpose

ResearchResultをProduction利用可能Knowledgeへ昇格させる前に、Validation・状態・互換性を確認する。

## Inputs

- Candidate Knowledge / Edge
- ResearchResult / EvidencePackage
- Constraint
- Uncertainty

## Outputs

- APPROVED
- HOLD
- REJECT / RETIRE candidate
- Revalidation requirement

## Prohibitions

- ResearchResultを無検証でProductionへ通す
- AI Reviewだけで承認する

## Note

独立巨大Layerではなく、正式なApproval責任を表すGovernance Function。

---

# E. PRODUCTION / TRADING

# ROLE-PRD-001: Production Thesis Builder

## Category
PRODUCTION / THESIS CONSTRUCTION

## Purpose

Approved / production-eligibleなHypothesis・Edgeを現在市場の条件へ照合し、`ApplicableHypothesisSet` と `TradeThesis` を一貫した責任主体として生成する。

Builderが答える問い:

> **今の市場で、どの承認済みHypothesis / Edgeが適用可能で、それらをどの役割・依存関係・反証条件のもとで一つのTrade Thesisとして構成できるか。**

## Owns

- `ApplicableHypothesisSet` のCanonical生成責任
- `TradeThesis` のCanonical生成責任
- Approved Poolから現在市場へのApplicability Selection
- PRIMARY / SUPPORTING / CONDITIONAL / CONTRADICTING役割付与
- Shared Evidence / Dependency / Redundancy / Common Causeの明示
- Thesis Composition / Invalidation Conditionの構造化

## Inputs

- HypothesisPoolEntry / Approved Hypothesis / Approved Edge references
- MarketContext
- Current Confirmed MarketDNA
- ApplicabilityProfile
- Constraint
- Evidence / Contradiction
- HypothesisAssessmentProfile
- KnowledgeLifecycleProfile
- Production Promotion Stage
- QualityProfile / Uncertainty
- MarketCase references（必要時）

## Outputs

- ApplicableHypothesisSet
- TradeThesis
- Diagnostics
- ResearchCandidate（適用不能・組合せ矛盾・Knowledge gap等を研究へ戻す場合）

## Upstream

- Knowledge Promotion / Approved Knowledge Pool
- Market Intelligence
- Market DNA
- Knowledge Domain

## Downstream

- External AI Review（optional）
- Signal Engine
- Post-Trade Analysis / Research（評価用）

## Internal Responsibility Split

新しいTop-Level Roleを複数作らず、内部責任を次へ分ける。

```text
Production Thesis Builder
├─ Applicability Selector
├─ Dependency / Redundancy Checker
├─ Contradiction Integrator
└─ Thesis Composer
```

### Applicability Selector

Approvedであることだけを理由に利用可能としない。

```text
Approved Knowledge
+
Current MarketContext / MarketDNA
+
ApplicabilityProfile
+
Constraint
+
Knowledge Aging
+
Production Promotion Stage
+
Quality / Uncertainty
↓
Applicable candidate
```

原則:

```text
Approved
≠ Applicable
≠ Trade-worthy
```

### Dependency / Redundancy Checker

- 同じEvidenceを複数Hypothesisが共有していないか確認する
- 依存Hypothesisを独立した票として数えない
- 同一Common Cause由来のHypothesisを独立Evidenceとして水増ししない
- 冗長Hypothesisを多数決へ利用しない

### Contradiction Integrator

- Contradicting Hypothesisを消さずSetへ保持する
- Soft / Hard Contradictionを後続Contractで区別可能な形にする
- Constraint違反を単なる減点として隠さない

### Thesis Composer

`ApplicableHypothesisSet` から、現在市場で検討可能な一つの論拠として `TradeThesis` を構成する。

TradeThesisでは次を表現可能とする。

```text
expected_direction
expected_effect
expected_horizon
primary / supporting / conditional / contradicting hypotheses
main_risks
invalidation_conditions
quality
uncertainty
```

ただし `BUY / SELL / NO_TRADE` の最終Decisionは生成しない。

## Responsibilities

- Production Poolから現在市場に本当に適用可能なKnowledgeだけを選ぶ
- Hypothesis数ではなく役割・Evidence独立性・Dependency・Constraintを評価する
- `PRIMARY / SUPPORTING / CONDITIONAL / CONTRADICTING` を明示する
- `shared_evidence_map / dependency_map / redundancy_map / common_cause_map` を保持する
- Current Confirmed MarketDNAとApplicability条件を照合する
- Production Promotion Stage / `max_production_stage` を超えた利用を許可しない
- Knowledge Aging / Staleness / Revalidation requirementを無視しない
- TradeThesisのExpected Effect / Horizon / Invalidationを明示する
- Thesis生成時点の入力Version / referencesを追跡可能にする

## Prohibitions

- 新しいCausal HypothesisをProduction内で作る
- DRAFT / RESEARCHING / 未承認HypothesisをLive Thesisへ追加する
- AIがその場で提案した未検証HypothesisをTradeThesisへ追加する
- Hypothesis多数決でDirectionを決める
- Shared Evidenceを重複加点する
- Constraintを無視する
- Production Promotion Stageを勝手に昇格させる
- Risk Limitを変更する
- `SignalDecision` を生成する
- `EntryThesis` を生成する
- BUY / SELL / NO_TRADEを最終確定する
- Thesisを作れない場合に不足情報を推測して捏造する

## Failure Behavior

有効なThesisを構成できない場合、無理にTradeThesisを生成しない。

代表例:

```text
PRIMARY候補なし
Approvedだが全候補がApplicability外
Hard Constraint違反
Critical Data Quality問題
Knowledge Stale / Revalidation required
重大なDependency / Shared Evidence問題
Contradictionが解消不能
```

この状態は `SignalDecision.NO_TRADE` と区別する。

```text
THESIS_NOT_BUILDABLE
= Signal Engineへ渡せる有効なTradeThesis自体が成立していない

SignalDecision.NO_TRADE
= 有効なTradeThesisは存在するが、Signal EngineがRiskを取らないと判断した
```

`THESIS_NOT_BUILDABLE` の正式な表現方法は後続Data / Processing Contractで固定する。

## Research Feedback

次のような状況をResearchCandidateへ戻せる。

```text
NO_APPLICABLE_HYPOTHESIS
REPEATED_THESIS_NOT_BUILDABLE
HYPOTHESIS_SET_COMPOSITION_CONFLICT
SHARED_EVIDENCE_OVERLAP
APPLICABILITY_GAP
CONTRADICTION_DOMINANCE
KNOWLEDGE_STALENESS_GAP
```

Builder自身がResearch結果を生成したりProduction Ruleを自動更新しない。

## Boundary

```text
Research / Knowledge Promotion
= Hypothesis / Edgeを研究・検証・承認する

Production Thesis Builder
= 承認済みKnowledgeを現在市場に照合しApplicableHypothesisSet / TradeThesisへ組み立てる

Signal Engine
= そのTradeThesisにRiskを取る合理性・期待値があるか判断する

Pre-Trade Defense
= Signalが存在しても今Riskを取って安全か判断する
```

## Long-Term Notes

- BuilderはResearchとProductionの境界を壊さないことを最優先する
- Selection Logic / Composition LogicはVersion管理し、過去TradeでどのBuilder Versionを使ったか再現可能にする
- Hypothesis SelectionとThesis Compositionを別Top-Level Layerへ分裂させず、必要な限り内部Submoduleとして管理する

---

# ROLE-AI-001: External AI Review

## Category
PRODUCTION ADVISORY

## Purpose

市場理解OSが作ったTradeThesis / Evidence / ApplicableHypothesisSetを外部知能として独立査読する。

## Inputs

- TradeThesis
- MarketDNA
- Evidence
- Contradiction
- EvidencePackage
- Constraint

## Outputs

- AIReviewResult
- ResearchCandidate（新規研究案が出た場合）

## Responsibilities

- 重複仮説指摘
- Shared Evidence二重評価指摘
- 最大反証・見落とし確認

## Prohibitions

- 最終Trade決定権
- 未検証Hypothesisをその場でProduction Thesisへ追加
- AI多数決を真実扱い
- EntryThesis生成

## Failure Behavior

AI Review停止でもProduction Coreが原則動作可能にする。

---

# ROLE-SIG-001: Signal Engine

## Category
PRODUCTION

## Purpose

Production Thesis Builderが生成した現在のTradeThesisに、Riskを取る合理性・期待値があるか判断する。

## Inputs

- TradeThesis
- ApplicableHypothesisSet reference
- Edge
- Expected Value
- MarketContext / MarketDNA
- MarketCase references
- FeaturePriorityProfile
- Constraint
- QualityProfile / Uncertainty
- AIReviewResult

## Outputs

- SignalDecision

## Responsibilities

- Expected Value中心の取引候補判断
- Causal / Empirical Edge双方を利用可能にする
- `BUY / SELL / NO_TRADE` のDecision責任を持つ

## Prohibitions

- ApplicableHypothesisSet / TradeThesisのCanonical生成責任を吸収する
- EntryThesis生成責任を持つ
- Defense責任を吸収する
- AIReviewResultだけで決定
- 全情報を意味不明な一つの総合Scoreへ潰す
- 未承認Hypothesisをその場でThesisへ追加する

---

# ROLE-DEF-001: Pre-Trade Defense

## Category
PRODUCTION / SAFETY

## Purpose

SignalDecisionが存在しても、現在Riskを取って安全かGateする。

## Inputs

- SignalDecision
- Constraint
- QualityProfile
- Liquidity / Spread / Slippage Risk
- DD / Consecutive Losses
- Exposure
- API / Exchange Health
- Abnormal Event
- GlobalRiskLimit

## Outputs

- DefenseDecision

## Responsibilities

- Hard / Safety条件確認
- Risk制約適用

## Prohibitions

- Expected Valueを作る
- TradeThesis生成
- EntryThesis生成
- BUY / SELL方向生成

## Research Feedback

BLOCKしたTradeThesisはShadow追跡候補。

---

# ROLE-EXEC-001: Execution Logic

## Category
PRODUCTION / EXECUTION

## Purpose

SignalDecision / DefenseDecision承認後、実際にRiskを取る直前の判断根拠を`EntryThesis`として固定し、そのSnapshotに基づいて取引所非依存の`OrderIntent`を生成する。約定後はExecution Domain内部のLive Evidence Collectorが、ExecutionRecordとLive市場文脈から`ProductionEvidence`を生成する。

## Owns

- 標準注文計画
- `Entry Snapshot Builder` の内部責任
- `Live Evidence Collector` の内部責任
- EntryThesisのCanonical生成責任
- ProductionEvidenceのCanonical生成責任

## Inputs

### Entry Snapshot / Order Intent生成

- TradeThesis
- ApplicableHypothesisSet reference
- SignalDecision
- DefenseDecision
- RiskState
- MarketContext
- Current Confirmed MarketDNA
- FeaturePriorityProfile
- Evidence / Constraint references
- Formula / Feature Version references
- Risk Budget
- Liquidity / Slippage information

### Live Evidence生成

- ExecutionRecord
- Trade / Position reference
- TradeThesis / EntryThesis reference
- MarketContext
- MarketEvent references
- Funding / Fee / Liquidity / Latency information
- QualityProfile / Diagnostics

## Outputs

- EntryThesis
- OrderIntent
- ProductionEvidence
- Diagnostics
- ResearchCandidate（Entry timing / slippage / live execution anomaly等）

## Internal Responsibility Split

新しいTop-Level Roleを追加せず、Execution Domain内部で次を分離する。

```text
Execution Logic
├─ Entry Snapshot Builder
├─ Order Planning / OrderIntent Builder
└─ Live Evidence Collector
```

### Entry Snapshot Builder

`TradeThesis → SignalDecision → DefenseDecision`まで成立した後、実際に注文を生成する直前のKnowledge / Market / Risk / Versionを`EntryThesis`へ固定する。

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

### Order Planning / OrderIntent Builder

EntryThesis生成成功後に、Position Size / Leverage / Order Type / Split / Stop / Take Profit / Slippage tolerance等を取引所非依存の`OrderIntent`へまとめる。

### Live Evidence Collector

Exchange Adapterが生成した`ExecutionRecord`をそのままProductionEvidenceとみなさず、実際のLive Productionで観測された約定品質・摩擦・Funding・Liquidity Impact等をLive Evidenceとして構造化する。

```text
ExecutionRecord
+ EntryThesis / TradeThesis
+ Live Market / Fee / Funding / Liquidity context
↓
Live Evidence Collector
↓
ProductionEvidence
```

## Responsibilities

- Entry時点のTradeThesis / Hypothesis Set / MarketContext / MarketDNA / FeaturePriority / Signal / Defense / Risk / Constraint / Versionを追跡可能なSnapshotへ固定する
- `entry_snapshot_at` とEntry Snapshot Builder Versionを残す
- EntryThesis生成成功後のみOrderIntent生成へ進む
- Position Size
- Leverage
- Limit / Market
- Split Order
- Stop / Take Profit
- Slippage tolerance
- Liquidity requirement
- ExecutionRecordからActual Fill / Slippage / Fee / Partial Fill / Latency等のLive Evidenceを抽出・構造化する
- Historical / Demo / Liveを混ぜずProductionEvidenceへ`LIVE` Channelを保持する
- ProductionEvidenceが不完全な場合、Quality / Diagnosticsを落として保存可能にし、推測値で穴埋めしない

## Prohibitions

- Exchange固有APIを直接実装
- 市場仮説を作る
- TradeThesis / SignalDecision / DefenseDecisionをEntry時に事後改変する
- EntryThesis生成前にOrderIntentを生成する
- Entry後にEntryThesisを書き換える
- `ExecutionRecord = ProductionEvidence` とみなす
- Historical / Demo EvidenceをProductionEvidenceへ無言で混ぜる
- ProductionEvidence不足を推測値で正常化する
- LoggerへObject生成責任を押し付ける

## Failure Behavior

### Entry Snapshot Failure

次のような場合は`ENTRY_SNAPSHOT_NOT_BUILDABLE`として扱い、OrderIntentを生成しない。

```text
TradeThesis version不明
ApplicableHypothesisSet参照不整合
SignalDecision missing / expired
DefenseDecision missing / blocked / invalid
RiskState stale / unavailable
Critical Context / DNA / Constraint reference不整合
```

`ENTRY_SNAPSHOT_NOT_BUILDABLE`の正式なResult表現は後続Processing Contractで固定する。

### Live Evidence Failure

取引・約定自体が成立していてProductionEvidenceの一部だけが取得不能な場合、Trade事実を消さない。ProductionEvidenceをDEGRADED / INCOMPLETE相当としてQuality / Diagnostics付きで保存し、後からResearch / Monitoringへ送れるようにする。

## Research Feedback

次をResearchCandidateへ戻せる。

```text
ENTRY_SNAPSHOT_TIMING_GAP
ENTRY_TO_SUBMIT_MARKET_DRIFT
SLIPPAGE_ANOMALY
PARTIAL_FILL_ANOMALY
LIVE_FEE_OR_FUNDING_DIVERGENCE
LIQUIDITY_IMPACT_ANOMALY
LIVE_EVIDENCE_INCOMPLETE
DEMO_LIVE_EXECUTION_DIVERGENCE
```

## Boundary

```text
Production Thesis Builder = Trade候補の論拠を構成
Signal Engine = Riskを取りたいか判断
Pre-Trade Defense = Riskを取ってよいか判断
Entry Snapshot Builder = 実際にRiskを取る直前の理由を固定
Execution Logic = どう注文するか
Exchange Adapter = 取引所へどう送り、どう約定したかをExecutionRecord化
Live Evidence Collector = Liveで実際に何が確認されたかをProductionEvidence化
Logger = 生成済みObjectを改変せず保存するCustodian
Post-Trade Analysis = 保存された事実の意味を分析するAnalyzer
```

## FIX-009 Responsibility Principle

```text
Generator
≠ Custodian
≠ Analyzer
```

- EntryThesis Generator = Execution Logic / Entry Snapshot Builder
- ProductionEvidence Generator = Execution Domain / Live Evidence Collector
- Custodian = Logger
- Analyzer = Post-Trade Analysis

## Long-Term Notes

Entry Snapshot / Live Evidence生成LogicはVersion管理し、何年後でも「どのVersionで何を固定・抽出したか」を再現可能にする。内部Submoduleは実装時に別`.py`へ分割可能だが、Architecture上はExecution Domain内責任として扱う。

---

# ROLE-SUP-001: Position Supervisor

## Category
PRODUCTION / IN-TRADE INTELLIGENCE

## Purpose

Position保有中にEntryThesis / TradeThesisがまだ成立しているか監督し、Exit Engineへ助言する。

## Inputs

- EntryThesis
- Hypothesis state updates
- Current Evidence
- MarketContext
- Time / Expected Horizon

## Outputs

- PositionThesisState
- Diagnostics

## Responsibilities

- TradeThesis全体の健全性監視
- Hypothesis Decay / Thesis Decay監視
- Contradicting Hypothesis強化監視

## Prohibitions

- 単一短期Featureだけで即Thesis否定
- 通常Exitを無制限に自動実行
- Entry後に理由を書き換える

## Boundary

```text
Supervisor = Thesis健全性助言
Exit Engine = 通常Exit判断
In-Trade Defense = Hard Safety
```

---

# ROLE-DEF-002: In-Trade Defense

## Category
PRODUCTION / HARD SAFETY

## Purpose

Position保有中の強制安全制約を監視する。

## Inputs

- Position
- Exchange / API Health
- Data Health
- Liquidity
- Risk Limits
- Hard Constraint

## Outputs

- ExitDecision / safety action request
- Diagnostics

## Responsibilities

- Exchange failure
- API failure
- Data collapse
- Liquidity disappearance
- Max Loss
- Global Risk breach

## Prohibitions

- Thesisの意味解釈
- 通常の利益最大化Exit判断

---

# ROLE-EXIT-001: Exit Engine

## Category
PRODUCTION / EXIT

## Purpose

通常のPosition決済判断を行う。

## Inputs

- PositionThesisState
- Supervisor Advisory
- PnL
- Time / Horizon
- Liquidity
- Stop / Take Profit
- RiskState
- In-Trade Defense information

## Outputs

- ExitDecision

## Responsibilities

- 通常Exitの最終判断
- Supervisorの助言とRisk条件を統合

## Prohibitions

- EntryThesisを書き換える
- ResearchCandidateを直接Production Ruleへ反映

---

# F. POST-TRADE / FEEDBACK

# ROLE-LOG-001: Logger

## Category
POST-TRADE / AUDIT / CUSTODY

## Purpose

各Generatorが生成したTrade / Trial / Runtime Objectと事実を、意味を変えず追跡可能な形で保存するCustodian。

## Inputs

- Trade / Trial Events
- EntryThesis
- ProductionEvidence
- SignalDecision / DefenseDecision / ExecutionRecord / PositionThesisState
- MarketEvent / Trace references

## Outputs

- AuditEvent / immutable log records
- TradeResult（Trade完了時）
- Stored Object references

## Responsibilities

- 事実保存
- Version / Trace / Event ID保持
- EntryThesisを改変せず保存
- ProductionEvidenceを改変せず保存
- Generator / created_by_role / provenanceを失わない

## Prohibitions

- EntryThesisを生成する
- ProductionEvidenceを生成する
- EntryThesis / ProductionEvidenceの意味を再解釈して書き換える
- 「なぜ負けた」を確定する
- ResearchResultを生成する
- ログから勝手にTrainerへ送る

## Boundary

```text
Generator = Objectを作る
Logger / Custodian = 作られたObjectを改変せず保存する
Post-Trade Analysis = 保存されたObjectの意味を分析する
```

---

# ROLE-ANL-001: Post-Trade Analysis

## Category
POST-TRADE / ANALYSIS

## Purpose

TradeResult / ProductionEvidence / Demo等の結果を、Trade OutcomeとHypothesis / Execution / Risk Outcomeへ分解する。

## Inputs

- Logger Records
- TradeResult
- EntryThesis
- ProductionEvidence
- Hypothesis states
- ExecutionRecord

## Outputs

- OutcomeAnalysisResult
- TradeThesisEvaluation
- HypothesisAttribution
- DefenseEvaluation
- SupervisorEvaluation
- DemoLiveDivergence
- CounterfactualResult
- ResearchCandidate

## Responsibilities

- WIN / LOSSだけで評価しない
- Trade成功とHypothesis正しさを分離
- ExecutionRecordとProductionEvidenceを使い、Execution OutcomeとLive Evidenceを分けて分析する
- Demo / Live divergence分析

## Prohibitions

- EntryThesisを生成・改変する
- ProductionEvidenceを生成・改変する
- 分析結果を直接Production変更へ反映

## Boundary

```text
ProductionEvidence = Liveで観測された事実寄りEvidence
Post-Trade Analysis = そのEvidenceが何を意味するかを分析
```

---

# ROLE-RTR-001: Research Router

## Category
POST-TRADE / RESEARCH INTERFACE

## Purpose

Post-Trade / Runtime / Market理解で発生したResearchCandidateを原因領域別に適切なResearchへ送る。

## Inputs

- ResearchCandidate
- HypothesisAttribution
- Failure / Unexpected Success

## Outputs

- ResearchRoute
- ResearchCandidate reference

## Responsibilities

例:

```text
Data issue → Data Quality Research
Event detection issue → Event Detection Research
Formula issue → Formula Research
Feature issue → Feature Research
Market interpretation issue → MI Research
Causal issue → Causal Research
Hypothesis Set / Thesis composition issue → Hypothesis Set / Production Thesis Research
DNA mismatch → Market DNA Research
Entry snapshot / Execution issue → Execution Research
ProductionEvidence / Demo-Live divergence → Execution / Live Evidence Research
Supervisor issue → Supervisor Research
```

## Prohibitions

- 全失敗を一律Trainerへ送る
- Routingだけで原因を確定する

---

# G. PLATFORM / OPERATIONS

# ROLE-RT-001: Python Runtime / Operations

## Category
PLATFORM / OPERATIONS

## Purpose

Python Process群を正しい順序・依存関係・状態で起動・停止・維持する。

## Inputs

- RuntimeCommand / Desired State
- Configuration
- Dependency Status
- Health Status

## Outputs

- SystemStatus
- Diagnostics

## Responsibilities

- Boot Sequence
- Process Management
- Scheduler
- Dependency Check
- Config load
- Graceful Shutdown
- Restart execution

## Prohibitions

- Market Signal生成
- Research Rule変更
- Trade方向判断

## Boundary

```text
Outer = どうしたいか
Runtime = 実際にどう動かすか
```

---

# ROLE-MON-001: Monitoring

## Category
PLATFORM / OBSERVABILITY

## Purpose

市場ではなく市場理解OS自身のHealthを監視する。

## Inputs

- Runtime / Process metrics
- Collector / DB / Queue / Exchange health
- CPU / Memory / Disk

## Outputs

- SystemStatus / health update
- ResourceSnapshot
- ErrorEvent / Incident candidate
- Alert

## Responsibilities

- Source latency
- Data freshness
- DB health
- Queue backlog
- Memory / CPU / Disk
- Error rate
- Position / Order operational state

## Prohibitions

- Market方向判断
- Trade Signal生成

---

# ROLE-REC-001: Recovery

## Category
PLATFORM / RESILIENCE

## Purpose

障害時に部分故障を隔離し、安全に復旧する。

## Inputs

- ErrorEvent / Incident
- SystemStatus
- Recovery Policy

## Outputs

- RecoveryAction
- RuntimeCommand / safe-pause request
- Diagnostics

## Responsibilities

- Retry上限
- Backoff
- Circuit Breaker候補
- DB reconnect
- Queue recovery
- Safe pause

## Prohibitions

- 無限Restart
- Failureを隠す
- Research Rule / Trading Ruleを勝手に変更

---

# ROLE-DEPLOY-001: Deployment / Release Control

## Category
PLATFORM / RELEASE

## Purpose

設計・コード・Ruleの変更を安全な段階を通してProductionへ配置する。

## Inputs

- Validated Implementation
- Version Metadata
- Release Approval

## Outputs

- Deployment state / release record candidate
- MigrationRecord（Migration時）
- Rollback target

## Responsibilities

- Version tracking
- Release sequence
- Rollbackability

## Prohibitions

- 未承認ResearchResultの直接Production投入

---

# ROLE-IF-001: Telegram Interface

## Category
INTERFACE / HUMAN CONTROL

## Purpose

人間向けConsoleとして報告・状態確認・操作要求・緊急操作を仲介する。

## Inputs

- SystemStatus
- Market Status Summary
- Alerts
- Human Command

## Outputs

- Display / Notification
- RuntimeCommand request / authenticated command request

## Responsibilities

- `/status`
- `/health`
- `/positions`
- `/risk`
- `/research_status`
- START / PAUSE / RESUME / STOP要求

## Prohibitions

- TelegramからExchange APIを直接叩く
- 独自Market判断
- 認証なし危険操作

## Boundary

```text
Telegram
→ Command Gateway / Validation
→ Outer / Runtime / Defense / Execution
→ Exchange Adapter
```

---

# 5. Cross-Cutting Governance Functions

以下は重要だが、原則として市場処理の独立Top-Level Layerではなく、全体横断Governance Functionとして扱う。

## Resource Governance

対象:

- RAM
- Disk
- DB
- Queue
- Cache
- CPU
- API Cost

目的:

無限増殖・メモリリーク・Storage枯渇を防ぐ。

## Failure Governance

対象:

```text
Error
→ Classification
→ Retry / Isolation
→ DEGRADED / STOP
→ Recovery
→ Reoccurrence Review
```

## Risk / DD Governance

対象:

- Money DD
- Edge Health
- Execution Health
- Data Health
- Market Novelty

Risk State候補:

```text
NORMAL
CAUTION
RISK_REDUCED
MICRO_ONLY
NO_NEW_ENTRY
EMERGENCY
```

## Research Governance

対象:

- Research Admission
- Budget
- Priority
- Early Stop
- Pause
- Reopen
- Archive

## Version / Migration Governance

対象:

- Schema
- Formula
- Feature
- DNA
- Hypothesis
- Trade Thesis
- Entry Snapshot
- Live Evidence Builder
- Config
- Code

## Backup / Disaster Recovery Governance

対象:

- Primary Storage
- Backup
- Restore Test
- Remote / Immutable Archive候補

## Knowledge Aging Governance

対象:

- last_validated
- current applicability
- Demo / Live degradation
- Revalidation due
- Suspend / Retire / Reopen

## Source Lifecycle Governance

対象:

- Provider変更
- API終了
- Schema変更
- Fallback Source
- Logical SourceとProvider分離

---

# 6. 明確にRoleとして扱わないもの

## Objects

```text
RawData
Observation
MarketEvent
Feature
Evidence
CausalHypothesis
MarketDNA
ResearchResult
Edge
Failure
FailureBoundary
Constraint
ApplicableHypothesisSet
TradeThesis
EntryThesis
ProductionEvidence
OrderIntent
TradeResult
```

これらは処理責任ではなくData Object。

## Views

```text
Case Library
Market Memory
Failure Museum
Knowledge Graph
Human Market Diary
```

## Experiment Modes / Tools

```text
Random Baseline
Historical
Replay
Paper
Demo Forward
Shadow
Counterfactual
Stress Lab
Quantum / Search
```

- Stress Lab = Validation Tool
- Quantum / Search = Research Tool

独立必須通過Layerへしない。

## AI Team / AI Meeting

Legacy候補。

現行方針ではExternal AI Reviewへ置換する。

---

# 7. Role間の主要責任境界

## Collector vs Normalizer

```text
Collector = 取る
Normalizer = 揃える
```

## Normalizer vs Data Quality

```text
Normalizer = 形式変換
Data Quality = 信頼可能性評価
```

## Feature vs Event Detection vs Market Intelligence vs Causal Engine

```text
Feature = どう測ったか
Event Detection = 何が発生したかを識別する
Market Intelligence = 発生したものを市場全体の文脈で解釈する
Causal Engine = なぜ発生した可能性があるかを検証可能な仮説へ変換する
```

MarketEventのCanonical生成責任はEvent Detection Processorに一本化する。

## Feature vs Feature Priority

```text
Feature = 測る
Priority = 今何を見るべきか決める
```

## Market Intelligence vs Causal Engine

```text
MI = 何が起きているかを文脈化する
Causal = なぜ起きている可能性があるか
```

## Causal Engine vs Research

```text
Causal = 何を検証すべきか
Research = 実際に検証する
```

## Market DNA vs Feature Priority

```text
MarketDNA_t = 現Evaluation Cycleで確定する詳細な市場状態
FeaturePriority_t = Previous Confirmed MarketDNA (DNA_t-1) + Current Basic Context / MarketEventを使い、現Cycleで何を見る価値が高いか決める
```

Cycle Boundary:

```text
DNA_(t-1)
→ FeaturePriority_t
→ MarketIntelligence_t
→ Causal_t
→ DNA_t
→ 次Cycle
```

`DNA_t → FeaturePriority_t` の同一Cycle逆流は禁止する。

## Research vs Production Thesis Builder

```text
Research / Knowledge Promotion
= Hypothesis / Edgeを研究・検証・承認する

Production Thesis Builder
= Approved Knowledgeを現在市場へ照合しApplicableHypothesisSet / TradeThesisへ構成する
```

未承認HypothesisをBuilderがProductionへ持ち込まない。

## Production Thesis Builder vs Signal

```text
Production Thesis Builder
= 今の市場で「どの承認済み仮説をどう組み合わせ、何を期待するか」を構成する

Signal
= そのTradeThesisに「Riskを取るだけの合理性・期待値があるか」を判断する
```

Builderは`expected_direction / expected_effect / expected_horizon`を表現できるが、`BUY / SELL / NO_TRADE`の最終DecisionはSignal Engineが持つ。

## Signal vs Defense

```text
Signal = 取りたいか
Defense = 取ってよいか
```

## TradeThesis vs EntryThesis

```text
TradeThesis = 取引候補として構成された論拠
EntryThesis = 実際にRiskを取る直前に固定した論拠・市場・Risk・VersionのImmutable Snapshot
```

EntryThesisはSignal / Defense成立後、OrderIntent生成前にEntry Snapshot Builderが生成する。

## Execution Logic vs Exchange Adapter vs Live Evidence Collector

```text
Execution Logic = どう注文するか
Exchange Adapter = 取引所へどう送り、どう約定したかをExecutionRecord化
Live Evidence Collector = 実Liveで確認された約定品質・摩擦・Funding・Liquidity Impact等をProductionEvidence化
```

`ExecutionRecord ≠ ProductionEvidence` を維持する。

## Generator vs Custodian vs Analyzer

```text
Generator = Semantic Objectを正式生成する
Custodian = 生成済みObjectを改変せず保存する
Analyzer = 保存された事実の意味を後から分析する
```

FIX-009では:

```text
EntryThesis Generator = Entry Snapshot Builder
ProductionEvidence Generator = Live Evidence Collector
Custodian = Logger
Analyzer = Post-Trade Analysis
```

## Supervisor vs In-Trade Defense vs Exit

```text
Supervisor = Thesisは生きているか
In-Trade Defense = Hard Safety違反か
Exit Engine = 通常決済するか
```

## Outer vs Runtime

```text
Outer = Desired State
Runtime = Actual State / Process execution
```

## Logger vs Analyzer

```text
Logger = 事実を保存
Analyzer = 意味を分析
```

## Analyzer vs Research Router

```text
Analyzer = 何が起きたか分解
Router = どの研究へ送るか
```

---

# 8. Roleが生成してよいResearch Candidate

各Roleは本来責務を超えて自分自身を研究しない。

ただしProcessing Contractの副産物としてResearchCandidateを出せる。

例:

```text
Collector → recurring source latency
Data Quality → recurring missing pattern
Feature → unstable measurement
Event Detection → detection miss / false positive / duplicate event issue
Feature Priority → wrong priority candidate
Market Intelligence → unexplained move / event detection miss candidate
Causal → strong alternative hypothesis
Market DNA → novel regime candidate
Production Thesis Builder → no applicable hypothesis / composition conflict / applicability gap
Execution / Entry Snapshot → entry timing / snapshot integrity issue
Live Evidence Collector → slippage / partial fill / fee / funding / evidence completeness anomaly
Defense → possible over-blocking
Supervisor → oversensitivity candidate
Monitoring → resource growth anomaly
```

流れ:

```text
Role
→ ResearchCandidate
→ Research Queue / Router
→ Research
```

Role自身が自分のProduction Ruleを自動書き換えない。

---

# 9. Role Failureの共通原則

各RoleはFailure時に少なくとも次を出せる設計候補とする。

```text
role_id
module
market_id
trace_id
error_id
severity
quality_impact
confidence_impact
runtime_impact
recommended_state
research_candidate
```

Failureの結果として許可される方向:

```text
RUNNING
DEGRADED
PAUSED
ERROR
RECOVERY
STOPPED
```

FailureしたRoleが推測値を作って正常処理を装わない。

---

# 10. Roleの長期運用原則

何十年以上運用するため、全Roleに次を適用する。

1. Provider名をRole責任へ埋め込まない
2. Object Schema Versionを追跡できる
3. Formula / Config Versionを追跡できる
4. Input / Output Traceを残せる
5. Role内部実装を将来別言語へ置換可能にする
6. KnowledgeとCodeを分離する
7. Resource Budgetを持てる
8. Failure / Recoveryを定義できる
9. Deprecated Roleを即削除せずMigration可能にする
10. Roleの責任変更はDecision履歴を残す

---

# 11. 新Role追加Gate

新しい案が出た時、次を順番に確認する。

```text
Q1. 既存Roleの責任に入らないか？
Q2. Existing RoleのSubmoduleで十分ではないか？
Q3. 新Object追加だけで解決しないか？
Q4. Contract変更だけで解決しないか？
Q5. View / Mode / Toolではないか？
Q6. 独立したInput / Output / Failure責任があるか？
Q7. 何十年維持する価値があるか？
```

Q1〜Q5で解決できるなら原則新Top-Level Roleを追加しない。

FIX-007の`Event Detection Processor`は、Canonical MarketEvent生成・Detection Rule適用・Event重複統合・Failure時の判定不能表現という独立責任を持つためRoleとして定義する。ただしMarket Observation Plane内の軽量Componentであり、新Top-Level Planeにはしない。

FIX-008の`Production Thesis Builder`は、Approved Knowledgeを現在市場へ照合して`ApplicableHypothesisSet`と`TradeThesis`を生成する独立Input / Output / Failure責任を持つためProduction Roleとして定義する。ただしApplicability SelectionとThesis Compositionを2つの新Top-Level Layerへ分割せず、1Role内部のSubmoduleとして持つ。

FIX-009の`Entry Snapshot Builder`と`Live Evidence Collector`は、独立したSemantic生成責任を持つが、Execution Domain内部責任として成立するため新Top-Level Roleへ昇格させない。実装時は別Module化できるが、Role Dictionary上は`ROLE-EXEC-001`の内部Submoduleとして管理する。

---

# 12. Role変更ルール

Role定義変更は `DESIGN_CHANGE_RULES.md` に従う。

特に次はMAJOR変更候補:

- Responsibility移動
- Prohibition解除
- Input / Output意味変更
- Production権限追加
- Research / Production境界変更
- Safety責任変更

Role名だけ変更して旧責任を失わないようMigration / Alias / Archiveを検討する。

---

# 13. Role Dictionary Definition of Done

Roleを正式定義済みとする最低条件:

```text
□ Role ID
□ Name
□ Category
□ Purpose
□ Inputs
□ Outputs
□ Responsibilities
□ Prohibitions
□ Upstream / Downstream
□ Failure Behavior
□ Research Feedback
□ Long-Term Notes
□ Object / Contractとの境界
□ 他Roleとの責任重複確認
```

---

# 14. 今後の詳細設計との関係

本書はRole責任の辞書。

今後、各Roleの内部アルゴリズム・State・Schema・Python Moduleを深掘りする場合でも、本書を上位参照する。

例:

```text
ROLE_DICTIONARY.md
↓
DOMAIN_RESPONSIBILITY.md
↓
DATA_CONTRACT.md
↓
各Role詳細設計
↓
PYTHON_ARCHITECTURE.md
↓
.py実装
```

下位設計が本辞書へ違反した場合、下位を修正するか、正式なDesign Changeとして本辞書を変更する。

---

# 15. 正式Object名の利用原則

RoleのInputs / OutputsでPersistent Objectを指す場合、`OBJECT_DICTIONARY.md` の正式Object名を使用する。

Legacy /曖昧表現:

```text
FeatureResult
Trial Result
ResearchTrialResult
Validation Result
Research Test Request
CausalEdge / EmpiricalEdge（別Object名としての使用）
```

新規設計での正式表現:

```text
Feature
ResearchResult
ResearchCandidate
Edge + edge_type
```

Historical / OOS / Regime / Demo / Stress等の研究結果は、別Result Object名を増やさず、`ResearchResult` の `evidence_source_channel` / `experiment_mode` / Trial参照で区別する。

---

# 16. 現段階の基本思想

市場理解OSでは、機能数を増やすことを完成度としない。

完成度を高める基準は、

> **各Roleが自分の仕事だけを正確に行い、責任境界を越えず、失敗時にも追跡でき、研究結果を安全に次へ戻せること。**

とする。

新しい案はまず「新Roleか」ではなく、

```text
既存Role
Submodule
Object
Contract
View
Experiment Mode
Governance Function
```

のどこへ置くべきか判定する。

これにより市場理解OSが長期間の設計変更で肥大化・重複・責任衝突することを防ぐ。

---

# 17. FIX-013 State Authority Responsibility Model

FIX-013ではState変更責任を次の4段階へ分離する。

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

## Responsibility Meaning

```text
REQUEST
= State変更の必要性を正式要求する。Current Stateは変更しない。

RECOMMEND
= Evidence / Validation / Domain判断に基づき、どのTransitionが妥当か専門的に推奨する。Current Stateは変更しない。

APPROVE
= Governance / Policy上、そのTransitionを実施してよいと許可する。Current Stateへ直接書き込まない。

APPLY
= Current State / expected_previous_state / Transition Rule / Authority / Approvalを検証し、実際にCurrent State変更を適用する唯一の書込み責任。
```

正式原則:

```text
Request Authority
≠ Recommend Authority
≠ Approve Authority
≠ Apply Authority
```

4責任を必ず別Process / 別人へ分けるという意味ではなく、**意味と権限を論理的に混同しない**ことを意味する。

## Single-Writer Principle

```text
1 State Machine
= 原則1 Apply Authority / single-writer responsibility
```

複数Roleが同じCurrent Stateへ無制限に直接書き込む構造は禁止する。

各Domainでは必要に応じて次の内部責任名を利用できる。

```text
Hypothesis State Controller
Edge State Controller
Knowledge Lifecycle Controller
Production Promotion Controller
Risk State Controller
Research Candidate Controller
Research Plan Controller
Research Plan Lock Controller
Research Trial Controller
Position Thesis Controller
Runtime Controller
Health Controller
Incident Controller
Source Lifecycle Controller
Data Quality Controller
Backup Controller
Migration Controller
Deployment Controller
```

これらは新しいTop-Level Layer / 巨大Roleを意味しない。**各既存Domain内のApply responsibility**を明示する論理名である。

## Shared State Transition Engine

実装では共通の`State Transition Engine`を利用可能とする。

ただし:

```text
State Transition Engine
= transition validation / CAS / atomic apply / event persist等の共通Mechanism

State Authority
= 誰がRequest / Recommend / Approve / Applyできるかという権限
```

であり、共通Engine自体を全State MachineのApproverにしない。

## Safety Asymmetry

Safety RestrictionとRisk Expansionを対称に扱わない。

```text
危険側 / Restrictive Transition
= 明示されたEmergency AuthorityによるFast Pathを許可可能

安全側へのRecovery / Permission Expansion
= Strict Approval / Revalidationを要求する
```

例:

```text
Risk NORMAL → EMERGENCY
Production NORMAL_LIVE → PAUSED
Runtime RUNNING → PAUSED
```

はHard Trigger時にRestrictive Fast Pathを許可できる。

一方:

```text
Risk EMERGENCY → NORMAL
Production PAUSED → NORMAL_LIVE
```

をRestrictive Authorityが単独で自動復帰させない。

## Human / Telegram Boundary

Human / TelegramはState DBを直接書き換えない。

```text
Human / Telegram
→ RuntimeCommand / Manual Override Request
→ Authentication / Authorization
→ State Authority Flow
→ APPLY
→ StateTransitionEvent
```

`/stop` / `/emergency` / `/no-entry`等のSafety Restrictionは、Policyで許可されたEmergency Fast Pathへ接続可能。

`/risk-normal` / `/normal-live`等のRisk Expansion / Permission ExpansionはStrict Approvalを要求する。

## AI Boundary

External AI Review / AI-generated analysisは原則:

```text
REQUEST / RECOMMEND
```

まで。

AI単独で:

```text
APPROVE
APPLY
```

しない。

AIが新しいState変更理由や仮説を提案した場合、必要に応じてResearchCandidate / Recommendationへ送る。

## Logger / Post-Trade Boundary

```text
Logger
= StateTransitionEvent / AuditEventのCustodian

Post-Trade Analysis
= State変更材料を分析しREQUEST / RECOMMEND候補を出せるAnalyzer
```

Logger / Post-Trade Analysisは、別途明示されたState Authorityを持たない限りCurrent Stateへ直接書き込まない。

## ApprovalDecision Boundary

FIX-013では`APPROVE`責任を正式化するが、正式な`ApprovalDecision` Objectは新設しない。

```text
ApprovalDecision Object formalization
→ FIX-015
```

FIX-013時点では`authorization_ref / recommendation_ref`等で後続Object化へ接続可能にする。

## Authority Rule

State Machine別のRequest / Recommend / Approve / Apply割当は`STATE_DICTIONARY.md`のFIX-013 Authority Matrixを正本候補とする。

State変更に関わるRoleは、そのMatrixを超える権限を自Roleへ暗黙追加してはならない。
